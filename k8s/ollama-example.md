# Running Ollama on K8S

A popular way to run open-source Large Language Models (LLMs) is using [**Ollama**](https://github.com/ollama/ollama), a sort of wrapper service around various models including Llama, Mistral, and others.  For example, if you have a reasonably high-perfomance personal machine, you can have a Llama 3.1:8B server running locally in minutes, with no compiler or other overhead required.  A benefit of using Ollama over downloading and running a particular model directly is that it standardizes a lot of the process and exposes a consistent API.  (A caveat is that Ollama may apply quantization to the models for efficiency in an opaque way.)

We can set Ollama up on K8S with a *service* to interact with its API.  This will allow any containers elsewhere on the cluster to call the LLM's API, all in their own containerized environments.

**Important note:**
> This post assumes you have `pod`, `deployment`, `namespace`, and `service` privileges on the cluster.  However, we will make note where you can modify the procedure to work in the more limited setting of just pod, or no namespace, etc.

## Deploying Ollama

To deploy Ollama on Kubernetes, we will largely cop the manifest on the official Github written for this purpose ([`ollama/ollama/examples/kubernetes](https://github.com/ollama/ollama/tree/main/examples/kubernetes)).

This YAML is actually two manifests, separated by `---`.  (Notice we've removed the portion of the example manifest at the link above that creates a namespace --- the assumption here is that you've already been given a namespace by the HPC, so trying to create one will return an error.  For this post we'll pretend your namespace is humorously called [`my-space`](https://myspace.com/discover/featured).)

- First, we setup a `deployment` which will create a `pod`.  Think of a deployment as a desired state you want some set of pods to be in, whereas the pods themselves are the host components of actual execution.  In our case, we'll create a deployment which ensures there is always a single pod running with an image ready to host Ollama models and services.

- Lastly, we setup a `service`.  This is a layer attached to the pod created by the deployment (all in the same namespace), which allows interface between other containers in the namespace and the Ollama pod itself, via a pre-defined dedicated port.

`ollama-gpu.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama
  namespace: my-space
spec:
  strategy:
    type: Recreate  # this strategy ensures one pod is spawned at all times
  selector:
    matchLabels:
      name: ollama
  template:
    metadata:
      labels:
        name: ollama
    spec:
      containers:
      - name: ollama
        image: ollama/ollama:latest
        env:
        - name: PATH
          value: /usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
        - name: LD_LIBRARY_PATH
          value: /usr/local/nvidia/lib:/usr/local/nvidia/lib64
        - name: NVIDIA_DRIVER_CAPABILITIES
          value: compute,utility
        ports:
        - name: http
          containerPort: 11434   # this is the default ollama API port
          protocol: TCP
        resources:
          limits:
            nvidia.com/gpu: 1    # this will ensure we're running on a GPU
      tolerations:
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
---
apiVersion: v1
kind: Service
metadata:
  name: ollama
  namespace: my-space
spec:
  type: ClusterIP  # this exposes the service at cluster level
  selector:
    name: ollama  # this hooks the service to the deployment
  ports:
  - port: 80   # this is the outward facing port the service is exposed on
    name: http
    targetPort: http
    protocol: TCP
```

We can then apply this manifest with

```bash
kubectl apply -f ollama-gpu.yml
```

And then let's go check what we have wrought.  First check on the deployment, you will see:

```
kubectl get deployments -n my-space
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
ollama   1/1     1            1           32s
```

where notice we have to remember to do this within our namespace with the `-n` flag.  Next, the service will show:

```
kubectl get services -n my-space
NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
ollama   ClusterIP   10.108.110.54   <none>        80/TCP    3m18s
```

**Note:** If you don't have a namespace, you can just strip the "namespace" lines out of the manifest above and run it in your own user namespace just fine.  **However,** you won't get some nice DNS resolution for your service that we'll use later, so note the Cluster IP listed here.

Lastly, try checking on pods:

```
kubectl get pods -n my-space
NAME                      READY   STATUS    RESTARTS   AGE
ollama-7c87b867c4-bap8r   1/1     Running   0          7m20s
```

You will see a pod with the deployment name and then a weird hash-looking string.  This is the first pod spawned by the deployment.  If you delete it with `k delete pod` and type out that exact name with the hash, **the deployment will immediately spawn another one**, because it was written with the "Recreate" strategy.  There are other strategies for deployments --- here's some [more on this topic](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).


## Interacting with the pod

We can now do a proof of concept that we can interact with this `ollama` pod from anywhere on the cluster, via the `ollama` service.  To do this, we'll setup a very simple pod that can only do one thing: run `curl` commands.  `curl` will let us send requests to the ollama API and play around.  

So, let's just create a "curl" pod that can do just that:

`curl-pod.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-pod
  namespace: my-space
spec:
  containers:
  - name: curl-container
    image: curlimages/curl:latest  # Lightweight image with curl pre-installed
    command: [ "sh", "-c", "sleep 3600" ]  # Keeps the pod alive for testing
    stdin: true  # To allow interactive mode
    tty: true
```

Now create it with `kubectl apply -f curl-pod.yml`, wait for it to be fully created, and then hop into it in interactive mode:

```
kubectl exec -it curl-pod -n my-space -- /bin/sh
~ $
```

Now let's try to check Ollama actually exists over there on that pod our deployment created.  Our service is listening on port 80, so from within our curl pod (which I'll continue denoting with the `~ $` prompt) we can call:

```
~ $ curl http://ollama:80/
```

and we should get the pleasant response `Ollama is running`.  Yay!  (**Note:** if you don't have a dedicated namespace, then instead of `ollama` in this url, you'll need to put the explicit cluster IP the service is running on.)

Let's try checking on the API:

```
~ $ curl http://ollama:80/api/version
{"version":"0.3.10"}
```

Nice.  So let's do something more interesting and pull an actual model.


## Pull a model and make calls to the API

Recall that Ollama is just a wrapper service --- we don't currently have any actual LLM on the pod, just a server and REST API interface.  We could `exec` into the ollama spawn pod and pull a model directly from there, but it is just as convenient to do it from a distance in our little curl pod.

Still in our curl pod, let's download the smallest Llama 3.1 model (8B parameters).  This takes only a minute or so so we'll set `"stream": false`.  You can check out their [API documentation](https://github.com/ollama/ollama/blob/main/docs/api.md) for all the parameters.

```
~ $ curl http://ollama:80/api/pull -d '{"name": "llama3.1", "stream": false}'
```

And after a minute or so you should see `{"status":"success"}`.  Now we can test the model is working with a prompt/response call:

```
~ $ curl http://ollama:80/api/generate -d '{"model": "llama3.1", "prompt": "Hi!", "stream": false}'
```

which will give something like

```
{"model":"llama3.1","created_at":"2024-09-17T01:47:29.158282962Z","response":"It's nice to meet you. Is there something I can help you with, or would you like to chat?","done":true,"done_reason":"stop","context":[128006,882,128007,271,13347,0,128009,128006,78191,128007,271,2181,596,6555,311,3449,499,13,2209,1070,2555,358,649,1520,499,449,11,477,1053,499,1093,311,6369,30],"total_duration":547907910,"load_duration":19225765,"prompt_eval_count":12,"prompt_eval_duration":23206000,"eval_count":24,"eval_duration":463147000}
```



