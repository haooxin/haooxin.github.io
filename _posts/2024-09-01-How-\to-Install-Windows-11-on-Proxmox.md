---
layout: post
title: "How to Install Windows 11 on Proxmox VE"
date: 2024-09-01 20:15:00 +0200
categories: Windows Proxmox
tags: Windows Proxmox VM Installation OOBE
image:
  path: /assets/img/headers/Windows4.webp
  lqip: 

---

## How to Install Windows 11 on Proxmox VE

First, download the latest [Windows 11 ISO](https://www.microsoft.com/software-download/windows11) and [VirtIO Windows Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers) and upload them to Proxmox VE.

![iso image upload](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE1.png)

### Preparing the Virtual Machine

1. Select **Create VM** to create a new virtual machine.

![create vm](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE2.png)

2. Enter the **VM ID** and **Name**, then select **Next.**

![create vm](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE3.png)

3. In the **OS** section, select the Windows 11 ISO image. Under **Guest OS**, make sure that **Microsoft Windows** and **11/2022/2025** are selected. Check the box **Add additional drive for VirtIO drivers** and select **VirtIO Drivers ISO** file. Then select **Next.**

![create vm](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE4.png)

4. In the System section, select the same storage location for the **TPM Storage** and **EFI Storage** that you intend on using for the disk. Select **QEMU Agent**, then ensure that **q35** and **OVMF (UEFI)** are selected.

![create vm](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE5.png)

5. In the **Disks** section, select the storage type you’d like to use, then set the **Disk Size**, you can always increase it later.

![create vm](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE6.png)

6. Set the **CPU**, **Memory**, and **Network** settings, then confirm the settings and select **Finish.** Make sure that the VM does not start after creation!

![create vm](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE7.png)


### Installing Windows 11 on Proxmox VE

1. Boot up the VM and access the Windows Setup. Select the correct language settings, then select **Next.**

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE8.png)

2. Select **Install Now**.

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE9.png)

3. Add a **Product Key** if you have one, or select **I don’t have a product key.**

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE10.png)

4. Select the version of Windows 11 you’d like to install, then select **Next.**

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE11.png)

5. Accept the terms (if you agree with them), then select **Next.**

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE12.png)

6. Select **Custom**, for a new installation of Windows.

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE13.png)

7. The windows installer will not be able to find the correct driver for the VirtIO drive and will display an empty list of drives. To install the drivers, select **Load Driver,** browse to the **VirtIO** disk, select **amd64**, then **w11,** and finally, **OK.**

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE14.png)
![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE15.png)
![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE16.png)

8. Windows should find the correct Driver. If it did, select **Next** to install it.

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE17.png)

9. After it’s installed, you’ll see the hard drive selected. You can now proceed and Windows will install! After the installation, the VM will reboot.

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE18.png)

### Windows 11 Setup after installation

Now that Windows 11 is installed, at this stage, you can configure a user account in Windows 11 in two ways: 
- By logging into a **Microsoft account** or, 
- By **[creating a local user account without using a Microsoft account](https://haooxin.github.io/posts/How-to-Install-Windows-11-Without-a-Microsoft-Account)**.


At this step, Windows will search for updates, then get the system ready. When complete, you’ll see the Windows 11 desktop!

If you open the **Device Manager**, you will notice that a few drivers must be installed.

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE19.png)

### VirtIO Driver Installation

1. Open the VirtIO CD Drive and run the installer **virtio-win-gt-x64.msi**.

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE20.png)

2. Run the installer, ensure that everything is selected, then select **Next** to install the drivers.

![windows 11 setup](/assets/img/posts/2024-09-01-Windows11-Proxmox-VM/WinPVE21.png)

3. After the installation is complete, all drivers should be successfully installed.


**That's it, Windows 11 is successfully installed on a virtual machine.**