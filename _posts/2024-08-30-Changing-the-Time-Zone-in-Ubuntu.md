---
layout: post
title: "Changing the Time Zone in Ubuntu!"
date: 2024-08-30 14:06:00 +0200
categories: self-hosted homelab
tags: homelab ubuntu cli

---

# Changing the Time Zone in Ubuntu

This guide will assist you in changing the time zone on your Ubuntu system. Changing the time zone may be necessary when you move to a new location or manage servers located in different time zones.


## Requirements

- Administrative rights (access to the `root` account or the ability to use the `sudo` command).
- Access to a terminal/console in the Ubuntu system.


## Checking the Current Time Zone

To check what time zone is currently set on your Ubuntu system, execute the following command in the terminal:

```bash
timedatectl
```

You will receive information about the current date, time, and set time zone.


## List of Available Time Zones

To display a list of all available time zones that you can set, use the command:

```bash
timedatectl list-timezones
```

You can also search for a specific time zone using `grep`, for example:

```bash
timedatectl list-timezones | grep Europe
```


## Changing the Time Zone

To change the time zone, use the command `sudo timedatectl set-timezone`, followed by the appropriate time zone from the list. For example, to set the time zone to "Europe/Warsaw", execute:

```bash
sudo timedatectl set-timezone Europe/Warsaw
```

After executing this command, the time zone configuration will be immediately updated.

## Confirming the Change

To ensure that the time zone has been changed correctly, run the command again:

```bash
timedatectl
```

Check in the output if the time zone is now set according to your expectations.

## Problems and Troubleshooting

If you encounter any problems while changing the time zone, ensure you have the proper permissions (sudo) and that the entered time zone is available in the system (check the list of available time zones).