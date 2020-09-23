# MacOS set up for Linux/Cross platform development

## Docker

## VirtualBox

* Make sure the virtual box system has a large enough disk for development. The defaults are too small for Linux Kernel work. At least 50GB.
   * It is possible to update this afterwards, but it is error-prone.
   * Make sure the CPUs in the VM are set to the same number as on the host - the default is 1

* Install Debian (or similar)
* Install NFS server & other basic development requirements
```sh
sudo apt install linux-headers-amd64 gcc make git bison flex bc libncurses-dev g++ unzip rsync lib32stdc++6 lib32z1 libc6-i386 nfs-kernel-server sudo
```
* Add the user in as a member of the `sudo` group (not needed for Ubuntu)
```sh
su -
vi /etc/group
```
* Set up /etc/exports:
```
# Just open it up entirely, and squash every UID down to the local user
/home/USER	192.168.56.0/24(rw,sync,no_subtree_check,insecure,all_squash,anonuid=1000,anongid=1000)
```
* Restart nfs-kernel-server
```
sudo /etc/init.d/nfs-kernel-server restart
```
* Install sshd
   * Copy the keys from the host onto the guest, so that git access can be done properly from either
   * Copy the id_rsa.pub file onto the guest machine and call it 'authorized_keys', so that ssh can go through smoothly
* Set up both a NAT'd network interface (so the VM can get to the internet ok), and a host-only network interface (so we can NFS mount the guest folders back to the host & ssh into it nicely)
   * We don't want to use the host-only interface for anything other than talking to the host, so shut down its gateway
   * cat /etc/network/interfaces
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp

# This is the host-only network adapter.
auto enp0s8
iface enp0s8 inet dhcp
	gateway 0.0.0.0
```

* Insert the virtualbox guest additions, mount & run them. Reboot so they take effect.
```sh
sudo mount /dev/cdrom /media/cdrom
sudo /media/cdrom/VBoxLinuxAdditions.run
sudo reboot
```

* Snapshot the Virtual Machine. This is the basic project-agnostic development setup.