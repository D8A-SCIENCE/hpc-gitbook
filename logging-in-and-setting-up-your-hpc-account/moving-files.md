# 🐍 Uploading Files

## How to get your files on the HPC

Just like SSH provides us a secure *shell* for working with the cluster, we also need a way to secure *copy* or move files from one machine to another (for example, your local computer to the cluster).  We'll look at three approaches:

- **Using command line tools like `scp` or `sftp`.**  This is probably the most reliable method and the best choice for large files, but is strictly for file transfer and can be cumbersome for complicated work.

- **Using a GUI like Filezilla.**  This is essentially a wrapper of `sftp`, and makes file transfer a little more user-friendly, but is still strictly for file transfer and probably not a good choice for large files.

- **Mounting a drive using `sshfs`.**  This is the most flexible option, since it allows moving and transferring files with your local, OS-native file explorer, *and* opening/editing files as if they were on your local machine.  Still not a great choice for moving large files (due to the risk of mid-transfer disconnect or syncing issues) but a good option for editing scripts --- we cover mounting a drive in a [separate post here](https://d8a-science.github.io/hpc-gitbook/logging-in-and-setting-up-your-hpc-account/using-sshfs.html).


## Using the command line

The most straightforward and out-of-the-box way is using the command line tool [`scp`](https://haydenjames.io/linux-securely-copy-files-using-scp/).  Let's say you're on your local machine and want to move a file in your current working directory to the subfolder `test` in your home directory on the cluster.  You can do (for example):

```bash
scp myFile.txt <user>@bora.sciclone.wm.edu:test/
```

If your [config file is setup](https://d8a-science.github.io/hpc-gitbook/logging-in-and-setting-up-your-hpc-account/configuring-ssh.html), this can simplify to just `scp myFile.txt bora:test/`.  (And don't forget the trailing `/` or else it will think its the filename and you want to rename `myFile.txt` to `test`.)

(**Note:** even if you don't want the file to go in a subfolder on your remote home, you still need the colon and some placeholder like `~/`.)

But there are a wide range of tools you can use to get files onto the HPC.  Another is the SSH Secure File Transfer Protocol, or [sftp](https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server), which creates a new command line session just for file transfer, which is convenient if you're doing several.  Another is [rsync](https://www.samba.org/rsync/).  

The HPC has a whole page on file transfer also, [check it out](https://www.wm.edu/offices/it/services/researchcomputing/using/filesandfilesystems/xfers/).


## Another option: Firezilla

While not appropriate for large files (say, >100Gb), there are also nice GUI options, like [filezilla](https://filezilla-project.org/), a free FTP platform. This brief guide shows how to do just that.

FileZilla is predominantly a GUI-based way to access FTP and SFTP sites, and as such has a simple graphical installer you can download for any platform here: [https://filezilla-project.org/](https://filezilla-project.org/)

### Basic Use

<!-- ![step1](https://user-images.githubusercontent.com/7882645/190205224-068f3893-07a9-4bf5-8165-9e61e248266e.png) -->

<img src="https://user-images.githubusercontent.com/7882645/190205224-068f3893-07a9-4bf5-8165-9e61e248266e.png" alt="Filezilla use" width="90%">

From a very high level, we seek to connect our local files (shown on the left) to some remote server, and then choose what to upload or download with the UI. The first thing we need to do is tell filezilla where the remote server (i.e., SciClone) is located. To do that, first click on "Configure new connection" as shown in the above image.

<!-- ![step2](https://user-images.githubusercontent.com/7882645/190205264-be9bdf45-cf4b-4d09-90ca-3de70e33f77c.png) -->

<img src="https://user-images.githubusercontent.com/7882645/190205264-be9bdf45-cf4b-4d09-90ca-3de70e33f77c.png" alt="Filezilla use step 2" width="90%">

Once clicked, you'll be shown an interface that looks something like this (though you likely will not have any sites pre-configured). First you want to click New Site, and then name your site something you'll remember (mine is called 'sciclone' in the image). You then need to select protocol `SFTP - SSH File Transfer Protocol`, and type in the host for sciclone (i.e., `vortex.sciclone.wm.edu`). Under logon type select Normal, and then type in your William & Mary username and password. Finally, click connect (next time, you'll be able to simply click connect, as the remote server location itself will be saved). Note you may be prompted that the server's host key is unknown - this is a security check; generally, you'll want to check "Always trust this host, add this key to the cache" and then click OK.

Note that "vortex.sciclone.wm.edu" may be inappropriate for many users; "bora.sciclone.wm.edu" should offer faster and more consistent performance in most cases, and is the recommended default as of September 2022.  More details about different fileservers you can connect to can be found [here](https://www.wm.edu/offices/it/services/researchcomputing/using/filesandfilesystems/xfers/) - if you are working in a specific lab on campus, it is best to call directly into the fileserver that they use to minimize network bandwidth and increase file transfer speeds.

<!-- ![step3](https://user-images.githubusercontent.com/7882645/190205296-529fe01c-84dc-4349-a825-9b31f51d4b9b.png) -->

<img src="https://user-images.githubusercontent.com/7882645/190205296-529fe01c-84dc-4349-a825-9b31f51d4b9b.png" alt="Filezilla use step 3" width="90%">

If everything went well, you will now see an interface that looks like the above. Dragging files initiates uploads and downloads, and you can right-click to create new folders as you desire. What you see on the right-hand side (the remote host) will match what you see (except hidden files that start with a '.') when you type "ls" in after you login to SciClone via command line.
