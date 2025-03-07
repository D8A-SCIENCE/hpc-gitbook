# 👩🌾 What is the batch system?

Think of the HPC like a community garden - users reserve individual plots within a garden space so that people can't plant in their area and they have room to grow what they wish. Think of a whole sub-cluster (e.g. Vortex) as the garden and each of the nodes as a plot you can reserve.

By submitting jobs, we reserve the resources on the nodes for ourselves. If we were to run python scripts, programs, etc.. without submitting jobs, we would all constantly be fighting for resources and our regularly collide with each other. This is what would happen if you were to log into the HPC and run a python program outside of a job - you would essentially be using resources that you did not fairly check out (i.e. you would be planting your veggies in everyone else's garden plots).

## Torque vs. SLURM

There are two common ways to approach this resource management and scheduling problem:

- [**Torque**](https://en.wikipedia.org/wiki/TORQUE) (Terascale Open-source Resource and Queue Manager), which uses PBS (Portable Batch System) and is integrated with a scheduler like Maui or Moab.

- [**Slurm**](https://en.wikipedia.org/wiki/Slurm_Workload_Manager) (Simple Linux Utility for Resource Management).  

**The HPC is currently migrating all subclusters from Torque to Slurm.**  

Most of the resources in this guide (as of September 2024) are written for Torque -- we (and the HPC) are in the process of updating our guides ([want to contribute?](https://github.com/D8A-SCIENCE/hpc-gitbook)).

Both systems use a Message Passing Interface (MPI) to coordinate processes in parallel, and both can run jobs in batch mode or interactive mode, among other similarities, although there also many differences, like where and how jobs are submitted ([check out this summary](https://www.wm.edu/offices/it/services/researchcomputing/using/running_jobs_slurm/)) --- so even instructions written for a Torque context will likely still provide useful help in a Slurm context, with appropriate caution.

Now let's cover basics of jobs, the job queue, and commands for Torque and Slurm, before diving into some more specific examples.

## 👷 Jobs

### What is a Job?

A batch system manages the allocation of resources on the cluster to users via jobs.  A job is basically a resource check-out (i.e. a reservation for a plot in the "community garden"). When we launch a job, we tell the computer a couple basic things:

1. Which sub-cluster you would like to run on
2. How many nodes and processors you would like to check out
3. How long you want your resources for (i.e. the maximum amount of time you expect your program to take to run) - this is called the _walltime_
4. The name of your job

You can run either non-interactive or interactive jobs.

### How the Job Queue works

When you submit a job (interactive or non-interactive), you **must** specify a few attributes regarding what type of computer node(s) you want to use for your analysis.  These include the number of nodes, the cluster (i.e., what computer architecture - including memory - you want), the length you want the resources for (hours, minutes and seconds), and the number of processors you need on the node.

Based on these factors, you are entered into a queue - if you request relatively few resources for short periods, the system aspires to give you resources as quickly as possible; jobs with larger walltimes and higher numbers of resources may take a long time before they are able to run.  The specific scheduler we use (Maui) seeks to optimize the cluster so that as many core-hours as is feasible can be provided across all requests.

The take-away from this is that it is important to not request more than you need - i.e., launching more, smaller jobs will enable more flexibility in scheduling, and thus provide you with a higher likelihood of receiving resources.

All of the above applies equally to servers running the Torque or Slurm resource managers.  Specific commands and procedures will differ, however, depending on the manager framework.
