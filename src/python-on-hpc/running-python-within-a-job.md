# Python + Conda in a Job

Using Python with Anaconda on the cluster in a non-interactive job basically just requires putting all the steps you would normally do interactively, in a batch job script.

To follow the instructions below, we'll assume you're already familiar with working with Conda and environments from [this post](conda-environments.md), and with writing basic batch job scripts from [this post](../the-batch-system/non-interactive-jobs-content/non-interactive-jobs.md).

## Slurm

Let's write a job script that allocates 1 gpu, loads Anaconda, activates an environment, and runs a simple script to test the GPU is working.

I'll use a preconfigured environment called `torch-env` that has a CUDA-enabled PyTorch package distribution.

Create a two-liner Python script to test our PyTorch package is working on GPUs:

`torch-test.py`

```bash
import torch
print(torch.cuda.is_available())
```

and your corresponding job script:

`torch-job.sh`

```bash
#!/bin/tcsh
#SBATCH --job-name=ttest 
#SBATCH -N 1 -n 1
#SBATCH -t 0:30:00
#SBATCH --gpus=1

# load the anaconda module
module load anaconda3/2021.05

# activate your environment
conda activate torch-env

# ensure we're in the correct directory
cd ~/tests/batch

# run the script in this directory and save outputs to file
python torch-test.py > output.out

# print something to shell as confirmation
echo "Complete"
```

Then submit with:

```bash
sbatch torch-job.sh
```

You'll see the submission confirmation with your job ID.  Now we can check on status with `sinfo` or `squeue` or `squeue -u <user>` as before.  Once it's complete, check your `slurm-<job id>.out` file for the "Complete" message, and your `output.out` file for the "True" message.

Now you can run Python scripts in non-interactive jobs on the HPC!

## Torque (archive)

A near-equivalent for Torque is included below:

```bash
#!/bin/tcsh
#PBS -N demojob
#PBS -l nodes=1:vortex:ppn=12
#PBS -l walltime=00:30:00
#PBS -j oe

source "/usr/local/anaconda3-2021.05/etc/profile.d/conda.csh"
module load anaconda3/2021.05
module load python/usermodules

unsetenv PYTHONPATH

conda activate aml35

cd dml
python hello.py >& output.out
```

```python
import sys
print(sys.version)
```

If you qsub the above files, you will be provided with a single output (output.out) that contains the outputs of hello.py. In this example, if the conda environment aml35 has a python 3.5 installation active, it should report back that the version of Python is 3.5 when it runs on the cluster. You can also manually create files from within your python script, just like you would on a local machine.
