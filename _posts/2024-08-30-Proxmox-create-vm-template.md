---
layout: post
title: "How to create Proxmox VM template!"
date: 2024-08-30 18:00:00 +0200
categories: self-hosted homelab proxmox linux
tags: homelab proxmox linux cli
image:
  path: /assets/img/headers/virtual-machines.webp
  lqip: 

---

# Proxmox VM Template  


```sh
apt-get install libguestfs-tools

wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img  

virt-customize -a focal-server-cloudimg-amd64.img --install qemu-guest-agent  

scp focal-server-cloudimg-amd64.img

Proxmox_username@Proxmox_host:/path_on_Proxmox/focal-server-cloudimg-amd64.img  

qm create 9000 --name "ubuntu-focal-cloudinit-template" --memory 2048 --net0 virtio,bridge=vmbr0  

mv focal-server-cloudimg-amd64.img focal-server-cloudimg-amd64.qcow2  

qm importdisk 9000 focal-server-cloudimg-amd64.qcow2 local-lvm  

qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0  

qm set 9000 --ide2 local-lvm:cloudinit

qm set 9000 --boot c --bootdisk scsi0  

qm set 9000 --serial0 socket --vga serial0  
```

Check the cloud-init tab, there you will find some more parameters that we will need to change.  

You will need to change the user name, password, and add the ssh public key so we can connect to the VM later using Ansible and terraform.  

Update the variables and click on Regenerate Image. You can also resize the main disk if you need in this stage. i usually resize them to 15 GB.  

```sh
qm template 9000
```