---
layout: post
title: "Proxmox VM Manual Backup Guide"
date: 2024-08-30 18:15:00 +0200
categories: self-hosted homelab proxmox linux
tags: homelab proxmox linux cli
image:
  path: /assets/img/headers/virtual-machines2.webp
  lqip: 

---

Proxmox Backup Guide
====================

Description
-----------

This document describes how to perform backups and restore virtual machines (VMs) and LXC containers on Proxmox VE.

Requirements
------------

*   A server with Proxmox VE installed
*   Access to the Proxmox administrative panel
*   Configured storage for backups (local, NFS, or other)

How to Perform a Backup
-----------------------

### Virtual Machine (VM) Backup

1.  Log in to the Proxmox VE panel.
2.  Select the appropriate server (node) and VM from the list on the left side.
3.  Go to the `Backup` tab.
4.  Click `Backup now`.
5.  Choose the backup storage, backup format, and type of operation (e.g., live backup).
6.  Click `Backup` to start the process.

### LXC Container Backup

1.  Log in to the Proxmox VE panel.
2.  Select the appropriate server (node) and LXC container from the list on the left side.
3.  Go to the `Backup` tab.
4.  Click `Backup now`.
5.  Choose the backup storage and other configuration options.
6.  Click `Backup` to start the process.

How to Restore from Backup
--------------------------

1.  Log in to the Proxmox VE panel.
2.  Go to the `Storage` tab and select the storage where backups are kept.
3.  Find the relevant backup and choose the `Restore` option.
4.  Follow the on-screen instructions to restore the VM or LXC container.

Backup File Location
--------------------

*   Default location for local backups: `/var/lib/vz/dump/`
*   For NFS/CIFS storages, the location depends on the storage configuration.
*   Check the storage configuration under `Datacenter -> Storage` in the Proxmox panel.

How to Transfer a VM or LXC Container to Another Host
-----------------------------------------------------

To transfer a virtual machine or LXC container to another host that is not part of a cluster:

1.  Perform a backup of the VM or container using the steps outlined above.
2.  Copy the backup file to the target Proxmox server using SCP or another secure file transfer method.
3.  On the target Proxmox server, go to the `Storage` where the backup file is located.
4.  Select the backup file and use the `Restore` option to recreate the VM or container on the new host.

Scheduling Automatic Backups
----------------------------

1.  Go to `Datacenter -> Backup`.
2.  Create a new backup job (`Add`).
3.  Configure the schedule, select VMs/containers for backup, storage, and other options.
4.  Save the configuration.