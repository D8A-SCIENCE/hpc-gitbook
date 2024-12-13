# Mounting a remote drive locally

You will likely find it convenient to "mount" a remote drive (like your `/sciclone/home/<user>` directory) on your local, personal computer.  This means you add the directory to your local file system in a way that looks just like a local directory to your computer and all the apps on it.  Once you do this, you can open your remote folder in your Files/File Explorer/Finder app just like any other folder, move files around, etc -- you can even open scripts on the remote machine in a coding IDE on your local computer, without any plugins/SSH necessary!

Let's learn how to do this.  It is easy on Linux, and fairly easy on Mac and Windows.  Instructions for Windows are not included here (if you have done it, send a PR to this repo and we'll add it!)


## Mounting on Linux

For Linux, we'll use the [`sshfs` utility](https://en.wikipedia.org/wiki/SSHFS) ("SSH Filesystem").  First, make sure it's installed:

```bash
sudo apt update
sudo apt install sshfs
```

Then create a directory where the remote directory will be mounted, for example:

```bash
cd ~
mkdir mounts
cd mounts
mkdir sciclone
```

Now, use `sshfs` to mount the remote directory.  Here we'll mount our home directory on SciClone (this uses the shorthand `bora` because we're assuming you've already setup your [SSH config file](https://d8a-science.github.io/hpc-gitbook/logging-in-and-setting-up-your-hpc-account/configuring-ssh.html), if not you'll need to use the full `<user>@bora.sciclone.wm.edu`):

```bash
sshfs bora:/sciclone/home/<your-user-name> ~/mounts/sciclone
```

If successful, it will return silently (no output) and you can try:

```bash
cd ~/mounts/sciclone
```

You should see your folders.  Try making a file with `touch test.txt`.  Now SSH directly into the remote server and you'll see the file is there.  Yay!

## Mounting at startup

If you restart your computer, it will disconnect this mount and you'll still see your `~/mounts/sciclone` directory but it'll be empty.  Instead of manually reconnecting on startup every time, there are several ways to do this automatically on startup, that we will not cover in detail here but you can look up:

- Add a line to your `/etc/fstab` file ("filesystems table") which tells Linux how to mount drives on startup.  
- Create a bash script `mount.sh` with the sshfs command and then configure Linux to run this script on startup ("Startup Applications")
- Use `systemd` to manage the mount (this is the most robust solution) but also most involved.

And another option is to just rarely restart your computer and 

## Technical notes

- 