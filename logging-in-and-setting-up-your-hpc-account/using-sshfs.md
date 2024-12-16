# Mounting a remote drive locally

You may find it convenient to "mount" a remote drive (like your `/sciclone/home/<user>` directory) on your local, personal computer.  This means you add the directory to your local file system in a way that looks just like a local directory to your computer and all the apps on it.  Once you do this, you can open your remote folder in your Files/File Explorer/Finder app just like any other folder, move files around, etc -- you can even open scripts on the remote machine in a coding IDE on your local computer, without any plugins/SSH necessary!

Let's learn how to do this.  It is easy on Linux, and fairly easy on Mac and Windows.  Instructions for Windows are not included here (if you have done it, [send a PR to this repo](https://github.com/D8A-SCIENCE/hpc-gitbook) and we'll add it!)


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

You should see your folders.  Note that your symlinked folders like `data10` or `scr*` will appear here, but are inaccessible -- those live on a different machine on SciClone and require their own mount.

Try making a file with `touch test.txt`.  Now SSH directly into the remote server and you'll see the file is there.  You should be able to open it and edit, in both places.  (This is because while working in the folder via SSHFS, you inherit your remote UID through the SSH actions happening under the hood.  It would be inconvenient if you were your local computer's UID when on the remote directory!)

To unmount the drive, you just do

```bash
umount ~/mounts/sciclone
```


## Re-connection options

If you restart your computer, it will disconnect this mount and you'll still see your `~/mounts/sciclone` directory but it'll be empty.  It is probably best practice to manually re-mount the drive each time, especially for example, a desktop with a hard-wired connection.  

Sometimes, when the computer goes to sleep, it disconnects, so you may try adding the flag `-o reconnect` to your `sshfs` command.

Also if you prefer, instead of manually reconnecting on startup every time, there are several ways to do this automatically on startup, that we will not cover in detail here but you can look up:

- Add a line to your `/etc/fstab` file ("filesystems table") which tells Linux how to mount drives on startup.  
- Create a bash script `mount.sh` with the sshfs command and then configure Linux to run this script on startup ("Startup Applications")
- Use `systemd` to manage the mount (this is the most robust solution) but also most involved.


## Mounting on Mac

Mac doesn't have native SSHFS support, so we need to use the third-party [`macFUSE`](https://macfuse.github.io/) (formerly OSXFUSE) utility.  You can install it with Homebrew (`brew install macfuse`), or directly from their [website](https://macfuse.github.io/).  (Use the "macFUSE" download, *not* the "SSHFS" download.)  MacFUSE allows you to use non-Mac filesystems, on a Mac --- for example, SSHFS.

Once you've installed macFUSE, you need to also install SSHFS (`brew install sshfs`).  (If you don't have Homebrew, you can [follow these instructions](https://brew.sh/).  It is possible to build/install sshfs without brew but it's much more complicated.)

Now you can use the same commands as Linux (see above), knowing that macFUSE is doing some middle-man work under the hood to let the remote FS play nice with your Mac's.



## Mounting on Windows

On Windows, you need tools like WinFsp and SSHFS-Win.  This section is pending someone's input who uses Windows -- come [be a contributor](https://github.com/D8A-SCIENCE/hpc-gitbook).  :)