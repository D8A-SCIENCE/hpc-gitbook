# 👷 Jobs

## What is a Job?

A batch system manages the allocation of resources on the cluster to users via jobs.  A job is basically a resource check-out (i.e. a reservation for a plot in the "community garden"). When we launch a job, we tell the computer a couple basic things:

1. Which sub-cluster you would like to run on
2. How many nodes and processors you would like to check out
3. How long you want your resources for (i.e. the maximum amount of time you expect your program to take to run) - this is called the _walltime_
4. The name of your job

You can run either non-interactive or interactive jobs.


## How the Job Queue works

When you submit a job (interactive or non-interactive), you **must** specify a few attributes regarding what type of computer node(s) you want to use for your analysis.  These include the number of nodes, the cluster (i.e., what computer architecture - including memory - you want), the length you want the resources for (hours, minutes and seconds), and the number of processors you need on the node. 

Based on these factors, you are entered into a queue - if you request relatively few resources for short periods, the system aspires to give you resources as quickly as possible; jobs with larger walltimes and higher numbers of resources may take a long time before they are able to run.  The specific scheduler we use (Maui) seeks to optimize the cluster so that as many core-hours as is feasible can be provided across all requests.

The take-away from this is that it is important to not request more than you need - i.e., launching more, smaller jobs will enable more flexibility in scheduling, and thus provide you with a higher likelihood of receiving resources.

All of the above applies equally to servers running the Torque or Slurm resource managers.  Specific commands and procedures will differ, however, depending on the manager framework.

Below we will cover some basic commands for Torque and Slurm to understand the job queue.


## Basic Commands

Most basic commands have analogues in Torque/Slurm.  Tor`q`ue commands typically start with `q`, while `S`lurm commands typically start with `s`.  The HPC provides a great Rosetta Stone between the two flavors of resource management on their updated page [here](https://www.wm.edu/offices/it/services/researchcomputing/using/running_jobs_slurm/).

For example, here are some Torque specific commands.  We'll cover these and the Slurm specific ones in subsequent posts.

`qstat` - Provides a list of all jobs currently in the system, requested times, status ("R" for running, "Q" for in queue) and other factors.  You may also provide a username flag, for example: `qstat -u <username>` to limit the list to *your* current jobs.

`showstats` - Provides a synopsis of cluster-wide availability.

`pbstop` - Shows specific use of clusters and nodes at a given time.

`checkjob <jobID>` - Show detailed information on what the job has requested, and some debug information regarding why a job hasn't started (or, if it's running, run statistics).

`showstart` - Shows the systems estimate for when a job will start (or when it started, if already underway).  

`showq` - Shows the full queue of jobs waiting to be scheduled.  `showq - u <username>` shows your queue.


