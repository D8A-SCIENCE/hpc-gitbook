# Jump Servers and Configuring SSH

Using the clusters from a personal laptop not on the WM network requires you to first SSH into a *proxy server* (or "jump" server) to get onto campus, then SSH into the subcluster you want from there.

In this section, we'll summarize how to do this step, and then how to make our SSH lives easier by using SSH keys and setting up `~/.ssh/config`.


## Using the Jump Server (bastion)

WM isolates its clusters on an internal private network and controls outside access using a "bastion" host, which acts as a sort of gatekeeper between the outside world and the internal school network.  (The term *bastion* originally referred to a critical part of a fortification.) 

So broadly, to access a server front-end like `vortex` from a personal laptop and/or off-campus, you need to:

```
$ ssh user@bastion.wm.edu
<authentication>
[bastion] $ ssh user@vortex.sciclone.wm.edu
<more authentication>
[vortex] $ <do stuff>
```

You can also use ssh's `-J` flag (for "jump") and do this all in one command:

```
ssh -J user@bastion.wm.edu user@vortex.sciclone.wm.edu
<lots of authentication>
[vortex] $ <do stuff>
```

This process is summarized on the HPC page ["Logging in to HPC"](https://www.wm.edu/offices/it/services/researchcomputing/using/connecting/).  Note that while authentication to a server from on-campus only involves providing your WM/CAS password, off-campus authentication adds the additional step of 2FA using either a passcode (phone/SMS) or Duo push.

From a personal laptop, go ahead and try to access an HPC subcluster using SSH and the bastion jump server.  (Have your phone ready for the Duo push verification!)


## Configuring SSH keys on Bastion

This 2FA with Duo push is tedious and finnicky.  Thankfully, a better way exists, using SSH-generated public/private keys we can keep stored locally and remote.  (This configuration process is described in detail [here](https://code.wm.edu/IT/bastion-host-instructions) and [here](https://wiki.osuosl.org/howtos/ssh_key_tutorial.html) but we'll summarize.)

First, we need to generate an [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) public/private key pair using SSH's built-in `ssh-keygen` command.  In your `~/.ssh` folder, run:

```
ssh-keygen -t rsa
```

name it something like `wmhpc`, accept the default location, and enter a secure password/phrase (*or if you prefer: just hit enter to not enter a passphrase at all, since the whole point here is to avoid typing a bunch of passwords*).  If you want, save the little ASCII art it generates somewhere.

This command generates two files: `wmhpc` (your private key) and `wmhpc.pub` (your public key).  We want to push the public key to the server, and setup our local machine to authenticate with the private key.

To push a public key to a server, generally speaking you can push directly to the server (which we'll cover below).  In the case of the bastion server, WM has us upload instead to a dedicated folder on `code.wm.edu`.  From the instructions [here](https://code.wm.edu/IT/bastion-host-instructions):

To add your SSH keys to code.wm.edu, visit the [SSH Keys settings](https://code.wm.edu/-/profile/keys) page in your user settings and paste your public key in the key field, give the key a suitable title, and click the add key button. Please make sure you paste your public key (e.g. `wmhpc.pub`) and not your private key! Once your public key is available on `code.wm.edu`, the bastion host should find it when you log in, enabling passwordless authentication.

Once you've upload your public key to Gitlab (code.wm.edu), try the `ssh -J` command again from before.  It should bypass the 2FA authentication for bastion and jump you directly to the subcluster. 


## Setting up `.ssh/config`

We can even further streamline this process by doing some basic configuration of our `config` file.  Navigate to your personal laptop's `~/.ssh` directory and create/open a file called `config`.  Then add the following configurations:

```
Host bastion
  HostName bastion.wm.edu
  IdentityFile ~/.ssh/wmhpc
  User <your username>

Host vortex
  HostName vortex.sciclone.wm.edu
  User <your username>
  ProxyJump bastion
```

The first set of instructions tells SSH that when it sees `bastion`, to replace it with the full domain location (bastion.wm.edu), to pass the private key at `~/.ssh/wmhpc`, and to use user `<your username>`.  So to access bastion now, you could just type `ssh bastion` (try it!).

The second set of instructions tells SSH that when it sees `vortex`, to do some similar replacements, **and** use the proxy server aliased as `bastion` which it now knows is `bastion.wm.edu`.  So to access `vortex`, without even first going on bastion, you can just type `ssh vortex` (try it!).


## Setting up SSH keys for other servers

For bastion, once we uploaded our public key to Gitlab and pointed our config at the associated private key, we avoided any manual authentication with SSH.  

We can mirror this with other servers, but instead of pushing to Gitlab, we just push to the server's `authorized_keys` file which lives in our home directory on that server.  There's even an SSH command to accomplish this:

```
ssh-copy-id -i ~/.ssh/<public_key_file> <user>@<remote_machine>
```

so in our case, since we've already done some work in the `config` file, this might look like:

```
ssh-copy-id -i ~/.ssh/hmwpc.pub vortex
```

Once you've pushed the public key to the server, add the appropriate `IdentityFile` line to your config file, and then try logging in again to the server with the very simple `ssh vortex`.

Voila!  Repeat as necessary for other servers. 

Note: our `config` file example earlier is very explicit, for clarity, but there are ways to reduce the repetitiveness, for example you can specify a config-wide user with:

```
Host *
  User <your username>
```

and WM offers a config-wide proxy jump setting with `Host *.wm.edu !bastion !code.wm.edu ProxyJump user@bastion`.  Mileage may vary --- try some things and see what works.  Of note, when you save your config, you don't have to restart the Terminal/Bash session, the changes take effect immediately, as you may have already noticed.
