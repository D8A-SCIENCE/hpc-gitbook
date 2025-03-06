# Using IDEs on the HPC

> *"All happy developers are alike; each unhappy developer is unhappy in its own way"* --- Leo Tolstoy (slightly modified)

There are as many preferences for developer environments as there are stars in the sky.  You may prefer to work strictly on the command line with tools like [i3](https://i3wm.org/) and [tmux](https://www.redhat.com/sysadmin/introduction-tmux-linux) and vim (or [Neovim](https://neovim.io/)).  You may prefer to avoid command lines wherever possible and use front-end services like the JupyterHub server in the [previous post](https://d8a-science.github.io/hpc-gitbook/python-on-hpc/jupyter.html).  Or you may prefer using an Integrated Developer Environment (IDE) like [VS Code](https://code.visualstudio.com/), or PyCharm, or RStudio, or others.

This post will cover the latter, how to connect your IDE to the HPC clusters and computing nodes.  We will focus on **VS Code**, because it is extremely popular, but similar steps will work for PyCharm and others.

We'll first cover how to connect to a front-end, and then how to connect to a specific compute node running an interactive Slurm job.  We'll assume you're familiar with VS Code and have experience using it to code in Python and Jupyter notebooks on a local machine, so we can focus on getting things working remotely --- this is not designed to be a VS Code crash course.

## Connecting to a Remote Server

Connecting to a remote server like bora, gulf, etc is very easy provided we have done the following key prequisite:

- **Fully setup your SSH config file.  Follow the directions in [this post](https://d8a-science.github.io/hpc-gitbook/logging-in-and-setting-up-your-hpc-account/configuring-ssh.html) if you haven't already.  VS Code will look in this config file for instructions on how to connect to the server you want (proxy servers, SSH keys, etc).  If you don't have this file setup, it will make the rest trickier.**

Once that's complete, in VS Code, install the **"Remote SSH" extension**.  You may also consider adding the Remote Explorer and/or Remote SSH: Edit Config extensions.  You can read the documentation [here](https://code.visualstudio.com/docs/remote/ssh).

Once the Remote SSH extension is installed, open your VS Code Command Palette (on Mac: ⌘ + P, on Windows/Linux: Ctrl+Shift+P) and select "Remote SSH: Connect to Host".  Since you've setup your config file, you will see the list of hosts in that file show up in the menu.  Select one, and wait while it connects (check bottom left of VS Code infobar).

Once it's connected, you can go to the Explorer tab and click Open Folder, or use File -> Open Folder, and you'll see your home directory on SciClone.

**Note:** VS Code will re-install several of your extensions *on the remote host*.  Generally speaking, extensions that affect UI (like themes or snippets) will remain only on your local, while extensions that affect development, compiling, server interaction, will be installed on remote.  This is to provide a more seamless experience and allow you to treat the remote machine as a separate entity.  *A good practice* is to periodically check your extensions while on the remote to ensure they're up-to-date.


## Using Jupyter notebooks in VS Code

We'd now like to open a Jupyter notebook in VS Code.  This is also easy provided we have done the following key prerequisite:

- **Familiarize with the HPC's Conda module and create at least one virtual environment.  Follow the directions in [this post](https://d8a-science.github.io/hpc-gitbook/python-on-hpc/conda-environments.html) if you haven't already.  VS Code knows to look in several standard places for Python environments, like `~/.conda/envs`, so if you already have an environment or two the rest is easy.**

As an additional note, **ensure that any environment you want to run Jupyter notebooks with has the `ipykernel` package installed.**

Given these prereqs, try opening a Jupyter notebook from your SciClone home directory in VS Code.  (We assume you already have the standard VS Code extensions for Jupyter installed.)  In the top right, you will see "Select Kernel".  Click this, and it will open up a list of environments that VS Code auto-detects on the remote machine (just like it does when you're working locally).  Since you already have several setup via conda, pick one from this list and test that it works.


## Editing and running Python scripts in VS Code

We can of course edit Python scripts easily in VS Code, and we can run those scripts by returning to the command line in Terminal or similar app, remoting to the server, activating a Conda environment, and running the script.

**Editing.**  Note that if you have the standard set of VS Code Python extensions (Pylance, Python), VS Code may need some help knowing which environment to use to error-check/lint your code.  You can set it to one of your Conda environments by opening the Command Palette and selecting "Python: Select Interpreter", then your environment.

**Running.**  Note that instead of opening a separate command line session, we can do code execution in VS Code also.  You can open a new Terminal within VS Code with Terminal menu -> New Terminal, or the keyboard shortcut (Windows/Linux) Ctrl-Shift-`, or if you'd like it to open to the side, the Command Palette command "Terminal: Create a New Terminal to the Side."  Go ahead and do this and notice you'll get a command prompt on the remote machine.  

(Note that you do still need to do `module load anacondaXXX` and `conda activate <env>`.)


## Connecting to a compute node

Editing and running things on the front-end is fine for a lot of debugging, but eventually we want to work with high performance resources like GPUs.  This requires getting allocated resources via Slurm and working on a specific compute node --- refer to [this set of posts](https://d8a-science.github.io/hpc-gitbook/the-batch-system/what-is-the-batch-system.html) if you haven't already.

We can connect VS Code to a specific compute node in much the same way we did a front-end, by using the Remote SSH extension and a properly configured SSH config file.

First try this --- SSH into a front-end, we'll use gulf for this example, and allocate a small job with GPUs:

```bash
[gulf] salloc -N 1 -n 1 -t 30:00 --gpus=1
salloc: Granted job allocation 1234
salloc: Nodes gu03 are ready for job
[gu03]
```

Leave this running, and open a new command line on your local machine.  We can now SSH directly into gu03, by using gulf as a proxy server:

```bash
~ $ ssh -J gulf stmorse@gu03
authentication stuff
[gu03]
```

Great!  What this means, is that we can treat the compute node like we've been treating front-ends, as a remote server.  To streamline this and set us up for VS Code, add the following to your SSH config file:

```
Host gu03
  HostName gu03
  User <your-username>
  ProxyJump gulf
```

where [we assume you've already](https://d8a-science.github.io/hpc-gitbook/logging-in-and-setting-up-your-hpc-account/configuring-ssh.html) separately setup `gulf` in the SSH Config file.

Now we're ready to try this from VS Code.  In VS Code, use the Command Palette to do "Remote SSH: Connect to Host", and now select your new entry: gu03.  

After a second or two of authenticating and logging on, you're in!  

Let's say you have a Conda environment pre-installed with `torch` and `ipykernel`.  Open a new Jupyter notebook and try out:

```python
import torch
torch.cuda_is_available()
```

And you'll see the confirmation `True`.


To close a remote session on VS Code, you can use File -> Close remote connection or just close the window.

