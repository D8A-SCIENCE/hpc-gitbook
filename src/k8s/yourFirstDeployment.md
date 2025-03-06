# Your First K8S Deployment

In Kubernetes, there are a number of different core object types that we can use to construct a framework for a project: for example so far we've heard the terms `Pod`, `Job`, `Deployment`, `Service`.  In this post, we'll explore using a `Pod`, which will allow us to quickstart operations on K8S that mirror things we were probably trying to do using Slurm.


## What is a Pod?

An aside: in Kubernetes, you will often hear submissons called "deployments", reflecting the long history of kubernetes as a dynamic scheduler for website resources.  However, there is also a specific Kubernetes object called a `Deployment` which has specific properties and behavior, so we'll try to avoid using the term too loosely.  You can choose to deploy multiple things - for example, here we'll be deploying a single Pod, later we'll learn to deploy an automatically spawning "replica set" of pods, etc.  

Creating a single pod is a simple way to run code against a K8S cluster.  Pods are made up of a few elements, including at least (a) your resource request; (b) your image; and (c) the code you would like to run.  First we'll go through the high level description of each element of a pod, and then we'll show an actual example of a submission script called a *manifest*.

> One critical consideration about Pods is that they are not persistent - anything you do within them will be deleted when the pod is shut down.  We will cover how to achieve persistence in a later tutorial.

### Resource Requests

A single pod can request resources up to what is available on the largest individual node on a cluster, but Pods cannot expand beyond a single physical, available computer.  For example, if you have - somewhere in your cluster - a machine with 1 terabyte of memory and 64 CPUs, then a pod can be built with up to that specification (but, you may have to wait in a queue to get access to those resources if others are using them).  Alternatively, if you have one node with 12 GPUs, then either 12 users could all request 1 GPU in a pod, or one user could request 12 and be allocated to the same node (though, again, you may end up in a queue if resources are not available at any given time).

One unique feature of Kubernetes is that you can request both a *minimum* amount of resources, and a *limit*.  Without the limit, jobs can scale up or down according to demand, but for now we will always be specifying both - ensuring that you receive exactly the resources you want, no more and no less.

### Image

A pod contains a *container* which is built on an *image*.  When you run a pod, you must also specify the base image that will run within it.  Think about a pod as an individual computer you control, and the image is the operating system that you would like it to run --- for example, all the files and binaries and libraries in your root `/` directory, like `/bin` and `/etc` and that other stuff you should never touch.  It's important that the operating system has any drivers you might require (i.e., NVIDIA drivers for GPUs), and you may want it to have some software you frequently use so you don't have to reinstall it every time (for example, Python).  

You can choose the image you want from a cloud-based host like [Docker Hub](https://hub.docker.com/), or build your own image locally (via a utility like podman or docker).  For now, we'll just use "off the shelf" images from Docker Hub, which is the default location our Kubernetes cluster pulls from.

### Scripts and other actions

Just like any other system, once you've checked out your compute resources, you then need to specify the actual code you want to run.  This isn't all that different from logging into a computer and typing, for example, "python myscript.py" to execute a program.


## Define your First Kubernetes Pod

To do this, first you'll need to log in to Kubernetes.  This requires special permission --- assuming you have it, you can access the Kubernetes frontend by jumping from any on-campus server (e.g. bora, gulf, or even bastion) to the K8S frontend `cm.geo.sciclone.wm.edu`.  Specifically, `ssh <your_username>@bora.sciclone.wm.edu` and then from within bora, `ssh <your_username>@cm.geo.sciclone.wm.edu`.  

Once you're logged in, you'll be in your Kubernetes home directory (/home/<your_username>).  **Note: Your K8S home directory is _not the same_ as your SciClone home directory, and they are not symlinked or other connected.**  

In your home directory, you can create manifest (typically using the `.YML` or `YAML` format, pronounced "yammel", although it is possible to use `JSON`) to submit to the K8S cluster.

Let's go ahead and create our first manifest - a *yaml* which defines what we want to do.  In this example, we'll request a single pod with one CPU, and then walk through some basic commands and how to observe what's happening in the Pod.  Using nano or vim, create a file with the following code:

`myFirstPod.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-info-pod
spec:
  activeDeadlineSeconds: 1800  # Pod will be terminated after 30 minutes
  containers:
    - name: resource-info-container
      image: "nvidia/samples:vectoradd-cuda11.2.1"  # this is a base image from Docker Hub
      resources:
        requests:
          memory: "1Gi"
          cpu: "1"
        limits:
          memory: "1Gi"
          cpu: "1"
      command: ["/bin/sh"]   # this ensures we have a shell for interactive mode
      args:
        - "-c"   # everything after this will be run during creation of the pod
        - |      # and show up in the logs (we won't see it if we just enter interactive mode)
          echo "CPU and Memory limits set for this container:"
          echo "Memory limit: $MEMORY_LIMIT"
          echo "CPU limit: $CPU_LIMIT"
          echo "Running the main container process indefinitely..."
          sleep infinity
      env:
        - name: MEMORY_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: resource-info-container
              resource: limits.memory
        - name: CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: resource-info-container
              resource: limits.cpu
```

The above pod does the following:

- It defines the kind of resource we want to deploy - in this case, a Pod.
- It names the pod - this can be anything you want, sans a few special characters, and is the name you'll use to inspect the pod later.
- It specifies an activeDeadlineSeconds - this is the walltime, at which point the pod will be destroyed.
- It defines the base image we want to use, which in this example is a nvidia image with various elements required for GPU utilization.
- It asks for 1 gigabyte of memory, and one cpu.  We request both a minimum (resources) and a maximum (limits).
- I defines the commands to run in the pod.  Here, we just ask it to print the CPU and Memory vailable.
- Finally, it defines a few environmental variables that will be available inside the pod.  These define the two we use - MEMORY_LIMIT and CPU_LIMIT, but you can use this method to ascribe values ot any environmental variables you want.

## Deploying your Pod

Deploying pods in kubernetes is very simple.  In this case, you would run:

```bash
kubectl create -f myFirstPod.yml
```

(Another nearly equivalent option is `kubectl apply -f myFirstPod.yml`, which you will also see used throughout these tutorials.)

If succesful, you will receive a message indicating that "pod/resource-info-pod" was created.  You can confirm the status of your pods by typing:

```bash
kubectl get pods
```

Which will show you the status of all of your running pods.  You should see something like this:

![image](https://github.com/heatherbaier/dist-ml/assets/7882645/45d01c6f-2f1a-45d6-8014-c5c566cfcbb5)

While you're waiting for it to fully create (you want "Running") you can use a command like

```
kubectl get pod resource-info-pod --watch
```

which will update the status automatically when it changes.  Just `Ctrl+C` to exit back to the command prompt.

You can also try `kubectl describe pod resource-info-pod` to see more detailed information --- this is often helpful when debugging.


## Reading the outputs from your Pod

To see the logs generated by your pod, you can run:

```bash
kubectl logs resource-info-pod
```

You will see an output similar to this:

![image](https://github.com/heatherbaier/dist-ml/assets/7882645/bea85702-6d72-42e1-8b02-cf42581011fd)

> Note that logs are not persistent, and are deleted when the pod is deleted.


## Logging in to your Pod

You can enter any pod in *interactive mode*.  This is useful to debug a running process, install software, or just poke around.  Note that your pod has to be running for this to work - the sleep infinity at the end of the pod is what allows this to happen in this example, which will keep the pod alive until it's walltime (defined in activeDeadlineSeconds) arrives - 30 minutes in this example.  To step into the pod in interactive mode:

```
kubectl exec -it resource-info-pod -- /bin/bash
```

The `-i` flag specifies interactive mode and keeps `stdin` open.  The `-t` flag allocates a TTY (terminal interface for you).  Together, `-it` creates an interactive command line session.  The `--` separates the `exec` stuff from a subsequent set of optional commands.  In our case, we've asked to immediately run `/bin/bash` to ensure we get a command prompt.

(Side note: since we already specified `command: ["/bin/sh"]` in the manifest, when we put `/bin/bash` here it is technically overriding our manifest by running a bash shell instead of a `sh` shell.)

Once you execute that line, it will drop you into a new terminal that is running inside of the pod. To confirm you're inside the pod, you can run a command such as:

```bash
cat /etc/os-release
```

This will output the base image the pod is using, which in this example is:

![image](https://github.com/heatherbaier/dist-ml/assets/7882645/c8453e79-9910-4842-a5b3-b0d2c1b5797d)

Once you're done, you can type "exit".

> As you type "kubectl" a lot, you may want to bind it to something shorter.  To do so, go into your Kubernetes home directory and open `~/.bashrc`.  If you want to bind "kubectl" to just "k", you can add the line `alias k='kubectl'` to that file.  Once you save it and restart your shell, you can replace "kubectl" with just "k" from that moment onward.  For example, my own .bashrc looks like:

> ![image](https://github.com/heatherbaier/dist-ml/assets/7882645/6c65863a-11fb-434f-ad2d-24558a22f20b)


## Deleting your Pod

If you do not delete your pod, it will exist until all commands have run or the "activeDeadlineSeconds" limit has been reached.  In this example, that is until 30 minutes have passed - the script sleeps for an infinite time, so will not ever finish, thus resulting in the walltime trigger of 30 minutes being the event that will destroy the pod.  If you do not put "sleep" at the end of your commands, the pod will simply run and then complete, which we'll show in future tutorials.

To manually delete a pod, you can always do so by typing:

```
kubectl delete pod <name-of-pod>
```

In this example, that would be kubectl delete pod resource-info-pod.  If succesful, that command will result in the output:

![image](https://github.com/heatherbaier/dist-ml/assets/7882645/b0f65487-0b19-47ec-8676-10ed1a4a660b)


## Gotchas and Notes

- Pods must have unique names.  You will receive an error if you try to launch two different pods with the same name.
- Remember, your pod is a completely independent OS from the frontend node.  Things you do on the frontend will generally not impact your pods, except in the YML files.
