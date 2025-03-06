# 🐍 Using Python: Modules, Conda, and Virtual Environments

<div class="warning">
<b>Due to licensing issues Anaconda has been disable on the HPC and this section has not been updated.</b> Use Miniconda or virtualenv instead.
</div>

Working with Python on the cluster is similar to how you would use it on a local machine, but with several key differences.  We will focus on using Python via the [**Anaconda ecosystem**](https://docs.anaconda.com/) because it is the sanctioned method on the cluster and it plays better with CUDA-enabled operations.  We will also cover how to setup and use Conda virtual environments --- it is _highly recommended_ that you use Conda environments when running code the HPC, and not "work in base".

## Using Environment Modules

Log onto a SciClone frontend and try to see where Python or Conda are located with the following commands:

```bash
which python
which conda
```

Nothing will come up.  Even though these software resources exist on the cluster, you do not yet have them loaded for use.

The cluster uses **environment modules** to manage software use.  This allows users to control which software is currently exposed to them, which helps with dependency and other environment resolution issues.  The official W&M help page is more thorough; you can [find it here](https://www.wm.edu/offices/it/services/researchcomputing/using/modules/).

To see what modules are currently loaded, type:

```bash
module list
```

You should see a short list of the `modules` resource itself, and the batching system (Slurm currently on all subclusters).  To see what modules are available, type:

```bash
module avail
```

This will show a couple dozen modules, which are available across all subclusters, regardless of frontend, and includes things like Julia, Matlab, Python, Conda, and many others.

## Loading Conda

To load Conda and start using it, we can just do:

<!-- ```
source "/usr/local/anaconda3-2021.05/etc/profile.d/conda.csh"
module load anaconda3/2021.05
``` -->

```bash
module load anaconda3/2023.09
```

where the `anaconda3/2023.09` portion needs to match the actual available module, and may update year to year.  (**Note:** you don't have to type that whole string, after you type `module load` you can start typing `ana` and then use tab to auto-complete the name.  You may also set an alias for this command in your `~/.bashrc` if you like.)

This will source and load the Anaconda module so it is ready for you to use.  Now if you try the `which conda` command again, you will see something like:

```bash
$ which conda
conda:   aliased to source /sciclone/apps/anaconda3-2023.09/etc/profile.d/conda.csh
```

You can check what packages are installed (in the base environment, more on this below) with:

```bash
conda list
```

There are dozens installed, so if you are really interested in scouring the list you could do something like `conda list | less`.  This "pipes" the output of `conda list` into the bash function `less`, which lets you go through one page at a time using the spacebar.  When you're done, press `q` to quit and you'll be back on the command prompt.

## Intro to virtual environments

If you are unfamiliar with virtual environments, they may seem intimidating at first.  Overcome this feeling!  It is essential you get comfortable with working in venvs: to keep a clean footprint on the subcluster, to allow you to work with multiple complex projects, to enable collaboration.  (Also, there is some conceptual similarity between venvs and containerization, which we'll explore later in the Kubernetes section, so getting comfortable with venvs will help jumpstart your comfort level with K8S.)

The idea of a virtual environment (venv) is to create a containerized space where you can install specific packages needed for a project, that does not affect other venvs.  So, for example, you can share your entire codebase with a `requirements.txt` that lists the packages (with version numbers!) needed to run it, and others can replicate your work.  Or, you can keep your many personal projects and their different dependencies separate.  Or, while constructing a venv, you make a mistake or some package is not playing well with others, you can *delete it all and start over again* without worrying about affecting your base install.

(**Note:** virtual environments are created by default with zero packages and on top of your core Python/Conda install. So for example if you're running Python 3.11, your venv will be whatever packages you want, but always on Python 3.11.  To change this, and specify the Python version, is sometimes straightforward but is beyond the scope of this tutorial.)


## Virtual environments in Conda

In base Python, creating a virtual environment involves some non-intuitive steps.  First you create it with `python -m venv <env>`, then you "source" the Python binary within that environment, then when you're done you "deactivate" it.  

Conda is very similar, but slightly more user friendly.

First, you can list existing venvs with either of the following commands:

```bash
conda env list
conda info --envs
```

If this is your first time using it, you will note a single env named `base` that lives in the cluster-wide install and is associated with the million packages you saw earlier with `conda list`.  We generally want to avoid this one.

Instead, let's create our own venv. Use the following command to create your environment and type `y` when prompted `Proceed ([y]/n)?`

```bash
conda create -n [ENVNAME]
```

(**Note:** some of the scikit learn examples assume your envname is "aml35" and you use python=3.5, i.e. `conda create -n "aml35" python=3.5`, an example of specifying a different Python version we mentioned earlier.)

Take a look again at your list of environments (`conda env list`) and you will see yours listed, and residing in your home directory in `/sciclone/home/<user>/.conda/envs/<env-name>`.  **Note: this means any packages you install are taking up space on your allocation on the cluster.**

Next, activate your new environment by typing:

```bash
conda activate [ENVNAME]
```

Your prompt will change slightly, with the name of the environment in parantheses ahead of the prompt, something like:

```bash
(env-name) [bora]
```

You can navigate around your home directory from this "environment" shell just like in a normal shell, with the knowledge that any `conda` commands will be within the virtual safe space of `(env-name)`.

Try peeking at the packages installed in this environment with `conda list`.  You will see only `pip` and `setuptools`!  It's a clean space.

You can now install any packages you need for a program using standard conda install commands, for example:

```bash
(env-name) [bora] conda install pandas
```

and if you want, check `conda list` again to ensure it's installed.

(Again, note we recommend sticking with `conda` for package install and management --- if you start using `pip` or `python` it can introduce strange conflicts that are hard to debug.)

Once you've finished in the venv, make sure to "deactivate" it with

```bash
(env-name) [bora] conda deactivate
```

and you will pop back to your normal shell.

## Using environments with `requirements.txt`

Let's say you are interested in replicating the code from an interesting paper with Python code hosted on a Github repository under `https://github.com/user/project`.  Typically, in the base folder for the repo, you will find the researcher has made a `requirements.txt` file with a newline-delimited list of the packages (+versions) used to run the code.

This makes our life very easy.  Assuming we don't need to fret about Python versions, we can download  and prepare to run this code all from the command line on the cluster:

First clone the project into your home directory:

```bash
git clone <url.git>
```

Then create a new venv *just for this project*

```bash
conda create -n project-env
```

Navigate to the base folder of the repo and activate into your environment

```bash
cd project
conda activate project-env
(project-env)
```

Now install all the packages into your venv based on the requirements.txt with

```bash
(project-env) conda install --file requirements.txt
```

(**Note:** you may need to run `conda install pip` before this.)

Now you're ready to run their scripts!  Remember to `conda deactivate` out when you're done.

There are [other ways](https://datumorphism.leima.is/til/programming/python/python-anaconda-install-requirements/) to accomplish this that you may explore, and of course, next you'll want to look up how to _create_ a requirements.txt for your own project.
