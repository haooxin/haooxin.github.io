---
layout: post
title: "How to install and configure Samba server on linux"
date: 2025-02-16 10:20:00 +0200
categories: linux
tags: homelab ubuntu linux cli samba smb
image:
  path: /assets/img/headers/Linux-NAS2.webp
  lqip: 

---


---

## Simple Instalation

---

### 1. Install Samba

**1. Update the package list:**

```bash
sudo apt update
```

**2. Install the Samba package:**

```bash
sudo apt install samba
```

---

### 2. Configure the Shared Resource

**1. Create the directory you want to share:**

```bash
sudo mkdir -p /srv/resources
```

**2. Set the owner and group for the directory (for example, owner: `root` and group: `users`):**

```bash
sudo chown -R root:users /srv/resources
```

**3. Assign the appropriate permissions (example: `2770` to allow access only for the owner and group):**

```bash
sudo chmod -R 2770 /srv/resources
```

---

### 3. Edit the Samba Configuration File

**1. Open the Samba configuration file in a text editor (e.g., using nano):**

```bash
sudo nano /etc/samba/smb.conf
```

**2. Add a new section at the end of the file for the shared resource. A sample configuration might look like this:**

```ini
[resources]
   comment = Shared resources folder
   path = /srv/resources
   browseable = yes
   writable = yes
   guest ok = no
   valid users = @users
```

- `path` – The path to the shared directory.
- `browseable` – Determines whether the share is visible when browsing the network.
- `writable` – Allows writing to the share.
- `guest ok` – Disables guest access (access is only for defined users).
- `valid users` – Specifies the users or groups that are allowed access (in this case, the group users).

**3.Save the changes and exit the editor (in nano: press `Ctrl+O`, then `Enter`, and `Ctrl+X` to exit).**

---

### 4. Configure Samba Users

**1. Add a system user (if the user does not already exist):**

```bash
sudo adduser username
```

**2. Add the user to the group that has access to the share (e.g., `users`):**

```bash
sudo usermod -aG users username
```

**3. Add the user to the Samba database and set a Samba password:**

```bash
sudo smbpasswd -a username
```

- You will be prompted to enter and confirm a password for the Samba user.

---

### 5. Restart Samba Services and Test the Configuration

**1. Restart the Samba services to load the new settings:**

```bash
sudo systemctl restart smbd
sudo systemctl restart nmbd
```

**2. Check the configuration file for any errors:**

```bash
testparm
```

- The `testparm` command will help detect any issues or syntax errors in your `smb.conf` file.

---

### 6. Additional Permission Settings

- Setting Read-Only Access:
  - To share the resource as read-only, add the following line in the share definition:

```ini
read only = yes
```

- Enabling Write Access:
  - To allow writing to the share, ensure the following line is set:

```ini
writable = yes
```

- Restricting Write Access to Specific Users:
  - You can limit write access to a selected list of users using the `write list` option:

```ini
write list = user1, user2
```

**Example of a share with limited write access:**

```ini
[shared]
   comment = Shared folder with limited write access
   path = /srv/shared
   browseable = yes
   read only = no
   valid users = user1, user2, @group1
   write list = user1, user2
```

---

### 7. Troubleshooting

If issues arise, review the logs in `/var/log/samba/` and ensure that:

- System users and Samba accounts are correctly configured.
- [Troubleshoot problems accessing shared folders on Windows.](https://learn.microsoft.com/pl-pl/troubleshoot/windows-client/networking/cannot-access-shared-folder-file-explorer)

---

### 8. Summary

After completing the above steps, your Samba server should be properly configured. System users added to the Ubuntu system and the Samba database will have access to the shared resources according to the permissions set in the configuration. To test access, you can connect to the share from a Windows machine or another network client using the appropriate credentials.

Feel free to adjust the configuration and directory permissions as needed for your specific requirements.

---
---

## Advanced Instalation

### 1. System Preparation and Package Installation

#### A. Update the System and Install Packages

**1. Update your system:**

```bash
sudo apt update && sudo apt upgrade -y
```

**2. Install Samba and additional tools:**

```bash
sudo apt install samba samba-common-bin winbind libnss-winbind libpam-winbind krb5-user -y
```

#### B. Description of Installed Tools

- **Samba:**
The core package that enables file and printer sharing over the SMB/CIFS protocol. It allows a Linux server to share resources with Windows systems and other SMB clients.

- **samba-common-bin:**
Contains additional utilities and helper scripts (such as `smbclient` and `testparm`) essential for managing and testing your Samba configuration.

- **Winbind:**
A service that retrieves user and group information from Windows systems (Active Directory or NT servers) and integrates it with your Linux system. Winbind helps to integrate Windows accounts into your Linux environment.

- **libnss-winbind:**
A library that integrates the Name Service Switch (NSS) with Winbind, allowing Linux to fetch user and group information from Active Directory as if they were local accounts.

- **libpam-winbind:**
A PAM (Pluggable Authentication Module) that enables authentication of Linux users against Active Directory accounts. This module supports single sign-on (SSO) in mixed environments.

- **krb5-user:**
Contains the tools and configuration files for the Kerberos client. It is essential for integrating with Active Directory, which uses the Kerberos protocol for authentication and ticket management.

**3. Backup the Default Samba Configuration:**

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

---

### 2. Global Samba Configuration

Open the Samba configuration file in your favorite text editor:

```bash
sudo nano /etc/samba/smb.conf
```

**Example Advanced [global] Section**

```ini
[global]
   # Basic Settings
   workgroup = MYGROUP
   netbios name = UBUNTUSERVER
   server string = Advanced Samba Server on Ubuntu
   security = user
   map to guest = Bad User
   dns proxy = no

   # Logging Options
   log file = /var/log/samba/%m.log
   max log size = 1000
   log level = 2

   # Performance and Socket Options
   socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF=8192
   deadtime = 15
   max smbd processes = 50

   # Extended Attributes and ACL Support (requires filesystem support)
   vfs objects = acl_xattr
   map acl inherit = yes
   store dos attributes = yes

   # Active Directory Integration (uncomment if needed)
   ;security = ADS
   ;realm = EXAMPLE.COM
   ;workgroup = EXAMPLE
   ;idmap config * : backend = tdb
   ;idmap config * : range = 3000-7999
   ;idmap config EXAMPLE : backend = rid
   ;idmap config EXAMPLE : range = 10000-999999
   ;winbind use default domain = yes
   ;winbind offline logon = yes

   # Disable printer support if not used
   load printers = no
   printing = bsd
   printcap name = /dev/null
```

Notes:

- `vfs objects = acl_xattr` allows storing extended attributes (such as ACLs) on the filesystem. Ensure your filesystem (e.g., ext4) is mounted with ACL support (add the `acl` option in `/etc/fstab` if needed).

- The AD-related options (commented out) should be enabled when joining an Active Directory domain.

---

### 3. Advanced Share Configuration

---

#### A. Private Share with Access Control (ACLs and File Masks)

**1. Create the share directory:**

```bash
sudo mkdir -p /srv/private
```

**2. Set the owner, group, and permissions:**

```bash
sudo chown -R root:privategroup /srv/private
sudo chmod -R 2770 /srv/private
```

> - Make sure the group `privategroup` exists. You can create it with:
>
> ```bash
> sudo groupadd privategroup
> ```

**3. Add the share definition in `smb.conf`:**

```ini
[private]
   comment = Private share for authorized group members
   path = /srv/private
   browseable = no
   writable = yes
   valid users = @privategroup
   create mask = 0660
   directory mask = 2770
   force group = privategroup
   inherit permissions = yes
   veto files = /Thumbs.db/._*/
```

- `valid users = @privategroup` restricts access to users in the `privategroup`.
- `force group` ensures all created files and directories are assigned to the specified group.
- `inherit permissions` allows new files/directories to inherit permissions from the parent folder.

#### B. Public Share (Read-Only)

**1. Create the public directory:**

```bash
sudo mkdir -p /srv/public
sudo chown -R nobody:nogroup /srv/public
sudo chmod -R 0755 /srv/public
```

**2. Add the share definition:**

```ini
[public]
   comment = Public share (read-only)
   path = /srv/public
   browseable = yes
   read only = yes
   guest ok = yes
```

- `guest ok = yes` enables anonymous access to the share.
  
---

### 4. Filesystem ACL Configuration

If you are using a filesystem that supports ACLs (e.g., ext4), ensure the `acl` option is enabled during mount:

**1. Edit `/etc/fstab`:**

- Add the `acl` option to your partition's mount options, for example:

```bash
/dev/sda1   /   ext4   defaults,acl   0   1
```

**2. Remount the filesystem or reboot the system:**

```bash
sudo mount -o remount,acl /
```

**3. Example: Set additional ACLs on your private share:**

```bash
sudo setfacl -R -m u:username:rwx /srv/private
sudo getfacl /srv/private
```

---

### 5. Active Directory (AD) Integration – Optional

#### Step 1: Install and Perform Initial Configuration of Winbind

**1. Install Required Packages:**

Make sure you have the necessary packages for AD integration installed:

```bash
sudo apt install winbind libnss-winbind libpam-winbind krb5-user -y
```

- **winbind** – Retrieves user and group information from the domain.
- **libnss-winbind** – Integrates data from Winbind with the local user database.
- **libpam-winbind** – Allows AD user authentication through PAM.
- **krb5-user** – Contains Kerberos client tools and configuration, which are essential for AD integration.
  
**2. Initial Configuration of Samba (`/etc/samba/smb.conf`):**

Open the Samba configuration file:

```bash
sudo nano /etc/samba/smb.conf
```

In the `[global]` section, add or uncomment the following lines, adjusting them to your domain:

```ini
security = ADS
realm = EXAMPLE.COM
workgroup = EXAMPLE
idmap config * : backend = tdb
idmap config * : range = 3000-7999
idmap config EXAMPLE : backend = rid
idmap config EXAMPLE : range = 10000-999999
winbind use default domain = yes
winbind offline logon = yes
```

- **security** = ADS – Sets the security mode to Active Directory.
- **realm** – Your domain name (e.g., YOURDOMAIN.COM).
- **workgroup** – The workgroup name, usually matching your domain.
- **idmap config** – Settings for mapping user and group identifiers.
- **winbind use default domain = yes** – Displays usernames without the domain prefix.
- **winbind offline logon = yes** – Allows users to log in even if AD is temporarily unavailable.

#### Step 2: Join the Domain and Finalize the Configuration

**1. Join the Server to the Domain:**

Execute the following command to join the server to the domain:

```bash
sudo net ads join -U administrator
```

Enter the domain administrator’s password when prompted.

**2. Modify the /etc/nsswitch.conf File:**

Ensure that the lines for `passwd` and `group` include `winbind`, which enables retrieval of user and group information from AD:

```plaintext
passwd:         compat winbind
group:          compat winbind
```

**3. Restart the Services:**

To apply all changes, restart the Samba and Winbind services:

```bash
sudo systemctl restart smbd nmbd winbind
```

---

### 6. Printer Sharing Configuration – Optional

If you want your server to act as a print server:

**1. Create the spool directory:**

```bash
sudo mkdir -p /var/spool/samba
sudo chown -R root:lp /var/spool/samba
sudo chmod -R 1777 /var/spool/samba
```

**2. Add a printers share in smb.conf:**

```ini
[printers]
   comment = All Printers
   path = /var/spool/samba
   browseable = no
   guest ok = no
   writable = no
   printable = yes
```

**3. Disable automatic printer driver loading in the [global] section (if not required):**

```ini
load printers = no
printing = bsd
printcap name = /dev/null
```

---

### 7. Advanced Performance and Monitoring Options

#### A. Performance Tuning

- **Socket Options:**
Optimize network buffers:

```ini
socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF=8192
```

- **Deadtime:**
Automatically disconnect inactive sessions:

```ini
deadtime = 15
```

- **Maximum SMBD Processes:**
Limit the number of concurrent smbd instances:

```ini
max smbd processes = 50
```

#### B. Monitoring and Logging

- **Log Level:**
Increase log detail to help with troubleshooting:

```ini
log level = 2
```

- **Log File:**
Store logs for each client in separate files:

```ini
log file = /var/log/samba/%m.log
```

- **Log Analysis:**
Monitor logs in the `/var/log/samba/` directory using tools like `tail`:

```bash
tail -f /var/log/samba/UBUNTUSERVER.log
```

---

### 8. Testing and Deploying Changes

**1. Check the Configuration: Before restarting services, run:**

```bash
sudo testparm
```

This command checks the syntax of your `smb.conf` file and reports any errors.

**2. Restart Samba Services:**

```bash
sudo systemctl restart smbd nmbd winbind
```

**3. Test the Shares:**

- **From the Terminal:**
Use `smbclient` to list available shares:

```bash
smbclient -L localhost -U username
```

- **From a Windows or Other Client:**
Connect to the share using the UNC path (e.g., `\\UBUNTUSERVER\private`) and appropriate credentials.

---

### 9. Troubleshooting

If issues arise, review the logs in `/var/log/samba/` and ensure that:

- Your filesystem is mounted with ACL support (if using `acl_xattr`).
- System users and Samba accounts are correctly configured.
- For AD integration, Kerberos and Winbind are functioning properly.
- [Troubleshoot problems accessing shared folders on Windows.](https://learn.microsoft.com/pl-pl/troubleshoot/windows-client/networking/cannot-access-shared-folder-file-explorer)


---

### 10. Summary

In this advanced guide, we covered:

- **Installation and Tool Descriptions**: Detailed explanations of Samba, samba-common-bin, Winbind, libnss-winbind, libpam-winbind, and krb5-user.
- **Advanced Samba Configuration**: Global settings, configuration of private and public shares, and optional printer sharing.
- **Active Directory Integration**: Using Winbind and Kerberos for centralized authentication.
- **Performance Tuning and Monitoring**: Network buffer optimizations, logging options, and system monitoring.

This comprehensive configuration transforms your Samba server into a flexible, secure, and high-performance resource-sharing tool in a complex network environment. Remember to regularly monitor logs and update your system to maintain security and stability.

Feel free to adjust the settings in `smb.conf` to suit your infrastructure and test every change using `testparm` and other diagnostic tools.
