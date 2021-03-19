# MacOS set up for Linux/Cross platform development

## VirtualBox

* Make sure the virtual box system has a large enough disk for development. The defaults are too small for Linux Kernel work. At least 50GB.
   * It is possible to update this afterwards, but it is error-prone.
   * Make sure the CPUs in the VM are set to the same number as on the host - the default is 1

* Install Debian (or similar)
* Install SSH server & other basic development requirements
```sh
sudo apt install linux-headers-amd64 gcc make git bison flex bc libncurses-dev g++ unzip rsync lib32stdc++6 lib32z1 libc6-i386 openssh-server sudo inotify-tools
```
* Install the public key from the Mac into the VM as `authorized_keys` to make SSH access easier
* Add the user in as a member of the `sudo` group (not needed for Ubuntu)
```sh
su -
vi /etc/group
```
* Set up a port forward on the virtual machine so that ssh works
   * `VBoxManage modifyvm myserver --natpf1 "ssh,tcp,,3022,,22"`
   * https://stackoverflow.com/questions/5906441/how-to-ssh-to-a-virtualbox-guest-externally-through-a-host
* Snapshot the Virtual Machine. This is the basic project-agnostic development setup.

## Visual Code

* Use the Visual Code Remote SSH extension to edit the files remotely
* https://code.visualstudio.com/docs/remote/ssh

## TMux + Iterm2

* Install iterm2 on the Mac machine, and tmux on the remote Linux machine
* Create a helper script that attaches to an existing tmux session (or creates it if it doesn't exist)
```sh
#!/bin/sh
# Assumes there is a NAT forward from localhost:3022 to VM:22

ssh -A -C -p 3022 andre@localhost -o ServerAliveInterval=5 -o ServerAliveCountMax=3 -t 'tmux -CC new-session -A -t laptopsession'
```

## Rsync/Inotify

* Using `inotifywait` & `rsync`, it is possible to set up an automatic mirror of a directory on the VM to a local directory on the Mac
* Don't use this for code, or large numbers of files, it is more for output artifacts/binaries
```sh
#!/bin/sh
# Watches a remote directory for changes and mirrors the files down to a local directory
# Beware - files in the local directory will be delete if they do not exist on the remote directory
# Assumes there is a NAT forward from localhost:3022 to VM:22
# Make sure the trailing / is on these paths, so they get mirrored properly
REMOTEDIR=/home/andre/work/project/release/
LOCALDIR=/Users/andrerenaud/work/project/release/
mkdir -p "$LOCALDIR"

# Start by getting set up
rsync -a --delete -e "ssh -p 3022" andre@localhost:"$REMOTEDIR" "$LOCALDIR"
ssh -p 3022 andre@localhost inotifywait -r -m -e close_write -e delete --format '%w%f' "$REMOTEDIR" | while read MODFILE ; do
        echo "$MODFILE changed. Resyncing..."
        rsync -a --delete -e "ssh -p 3022" andre@localhost:"$REMOTEDIR" "$LOCALDIR"
done
```
