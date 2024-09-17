# Introduction to Kubernetes

Here at William & Mary, we have access to two types of clusters - a Slurm-based batch cluster (see [this](https://hmbaier.gitbook.io/distributed-ml-w-and-m/the-batch-system/what-is-the-batch-system) guide), and a Kubernetes cluster.

## Overview

Let's first try to understand a very high-level idea of how Kubernetes works, then we will explore some concrete differences with batch systems.  In subsequent posts, we will get into technical details and examples.  But first: Kubernetes manages resources and jobs by placing everything inside of *containers*.  Think of a container as a self-contained environment including your scripts, code environments, all the way down to base operating system level files, which is deployed and run on top of the real underlying hardware on the cluster.  Because it's "containerized", it is easier to manage, safer to run, and more efficient to collaborate with and replicate.  

This idea of *containerization* may seem familier because it is so fundamental we see it manifest in other places:  Virtual Machines (VMs) are a very similar idea to the K8S/Docker idea of containers, with a key  difference being VMs include a kernel, which is one layer deeper than containers and makes containers lighter weight.  You are familiar with Python virtual environments --- there is even some rough analogy there: in a venv we specify our base Python install, in a container we specify our base OS; in a venv we then install packages on top of that base, in a container we install programs/scripts/etc; we use venvs because they are contained, safe, and replicable, and we use containers for the same reasons.

As a quick summary, in Kubernetes we create *containers* built on *images* that are associated with and running on certain requested hardware resources on the cluster.  A container is run inside what's termed a *pod*, and we create this entire setup using a file called a *manifest* (a `.YML` file).  We can create the images themselves using tools like podman or docker.  We will later introduce other concepts within this framework, like *deployments* and *services* and *namespaces*. 

But for now, let's understand how K8S differs from the batch system we're familiar with.  We'll cover two ways: **resource management** and **containerization**.


## Differences between K8S and Batch systems

### Node Architectures & Resource Management

In our batch systems, all nodes within a cluster are identical.  In our K8S clusters, you have mixed architectures (i.e., one node may have 0 GPUs, another may have 12).  This has implications for how resources are requested.  In our Batch system, if you wanted to check out a node with 12 cores and 128GB of memory, you could look up what node types have the memory you need and then explicitly specify what cores you want, with a Slurm command like ```salloc -N 1 -n 1 -t 30:00 --gpus=1``` or batch script like

```
#!/bin/tcsh
#SBATCH --job-name=serial 
#SBATCH -N 1 -n1
#SBATCH -t 0:30:00
```

In Kubernetes, you don't specify the explicit node or node type you want - instead you request resources, and the K8S scheduler automatically identifies the optimal node for you to run on.  A resource request looks something like:

`(excerpt)`
```yaml
containers:
  - name: pytorch-setup-container
    resources:
      requests:
        memory: "32Gi"
        nvidia.com/gpu: 1
        cpu: "2"
      limits:
        memory: "32Gi"
        nvidia.com/gpu: 1
        cpu: "2"
```

Practically, this means you don't have to know much about the underlying architecture of the system - you just tell it what resources you need, and it will find them on the appropriate node(s).  The downside is that you get less granular control over the exact resources you may receive.

In our the batch system, when you check out a series of cores, you are sharing some types of resources (i.e. memory) on an node.  Say, for example, you check out 6 cores on a node that has a total of 12, and then another user checks out the other 6.  Because our batch system doesn't have the ability to segment memory utilization, both users are now sharing the memory, and can crash one-anothers jobs if they use all of the memory on the node.  Because of that, we frequently suggest users check out all cores on a node.

K8S isolates your resources into a special construct called a "Pod".  Once a Pod is created on a node, it is allocated exactly the resources requested, and is guaranteed access to those resources, irrespective of what other user pods may be running on the node.  


### Containers and Research Replication

One of the biggest advantages of Kubernetes is that jobs are run within dynamically built containers.  A container is comprised of two things: a base image (essentially, an operating system such as Ubuntu, RHEL, Cent, or Windows), and then anything installed on top of that operating system (for example, a program, a script, a Python environment).  When you request resources and a pod, you always specify the base image that the pod will load.  This then ensures reproducibility for other researchers --- you can share your entire image (for example on an image repository within Github or an image hub like Docker Hub) and collaborators can pull and use your exact environment down to the OS level.  (Again, using the Python venv analogy, there is a similarity here to sharing your code and `requirements.txt` file, just doing it at an OS level.)

Say, for example, you always want a script that runs on Ubuntu 18.04 - you can build an image that will always be identical, irrespective of how external dependencies may change.  This is specified in your job files, and generally looks something like this:

```yaml
containers:
  - name: pytorch-setup-container
    image: "nvidia/samples:vectoradd-cuda11.2.1"  # this is the "image"
```

The job submission file itself then specifies any commands you want to run within that operating system on the pod.  For example, after the base image example above is loaded (an nvidia debian environment), the below code would then install the program "wget" on the image when the job is submitted.  You can choose to either bundle wget into the image itself, or install it later as a dependency, depending on your risk appetite for future changes to dependencies:

```yaml
command:
  - /bin/bash
  - -c
  - |
    apt-get update && apt-get install -y wget 
```

This contrasts dramatically to our Batch system, in which the operating system is prescribed by our HPC team, and any binaries you need installed either need to be compiled on a node, or installed by the HPC team.  In containers, you have considerable control over the types of system-level binaries you may have access to.


