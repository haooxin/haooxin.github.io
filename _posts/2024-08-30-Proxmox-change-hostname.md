---
layout: post
title: "How to update Snap Store!"
date: 2024-08-30 17:50:00 +0200
categories: self-hosted homelab proxmox linux
tags: homelab proxmox linux cli
image:
  path: /assets/img/headers/Proxmox-server-room.webp
  lqip: 

---

# Renaming a PVE node


Proxmox VE uses the hostname as a nodes name, so changing it works similar to changing the host name. This must be done on a empty node.

## Change Hostname
Rename a standalone PVE host, you need to edit the following files:

- **/etc/hosts**
- **/etc/hostname**

You can use pre-installed editor like nano (easy) or vi (advanced), for example:
```
nano /etc/hosts
```
In the above files replace all occurrences of the old name with the new one. Ensure that **/etc/hosts** has an entry with the hostname mapped to the IP you want to use as main IP address for this node.

There are other files which you may want to edit, they are not important for the functions of Proxmox VE itself.

- **/etc/mailname**
- **/etc/postfix/main.cf**

If your node is in a cluster, where it is not recommended changing its name, adapt **/etc/pve/corosync.conf** so that the nodes name is changed also there. See https://pve.proxmox.com/pve-docs/chapter-pvecm.html#pvecm_edit_corosync_conf

## Cleanup
The SSH keys don't need to be edited unless you really want to (and if you do, make sure you make the corresponding change on every other machine that SSH public key appears on).

Now move the configuration files, as the pmxcfs has a few restrictions to ensure consistency you cannot rename non empty folders. Thus if you have VMs or Containers on the node, which is not recommended when changing a nodes name, you have to recreate the folder structure and copy files per folder level.

Also copy the contents of ***/var/lib/rrdcached/db/pve2-{node,storage}/old-hostname*** to ***/var/lib/rrdcached/db/pve2-{node,storage}/new-hostname*** and remove the old directory.

Alternatively, make a backup of the VMs and LXC containers, clean up the node and restore the copy to a node with a new name.

## How To:
### Solution I:
To change the hostname of Proxmox you should first turn off all virtual machines or containers on Proxmox. Then change the hostname of the /etc /hosts file from **“old-hostname.local old-hostname″** to **“new-hostname.local new-hostname″** and the **/etc/hostname** file from **“old-hostname″** to **“new-hostname″**.

 
You can check how many nodes there are on Proxmox in the **/etc/pve/nodes/** folder.

To check node:
```
ls /etc/pve/nodes/
```
First backup the data in old-hostname:
```
cp -R /etc/pve/nodes/old-hostname/ /root/
``` 
Next, move all the files and folders from the old-hostname folder to new-hostname:

```
mv /etc/pve/nodes/old-hostname/* /etc/pve/nodes/new-hostname/*
```
You might go through the below error:
```
mv: cannot move ‘/etc/pve/nodes/old-hostname/qemu-server’ to ‘/etc/pve/nodes/new-hostname/qemu-server/qemu-server’: Directory not empty
``` 
Then need to move the file manually, and execute the below command:
```
mv /etc/pve/nodes/old-hostname/qemu-server/100.conf /etc/pve/nodes/new-hostname/qemu-server/
``` 
And make sure that the **old hostname** node is deleted from the Proxmox dashboard. You can also delete the unnecessary nodes in the /etc/pve/nodes/old-hostname folder.

 
### Solution II:
 
Changing the hostname of a Proxmox node finds a bit harder than changing the hostname of your typical Linux machine. However, you can follow the below simple steps to change the hostname.

First, change the hostname as you would do on any Debian-based machine.
```
hostnamectl set-hostname new-hostname
``` 
Then update your host’s file:
```
vim /etc/hosts
``` 
By this Proxmox will momentarily create a new node with the **new hostname**:
```
ls -la /etc/pve/nodes
``` 
Now you will find a folder with the same name as your **old hostname** and one with your **new hostname**. If you can’t find the folder with the **new hostname**, then you have to wait a bit or you can restart the machine.

Before proceeding  it is better to take a backup of the current config files:
```
cp -r /etc/pve/nodes/old-hostname /root/
``` 
Next, you need to copy the configuration files for the **old hostname** to the **new one**.
```
cp /root/old-hostname/qemu-server/* /etc/pve/nodes/new-hostname/qemu-server
``` 
Finally, reboot the system.
```
systemctl reboot
```