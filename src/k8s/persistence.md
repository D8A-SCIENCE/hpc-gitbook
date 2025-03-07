# Persistence on Kubernetes

In the previous post we got a simple pod running that logged some system information.  But we had no way of storing anything --- everything was [ephemeral](https://www.merriam-webster.com/dictionary/ephemeral), as part of the container philosophy.  In this post, we'll cover two ways to achieve *persistence* in a Kubernetes environment:

- **NFS Mounted Drive.**  Our primary way to achieve persistence is to mount a directory from another filesystem (say, your `/sciclone/home/<user>` directory) into the Pod using the Network File Sharing (NFS) protocol.

- **Persistent Volume.**  A more organic, but in a sense more advanced, approach is to establish a persistent volume (PV) via the Kubernetes framework itself.  A PV is actually an abstract resource in K8S representing a piece of storage in the cluster --- it could be local storage, cloud services, or even NFS.  We request a PV with a Persistent Volume Claim (PVC), which incorporates the storage deliberately into the K8S resource management ecosystem.

But first:

## NFS Mounted Drives

Let's use our SciClone home directory as NFS-mounted persistent storage in a pod.  We'll take our simple manifest from the previous post and add specification to make this happen:

`vol-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vol-pod
spec:
  activeDeadlineSeconds: 1800
  securityContext:
    runAsUser: 123456   # this is your UID on the CM cluster
  containers:
    - name: vol-pod-container
      image: "python:3.11-slim"  # put whatever image you want
      resources:
        requests:
          memory: "1Gi"
          cpu: "1"
      volumeMounts:
      - name: home
        mountPath: "/sciclone/home/stmorse"   # replace with home path
      command: ["/bin/sh", "-c", "sleep infinity"]
  volumes:
  - name: home
    nfs:
      server: 128.239.56.166  # this is the sciclone IP
      path: /sciclone/home/stmorse  # replace with your home path
```

Note the new portions of this YML:

- Security context:  this needs to be your UID on the CM cluster, to put you in the correct context to enable access to the home directory you're requesting.  If you leave this out, it will mount the drive but not let you into it!  One way to find this number is to look in the (read-only) `/etc/passwd` file on CM.  It will be the first 6-digit number after your username.  Another way is to ask your helpful HPC team.

- Volumes.  We need to add the volume mount details in two places:  once at the container level (`volumeMounts`) and once at the spec level (`volumes`).  The container level lines are hooked to the spec-level lines via the "name" attribute, so ensure that matches (in this example, we used "home").  The spec level lines need to include the IP to the server where the filepath resides, in this case the SciClone IP.

Once you modify this YML to match your UID and home directory path, you can go ahead

```bash
k apply -f vol-pod.yml
```

and once it finishes creating, `exec -it` into it

```bash
k exec -it vol-pod -- /bin/sh
```

and go check that your home directory is actually mounted:

```sh
cd /sciclone/home/your-username
ls
```

You could even try saving a test file with `touch test.txt`, go to a different terminal and login to SciClone, and ensure it showed up.

## Persistent Volumes

Another method for creating persistent storage on Kubernetes is using Persistent Volumes (PVs), which we allocate using Persistent Volume Claims (PVCs).  As mentioned at the beginning of this post, PVs are a little more involved than just mounting a home drive with NFS, but they are more organic to the K8S ecosystem and may be the more appropriate choice as your project scales.

To access a persistent volume, you must specify both (a) the claimName for your volume, which the HPC team will provide, and (b) the path you want your persistent volume to be mounted in inside the pod.  In the below example, I have been asigned a 500GB volume with the claimName dsmr-vol-01.  You will not have access to this claim, and will need to replace it with your own.  I am mounting this persistent volume to the path /kube/home within my pod, and then printing out the total disk space available.

Here's an analogous example to the manifest above, but using a PVC:

`pvc-vol-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: claim-example
spec:
  restartPolicy: Never
  activeDeadlineSeconds: 1800  # 30 minutes
  containers:
    - name: pv-container
      image: "python:3.11-slim"  # image doesn't matter here
      volumeMounts:
        - name: home-volume
          mountPath: /kube/home/
      command: ["/bin/sh", "-c", "sleep infinity"]
  volumes:
    - name: home-volume
      persistentVolumeClaim:
        claimName: dsmr-vol-01
```

Notice the differences from the NFS-mounted example.  First off all, we don't need to specify a security context because that authorization layer is already captured by the PVC itself.  We also aren't specifying an external IP, because again, the PV is (at least abstractly) within Kubernetes.

## Checking that it worked

Whether you're working on an NFS-mounted drive or PV, let's check the persistence is working.

We are going to create two files - one in the normal file system (that will be destroyed), and another in the persistent file system (that will be retained).  Go ahead and `k exec -it` into the pod.  Now, first let's create the file we know will be destroyed:

(Your command prompt may be slightly different depending on which YML example you're running.)

```sh
root@claim-example:~# echo "This file is a goner" > ~/doomedFile
root@claim-example:~# cat ~/doomedFile
This file is a goner
```

Now, let's create a file in our persistent storage.  For the PV example, this is `/kube/home/` --- for NFS, it's `/sciclone/home/your-username`.

```sh
root@claim-example:~# echo "This file will live on" > /your/persistent/path/persistentFile
root@claim-example:~# cat /your/persistent/path/persistentFile 
This file will live on
```

Now that we have our files, let's destroy the pod and check it worked.  Type `exit` to get out of the pod, then `kubectl delete pod <pod-name>` to delete the pod.  Once it's deleted, create it again, log back in --- if you try to cat the two files we just created, you'll now see:

```sh
root@claim-example:/# cat ~/doomedFile
cat: /root/doomedFile: No such file or directory
root@claim-example:/# cat /your/persistent/path/persistentFile 
This file will live on
```

Voila!  We now have persistent storage ability while working in a pod.
