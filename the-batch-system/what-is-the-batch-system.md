# 👩🌾 What is the batch system?

Think of the HPC like a community garden - users reserve individual plots within a garden space so that people can't plant in their area and they have room to grow what they wish. Think of a whole sub-cluster (e.g. Vortex) as the garden and each of the nodes as a plot you can reserve. 

By submitting jobs, we reserve the resources on the nodes for ourselves. If we were to run python scripts, programs, etc.. without submitting jobs, we would all constantly be fighting for resources and our regularly collide with each other. This is what would happen if you were to log into the HPC and run a python program outside of a job - you would essentially be using resources that you did not fairly check out (i.e. you would be planting your veggies in everyone else's garden plots).

## Torque vs. SLURM

There are two common ways to approach this resource management and scheduling problem: 

- [**Torque**](https://en.wikipedia.org/wiki/TORQUE) (Terascale Open-source Resource and Queue Manager), which uses PBS (Portable Batch System), and is integrated with a scheduler like Maui or Moab, and

- [**Slurm**](https://en.wikipedia.org/wiki/Slurm_Workload_Manager) (Simple Linux Utility for Resource Management).  

**The HPC is currently migrating all subclusters from Torque to Slurm.**  

Most of the resources in this guide (as of September 2024) are written for Torque -- we (and the HPC) are in the process of updating our guides ([want to contribute?](https://github.com/D8A-SCIENCE/hpc-gitbook)). 

Both systems use a Message Passing Interface (MPI) to coordinate processes in parallel, and both can run jobs in batch mode or interactive mode, among other similarities, although there also many differences, like where and how jobs are submitted ([check out this summary](https://www.wm.edu/offices/it/services/researchcomputing/using/running_jobs_slurm/))--- so even instructions written for a Torque context will likely still provide useful help in a Slurm context, with appropriate caution.

In the next post we'll cover basics of jobs, the job queue, and commands for Torque and Slurm, before diving into some more specific examples.

####
