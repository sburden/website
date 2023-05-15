---
layout: page
title: hyak
subtitle: working with the hyak cluster
---

Tutorial created for folks to work with UW's [hyak](https://hyak.uw.edu/) HPC / supercomputing cluster.

---
## contents
{:no_toc}

* toc placeholder -- gets replaced automagically
{:toc}

---
## overview

We use hyak for two key purposes:  **storage** and **compute**.  This guide will cover basic workflows for both uses.  Regardless of what you are using hyak for, you will need to follow the instructions in the next section for **logging in to hyak**.

The next section after that is on **data transfer/storage/backup with hyak**, and contains several subsections.

My recommended compute workflow for hyak is based on [Apptainer containers](https://apptainer.org/) (formerly [Singularity](https://en.wikipedia.org/wiki/Singularity_(software))).  Apptainer/Singularity only runs natively on the Linux kernel for ([reasons](https://apptainer.org/docs/admin/main/installation.html)), so the first step is getting access to a Linux commandline.  

Fortunately hyak runs Linux and already has Apptainer installed, so if you're happy doing all your development remotely you can skip to the next section, **using Apptainer on hyak**.

If you want to do local development on a Mac / OSX computer, read the section on **getting Apptainer (Singularity) on Mac / OSX**.  I list this first because it's easier (because of course it is).

If you want to do local development on a Windows computer, read the sections on **getting Linux on Windows** and **getting Apptainer (Singularity) on Linux**.  Although this takes some doing, it's 100% viable and exactly what I use.

---
## logging in to hyak

***Tested by:*** TODO

The first thing to know about hyak is that logging in requires [2-factor authentication (2FA)](https://itconnect.uw.edu/security/uw-netids/2fa/) -- *there is no way around this*, it's just the cost of doing business on hyak :)

First, log in to hyak -- you'll need to enter your UW Net ID password (you may want to change yours to something easy to remember and type like [four random words](https://xkcd.com/936/)):
~~~
(local)$ ssh YOU@klone.hyak.uw.edu
~~~
Your command line prompt will now be preceded by `[YOU@klone ~]`, indicating you are logged in to the `klone` cluster on hyak.

*If this is your first time logging in to hyak (only),* enter these commands to enable passwordless login between nodes in the cluster:
~~~
[YOU@klone ~]$ ssh-keygen -C klone -t rsa -b 2048 -f ~/.ssh/id_rsa -q -N ""
[YOU@klone ~]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
[YOU@klone ~]$ chmod 600 ~/.ssh/authorized_keys
~~~

---
## data transfer/storage/backup with hyak

First, it is important to understand that there are multiple locations for data storage on hyak, each with their own uses and constraints.
1. `/tmp`: 350 GB quota per `JOB` on a node's SSD;
2. `~` (i.e. "home"): 10 GB quota per `USER` on the network drive;
3. `/gscratch/GROUP`: 1 TB quota per `GROUP` on the network drive;
4. `lolo:/archive/GROUP`: 1 TB quota per `GROUP` [backed up on tape](https://hyak.uw.edu/docs/storage/data).

The node SSD (1.) is the fastest, but *it is purged when your job ends*, i.e. when the `salloc` command below terminates and/or you disconnect from the node.  So `/tmp` is a great place to build containers, compile software, or do jobs that require a lot of reading from or writing to disk.  It is a [terrible, horrible, no good, very bad](https://en.wikipedia.org/wiki/Alexander_and_the_Terrible,_Horrible,_No_Good,_Very_Bad_Day) place to store irreplaceable data.

The network drive (2. & 3.) is quite fast (uses a smart combination of spinning disks and fast solid-state drives), and is preserved between jobs; furthermore, the directory in (3.) is accessible by everyone associated with `GROUP`, so it's a great place to share data between collaborators.  

However, the only storage that is backed up is (4.), so it is important to regularly archive data from (1.--3.) to (4.), but **(4.) should only be used for backup -- not active work** since (4.) is written to physical tape.

Finally, the networked and backup storage (2., 3., 4.) place limits on the number of individual files (sometimes termed `inodes` in the hyak documentation) in addition to the amount of storage.  So particularly when backing up lots of data, it is important to minimize the number of files.  One easy way to do this is to package stuff up into [Tar](https://www.gnu.org/software/tar/) files -- additional instructions provided below.  But it may be worthwhile to consider this constraint when deciding on a data architecture for your work.

More specifically, if we wanted to use the entire storage quota, the average filesize in the network drive (2., 3.) would need to be > 10 MB, and the average filesize in the backup (4.) would need to be > 1 GB.  So please package your data into sufficiently-large chunks before transferring to hyak or lolo.

With these points in mind, here are instructions for transferring files and directories to/from hyak and backing up data.

### transferring files to/from hyak

First, get an [SFTP](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol) client for your local machine -- some options: [WinSCP](https://winscp.net/eng/index.php) on Windows; [Cyberduck](https://www.ssh.com/academy/ssh/cyberduck) on Mac / OSX; `sftp` on Linux (or in a Unix-style shell on Windows / Mac).  If you opt for a graphical client, you'll need to refer to its documentation -- I'll provide instructions for the Unix-style commandline client below.

Using a commandline client, enter the following command:
~~~
sftp YOU@klone.hyak.uw.edu
~~~
You will need to log in using 2FA as described in the section **logging in to hyak**, at which point you will see the following prompt:
~~~
sftp>
~~~
This is the SFTP prompt, and it functions similarly to the Unix-style command prompt but with a limited set of commands.  For instance, you can `ls`,  `cd`, `pwd`, `mkdir`, `rmdir` just like if you had SSH'd into hyak (which you have technically done, since SFTP is based on SSH).  

But you can also `lls`, `lcd`, `lpwd`, `lmkdir` to navigate your **local** filesystem (thus the `l` prepended to each of the commands), and you can `get` and `put` to transfer files back-and-forth: `get FILE` will transfer `FILE` from hyak to your local machine, and `put FILE` will transfer `FILE` from your local machine to hyak.  

SFTP recognizes wildcards similarly to the local commands, so `put *.tar` will transfer all the [Tar](https://www.gnu.org/software/tar/) files from the present working directory on your local machine (`lpwd`) to the present working directory on hyak (`pwd`).

I'll note that you could alternatively use [rsync](https://linux.die.net/man/1/rsync), which is more full-featured, but not interactive (i.e. you will have to log in to hyak with 2FA each time you issue a command).

### transferring directories to/from hyak

Suppose you have a directory `dir` that you want to transfer to hyak.  If the directory only has a couple of files and it is easy to verify they transferred without errors (e.g. a video file you can watch, or a data file you can load and visualize), I recommend you transfer them directly using the instructions in the previous section.  If the directory has many files, or if it is difficult to verify the files have transferred correctly (e.g. if you have hundreds of individual data files from an experiment), I recommend you use the following workflow to package the directory into an "archive file".  

The reasons for this recommendation are:
* hyak limits the number of individual files in addition to storage on options (2., 3., 4. listed above), so we want to be careful not to exceed the file quota;
* transferring individual files between computers or storage media (e.g. local to/from hyak, or network drive to/from SSD drive) takes longer than transferring a single archive file containing the same data.

If you do not need to compress your data (either because the data are already compressed as in `.npz` or `.mp4` files or you are not worried about storage limits) I recommend using [Tar](https://www.gnu.org/software/tar/) to create an *uncompressed* archive (i.e. a `.tar` file, not a `.tar.gz` or `.tgz` file).  If you want to compress the data while you archive it, I recommend using Tar or [Zip](http://gnuwin32.sourceforge.net/packages/zip.htm), but you will need to modify the following instructions.

Given directory `dir`, creating an uncompressed archive is easy:
~~~
tar -cvf YYYYMMDD-dir.tar dir
~~~
The flags on this command mean to `c` = create a `f` = file, and to provide `v` = verbose output on the commandline as files are added.  The filename format I recommend includes a 4-digit year (`YYYY`), 2-digit month (`MM`), and 2-digit day (`DD`) -- use today's date or some date that is associated with the data (e.g. the date the experiment was conducted).

This `.tar` file can be directly transferred to hyak using the instructions above once you start an SFTP session:
~~~
sftp> cd /gscratch/GROUP/PROJECT
sftp> put YYYYMMDD-dir.tar
~~~
However, I recommend the following additional steps to ensure the data's integrity.

First, check that there aren't any obvious errors with the `.tar` file by `t` = testing it:
~~~
tar -tf YYYYMMDD-dir.tar
~~~
If this command outputs a list of all the files from directory `dir` then you can move on; if it generates an error, you need to re-create the `.tar` file.

Second, create a [checksum](https://en.wikipedia.org/wiki/Checksum) for the `.tar` file:
~~~
md5sum YYYYMMDD-dir.tar > YYYYMMDD.sum
~~~
If the `.tar` file is large (> 10 GB) this command will take more than a few seconds to complete.  Now transfer the `.tar` and `.sum` files to the same place on hyak using the instructions above and use the checksum to verify the `.tar` transferred without errors:
~~~
md5sum -c YYYYMMDD.sum
~~~
If this command returns without errors (it may take longer on hyak than your local machine), you can be reasonably confident that the `.tar` file was transferred without errors!

When you want to transfer the data from hyak back to your local machine, you reverse the process:
~~~
sftp> cd /gscratch/GROUP/PROJECT
sftp> get YYYYMMDD-dir.tar
sftp> get YYYYMMDD.sum
[Ctrl+D to close SFTP connection to hyak]
local$ mdsum -c YYYYMMDD.sum
local$ tar -xvf YYYYMMDD-dir.tar
~~~
Here the flag `x` = extract will create a subdirectory `dir/` in the present working directory (`pwd`) containing all the original directory contents.

As a final note, it can be handy to trim the directory structure when creating a `.tar` file.  So if your directory is at `path/to/dir/` and you want to trim `path/to/` so that `dir/` is the only subdirectory created when you extract the `.tar` file, use the `-C` flag (`C` = change to directory):
~~~
tar -cvf YYYYMMDD-dir.tar -C path/to dir
~~~

## backing up to/from lolo

The [lolo](https://wiki.cac.washington.edu/display/DataBackupsandArchives/lolo+User+Documentation) service is accessed by SSH'ing into `lolo.washington.edu`.  Once there, your storage is located at `/archive/GROUP` where `GROUP` is the same as on hyak.

To make things easy, I recommend first setting up passwordless login from hyak.  Assuming you followed the instructions in the **logging in to hyak** section above, issue the following commands from hyak:
~~~
[YOU@klone ~]$ scp .ssh/id_rsa.pub lolo.washington.edu:id_rsa_hyak.pub
[enter password]
[YOU@klone ~]$ ssh YOU@lolo.washington.edu
[enter password]
-bash-4.2$ mkdir .ssh
-bash-4.2$ chmod 700 .ssh
-bash-4.2$ cat id_rsa_hyak.pub >> .ssh/authorized_keys
-bash-4.2$ chmod 600 .ssh/authorized_keys
-bash-4.2$ exit # <-- or just press Ctrl+D
[YOU@klone ~]$ ssh YOU@lolo.washington.edu
[you shouldn't have to enter your password :)]
-bash-4.2$ exit 
~~~

Now it's easy to move an individual file FILE or entire directory DIR to lolo:
~~~
[YOU@klone ~]$ rsync -av --whole-file FILE lolo.washington.edu:/archive/GROUP/
[YOU@klone ~]$ rsync -av --whole-file DIR lolo.washington.edu:/archive/GROUP/
~~~
The flags specify that we're `a` = archiving data (so directories will be packaged up and sent in their entirety) and we want `v` = verbose output.  Since `rsync` computes its own checksums, we don't need to repeat that process from the `sftp` instructions above.

Reversing the process is also easy to move an individual file FILE or entire directory DIR to lolo:
~~~
[YOU@klone ~]$ rsync -av --whole-file lolo.washington.edu:/archive/GROUP/FILE /gscratch/GROUP/FILE
[YOU@klone ~]$ rsync -av --whole-file lolo.washington.edu:/archive/GROUP/DIR /gscratch/GROUP/DIR 
~~~

In case you're wondering:  yes, you could use `rsync` instead of `sftp` to transfer files to/from hyak -- I chose to explain how to use `sftp` because it is interactive, which is much less of a pain in the tuchus with 2FA.  Since lolo permits passwordless login, I use `rsync`.

## SSH config

If you want to make your life *marginally* easier, I recommend setting up an [SSH config](https://linuxize.com/post/using-the-ssh-config-file/) file.  From a commandline:
~~~
mkdir -p ~/.ssh && chmod 700 ~/.ssh
touch ~/.ssh/config
chmod 600 ~/.ssh/config
~~~
Then edit the `~/.ssh/config` file as follows on both hyak and your local machine:
~~~
Host lolo
  Hostname lolo.washington.edu
  IdentityFile ~/.ssh/id_rsa
~~~
On your local machine, I'd add:
~~~
Host hyak
  HostName klone.hyak.uw.edu
~~~
Now assuming your username on your local machine is the same as your UW Net ID, you can just `ssh hyak` and `ssh lolo`, and from hyak you can just `ssh lolo` -- same goes for using `sftp` and `rsync` <3

---
## using Apptainer on hyak

First, log in to hyak using the instructions above.

Second, know that you will need to log in twice if you want to use the Jupyter notebook, e.g. by opening two separate terminal windows -- *there are probably ways around this*, but I am too novice of a sysadmin to find another solution.

Third, determine the computational resources you need for your work -- `N = # cores`, `M = GB of memory`, `T = hours of work` -- and [select an integer at random between 4096 and 16384](https://hyak.uw.edu/docs/tools/jupyter#installing-and-configuring-jupyter-notebook-for-hyakklone), say `S = special integer`.

With those three prereqs in place, [*lets gooo!*](https://tenor.com/search/lets-goooo-gifs)


Now request your computational resources -- [the values for Colab](https://colab.research.google.com/drive/1_x67fw9y5aBW72a8aGePFLlkPvKLpnBl) would be something like `N = 2` cores, `M = 15` GB of RAM; I'd choose the duration to be approximately twice as long as you actually expect to work, e.g. `T = 2` hours:
~~~
[YOU@klone ~]$ salloc -A GROUP -p PARTITION -c N --mem=MG --time=T:00:00
~~~
(You will need to get your GROUP and PARTITION information from your friendly sysadmin :)

This request will take a few seconds to be fulfilled, at which point you are finally ready to get computin'.  Your command prompt should now be preceded by `[YOU@nABCD ~]`, indicating that you are logged into compute node `nABCD` -- you'll need the integers `ABCD` in a moment.

Enter this command to start the Jupyter notebook server:
~~~
[YOU@nABCD ~]$ module load singularity
[YOU@nABCD ~]$ singularity exec /gscratch/GROUP/sburden.sif jupyter-notebook --ip 0.0.0.0 --port S
~~~
where `S` is your [special random integer between 4096 and 16384](https://hyak.uw.edu/docs/tools/jupyter#installing-and-configuring-jupyter-notebook-for-hyakklone).

Now the Jupyter notebook server is running on node `nABCD`, but you have no way of accessing it :(  Unless, of course, you LOG IN AGAIN just like we said we would:
~~~
(local, again)$ ssh YOU@klone.hyak.uw.edu -L S:nABCD.hyak.local:S
~~~
(In case you're curious, this particular incantation is called [port forwarding](https://www.ssh.com/academy/ssh/tunneling/example).)

***Whew --*** you are now ~done!  Just open your local web browser and navigate to the following URL:
~~~
localhost:S
~~~
where, in case you forgot, `S` is the [super-special random integer you chose between 4096 and 16384](https://hyak.uw.edu/docs/tools/jupyter#installing-and-configuring-jupyter-notebook-for-hyakklone).

You should now be able to get to work ...

...

... oh, what's that you say? ...  

... you're all done?

OK then just type Ctrl+D in the second terminal once (to disconnect from `klone`) and Ctrl+C in the first terminal twice (to close the Jupyter notebook server) and then Ctrl+D in the same (first) terminal twice (first to disconnect from `nABCD`, second to disconnect from `klone`).  Disconnecting from `nABCD` returns your `salloc` compute resources to pasture and disconnecting from `klone` closes your connection with hyak.

IT'S SOO EASY !!!


---
## getting Apptainer (Singularity) on Mac / OSX

***Tested by:*** TODO

The following tl;dir instructions are based on [this](https://sylabs.io/guides/3.0/user-guide/installation.html).  You'll use the [Terminal](https://support.apple.com/guide/terminal/welcome/mac) or your favorite [command line interface (CLI)](https://www.davidbaumgold.com/tutorials/command-line/).

***Note:*** the following instructions are a little out of date since [Singularity evolved into Apptainer](https://medium.com/@dcat52/why-you-should-use-apptainer-21ef1fe7e0bb).  However, since Apptainer can load Singularity containers, the instructions remain totally valid -- you'll just use `singularity` on your local machine and `apptainer` on [hyak](https://hyak.uw.edu/docs/tools/containers/).

First install [Homebrew](https://brew.sh/):
~~~
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~

Next install [VirtualBox](https://www.virtualbox.org/) and [Vagrant](https://www.vagrantmanager.com/downloads/):
~~~
brew cask install virtualbox && \
    brew cask install vagrant && \
    brew cask install vagrant-manager
~~~

Now create and log in to a virtual machine that has Apptainer set up:
~~~
mkdir vm-singularity && \
    cd vm-singularity

export VM=sylabs/singularity-3.0-ubuntu-bionic64 && \
    vagrant init $VM && \
    vagrant up && \
    vagrant ssh
~~~

Your command line prompt should now have changed to `vagrant@vagrant:~$` -- assuming so:
~~~ 
singularity version
~~~
should print out something like `3.0.3-1`.

You're now ready for **using Apptainer (Singularity) locally**.

---
## getting Linux on Windows

***Tested by:*** TODO

It has become remarkably easy to get a full Linux commandline on Windows via the obversely-named [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/about).  The following tl;dr instructions are based on [this](https://docs.microsoft.com/en-us/windows/wsl/install) and [this](https://www.scivision.dev/install-windows-subsystem-for-linux/).

Open a PowerShell window as an administrator:
1. press the Windows key
2. type "powershell"
3. click "Run as administrator"

Enter the following command in the PowerShell window:
~~~
wsl --install
~~~

By default, this command will install the [latest long-term support (LTS) version of Ubuntu](https://ubuntu.com/blog/tag/LTS) using the [latest version of WSL (WSL2)](https://docs.microsoft.com/en-us/windows/wsl/compare-versions).

I recommend installing the [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/) to access this Ubuntu command line.

You're now ready for **getting Apptainer (Singularity) on Linux**.

---
## getting Apptainer (Singularity) on Linux

***Tested by:*** TODO

The following tl;dr instructions are based on [this](https://sylabs.io/guides/3.0/user-guide/installation.html) and should work on (Ubuntu/Debian-style) Linux regardless of whether you are using WSL on Windows or running natively. 

***Note:*** the following instructions are a little out of date since [Singularity evolved into Apptainer](https://medium.com/@dcat52/why-you-should-use-apptainer-21ef1fe7e0bb).  However, since Apptainer can load Singularity containers, the instructions remain totally valid -- you'll just use `singularity` on your local machine and `apptainer` on [hyak](https://hyak.uw.edu/docs/tools/containers/).

***Note:*** Apptainer is already installed on hyak, so you *do not* need to follow these instructions there -- you can skip to the next section.

Start by installing dependencies:
~~~
sudo apt-get update && sudo apt-get install -y \
    build-essential \
    libssl-dev \
    uuid-dev \
    libgpgme11-dev \
    squashfs-tools \
    libseccomp-dev \
    pkg-config
~~~

Next, install and set up [Go](https://go.dev/solutions/#case-studies) (a Google tool for building software):
~~~
export VERSION=1.11 OS=linux ARCH=amd64 && \
    wget https://dl.google.com/go/go$VERSION.$OS-$ARCH.tar.gz && \
    sudo tar -C /usr/local -xzvf go$VERSION.$OS-$ARCH.tar.gz && \
    rm go$VERSION.$OS-$ARCH.tar.gz

echo 'export GOPATH=${HOME}/go' >> ~/.bashrc && \
    echo 'export PATH=/usr/local/go/bin:${PATH}:${GOPATH}/bin' >> ~/.bashrc && \
    source ~/.bashrc

go get -u github.com/golang/dep/cmd/dep
~~~

Finally, get and install Singularity:
~~~
go get -d github.com/sylabs/singularity

export VERSION=v3.0.3 \
    cd $GOPATH/src/github.com/sylabs/singularity && \
    git fetch && \
    git checkout $VERSION

export VERSION=3.0.3 && \
    mkdir -p $GOPATH/src/github.com/sylabs && \
    cd $GOPATH/src/github.com/sylabs && \
    wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-${VERSION}.tar.gz && \
    tar -xzf singularity-${VERSION}.tar.gz && \
    cd ./singularity && \
    ./mconfig

./mconfig && \
    make -C ./builddir && \
    sudo make -C ./builddir install
~~~

You're now ready for **using Apptainer (Singularity) locally**.

----
## using Apptainer (Singularity) locally

***Tested by:*** TODO

These instructions should work the same on your local machine as they do on hyak, so long as you have Apptainer (Singularity) version 3 or above.

***Note:*** the following instructions are a little out of date since [Singularity evolved into Apptainer](https://medium.com/@dcat52/why-you-should-use-apptainer-21ef1fe7e0bb).  However, since Apptainer can load Singularity containers, the instructions remain totally valid -- you'll just use `singularity` on your local machine and `apptainer` on [hyak](https://hyak.uw.edu/docs/tools/containers/).

First, create the following file named `sburden.def`:
~~~
Bootstrap: docker
From: ubuntu:20.04

%post
  apt-get update -y
  apt-get install -y python3 python3-pip
  pip3 install --upgrade pip
  pip3 install numpy scipy matplotlib ipython jupyter sympy pandas control
~~~

Then execute the following:
~~~
singularity build --fakeroot sburden.sif ./sburden.def
~~~

Caveats on hyak:
* you'll need to enter `module load singularity` first to .. load ... apptainer;
* you'll want to replace `sburden.sif` with `/tmp/sburden.sif` and then `cp /tmp/sburden.sif .` once the building finishes for higher performance (`/tmp` is physically on the node, whereas `~` and `/gscratch` are on the network);

