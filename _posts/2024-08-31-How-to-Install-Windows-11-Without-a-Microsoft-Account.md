---
layout: post
title: "How to Install Windows 11 Without a Microsoft Account"
date: 2024-08-31 17:45:00 +0200
categories: Windows
tags: Windows Installation OOBE
image:
  path: /assets/img/headers/Windows4.webp
  lqip: 

---

By default, users must log in with a Microsoft account to install Windows 11 or go through the Out of the Box (OOBE) setup process, which runs the first time they turn on a new laptop or desktop computer. If you want to use a local account and don't want to use a Microsoft account or your computer won't have Internet access, just follow a few simple steps.

## How to Install Windows 11 Without a Microsoft Account

There's a simple trick for setting up a local account that involves issuing a command to keep Windows from requiring Internet to install / set up and then cutting off Internet at just the right time in the setup process. This works the same way whether you are doing a clean install of Windows 11 or following the OOBE process on a store-bought computer.

1. **Follow the Windows 11 install process until you get to the "choose a country" screen.**

![OOBE1](/assets/img/posts/2024-08-31-OOBE/OOBE1.png)

Now's the time to cut off the Internet. However, before you do, you need to issue a command that prevents Windows 11 from forcing you to have an Internet connection.

2. **Hit Shift + F10**. A command prompt appears.

3. **Type**  `OOBE\BYPASSNRO` to disable the Internet connection requirement.

![OOBE2](/assets/img/posts/2024-08-31-OOBE/OOBE2.png)

The computer will reboot and return you to this screen.

4. **Hit Shift + F10 again** and this time **Type** `ipconfig /release`. Then **hit Enter** to disable the Internet.

![OOBE3](/assets/img/posts/2024-08-31-OOBE/OOBE3.png)

5. **Close the command prompt**.
6. **Continue with the installation,** choosing the region. keyboard and second keyboard option. 

A screen saying "Let's connect you to a network" appears, warning you that you need Internet.

7. **Click "I don't have Internet"** to continue.

![OOBE4](/assets/img/posts/2024-08-31-OOBE/OOBE4.png)

8. **Click Continue with limited setup**.

![OOBE5](/assets/img/posts/2024-08-31-OOBE/OOBE5.png)

A new login screen appears asking "Who's going to use this device?"

9. **Enter a username** you want to use for your local account and **click Next**.

![OOBE6](/assets/img/posts/2024-08-31-OOBE/OOBE6.png)

10. **Enter a password** you would like to use and **click Next**. You can also leave this field blank and have no password, but that's not recommended.

![OOBE7](/assets/img/posts/2024-08-31-OOBE/OOBE7.png)

11. **Complete the rest of the install process** as you normally would.