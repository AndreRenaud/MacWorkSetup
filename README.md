# MacOS set up for Linux/Cross platform development

## Docker

## VirtualBox

* Install Debian (or similar)
* Install nfs-kernel-server
* Set up /etc/exports:
```
# Just open it up entirely, and squash every UID down to the local user
/home/USER	192.168.56.0/24(rw,sync,no_subtree_check,insecure,all_squash,anonuid=1000,anongid=1000)
```
* Install sshd
** Copy the keys from the host onto the guest, so that git access can be done properly from either
* Set up both a NAT'd network interface (so the VM can get to the internet ok), and a host-only network interface (so we can NFS mount the guest folders back to the host)
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