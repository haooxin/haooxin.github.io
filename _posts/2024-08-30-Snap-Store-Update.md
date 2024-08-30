---
layout: post
title: "How to update Snap Store!"
date: 2024-08-30 18:45:00 +0200
categories: self-hosted homelab linux
tags: homelab linux ubuntu cli
image:
  path: /assets/img/headers/Linux-system.webp
  lqip: 

---

# Snap Store Update

This guide will help you update the **Snap Store** on your Ubuntu system.


## Requirements

- Administrative rights (access to the `root` account or the ability to use the `sudo` command).
- Access to a terminal/console in the Ubuntu system.

## Snap Store Update:

To update **Snap Store** on your Ubuntu system, execute the following commands in the terminal:

```bash
killall snap-store
```

```bash
sudo snap refresh
```