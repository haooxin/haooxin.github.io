---
layout: post
title: "Intel NIC e1000e hardware unit hang"
date: 2025-05-30 12:26:00 +0200
categories: linux
tags: homelab proxmox linux cli e1000e
image:
  path: /assets/img/headers/proxmox2.png
  lqip: 

---

## **Hardware**

I’m running **Proxmox VE** on the following hardware:

**HP ProDesk 600 G6**
- Intel Core i5-10500T
- 64 GB DDR4 RAM
- Storage:
    - 2x **WD Red SN700 1TB NVMe SSDs**, configured as a **ZFS mirror** (RAID-1) for both the Proxmox system and VM storage.
    - Additionally, a **Samsung 870 EVO 2TB SATA SSD** used for extra storage.

The networking runs on the **integrated Intel I219LM Gigabit Ethernet NIC**, bridged using `vmbr0` so the virtual machines can share the host network.

---

## **e1000e Driver Hang Issue**

Intel’s **e1000e** driver (used for I219, I210, and similar NICs) sometimes runs into a hardware hang where the card stops transmitting or receiving packets until it’s physically reset (e.g., unplugging/replugging the cable).

---

### **Example Log Snippet**

Here’s what the kernel log typically looks like when this problem happens (you can see it with `dmesg` or in `/var/log/syslog`):


```
May 13 13:56:59 hostname kernel: e1000e 0000:00:1f.6 eno1: Detected Hardware Unit Hang:   
    TDH                  <42>   
    TDT                  <6b>   
    next_to_use          <6b>   
    next_to_clean        <41> 
buffer_info[next_to_clean]:   
    time_stamp           <101debc3e>   
    next_to_watch        <42>  
    jiffies              <101dec140>   
    next_to_watch.status <0> 
MAC Status             <80083> 
PHY Status             <796d> 
PHY 1000BASE-T Status  <3800> 
PHY Extended Status    <3000> 
PCI Status             <10>
```

This indicates the card’s transmit queue has stalled — it’s unable to clean up or send packets.

---

## How to Read *Hardware Unit Hang* Logs

### 1. Log header

`e1000e 0000:00:1f.6 eno1: Detected Hardware Unit Hang`

The **e1000e** driver (Intel PRO/1000) has detected that the network-card hardware transmit unit (TX unit) on PCI bus **0000:00:1f.6** – interface **eno1** – has “hung,” meaning it stopped emptying its packet queue. The same pattern appears in Red Hat guides and kernel.org bug reports. [Red Hat Customer Portal](https://access.redhat.com/solutions/63042) [bugzilla.kernel.org](https://bugzilla.kernel.org/show_bug.cgi?id=118721)

---

### 2. Transmit-queue pointers

```
TDH <42>
TDT <6b>
next_to_use <6b>
next_to_clean <41>
```

|Pointer|Role|Hex value (dec)|Meaning here|
|---|---|---|---|
|**TDH**|Head seen by hardware|0x42 (66)|Hardware stopped on descriptor 66|
|**TDT**|Tail set by driver|0x6b (107)|Driver queued packets up to 107|
|**next_to_use**|Driver’s internal counter|0x6b (107)|Matches TDT – driver finished filling|
|**next_to_clean**|First descriptor that **should** return|0x41 (65)|Last cleaned packet was 65|

The difference **TDT – TDH = 0x29 (41)** shows 41 descriptors waiting – the card is not processing data, hence the “hang.”

---

### 3. Details of the blocked packet’s descriptor

```
buffer_info[next_to_clean]:
    time_stamp <101debc3e>
    next_to_watch <42>
    jiffies <101dec140>
    next_to_watch.status <0>
```

- **time_stamp** – moment the packet was queued (jiffies). 
- **jiffies** – current kernel timer. The gap of ~1282 ticks ≈ 5 s (with HZ = 250) is well beyond the safety threshold. 
- **next_to_watch.status = 0** – hardware never set the “DD” (Descriptor Done) bit, so the packet was not sent.

---

### 4. MAC-layer status

`MAC Status <80083>`

Rejestr **STATUS** kontrolera:

- **bit 0 (0x1) – Link Up** – link is active.
- **bit 1 (0x2) – FDX** – full duplex
- **bit 7-19 (0x80000)** – "Tx/Rx unit stalled" for many 8257x/219 families.  
    Summary: card sees 1 Gbps FDX link, but reports its own jamming. [bugzilla.kernel.org](https://bugzilla.kernel.org/show_bug.cgi?id=118721)

---

### 5. PHY layer

```
PHY Status            <796d> 
PHY 1000BASE-T Status <3800> 
PHY Extended Status   <3000>
```

The hex codes confirm:
- **Autonegotiation complete & link OK**
- **Operating at 1000 Mb/s**
- No transmitter/receiver physical errors.

Thus the physical link is healthy; the issue is higher, in the TX unit.

---

### 6. PCI status

`PCI Status <10>`

0x0010 → **Master Data Parity Error Cleared**. This is a standard bit cleared by the driver after reading; it does not indicate a fresh bus error.

---

### 7. Overall conclusions

1. **Transmit unit is stuck** – hardware does not advance TDH even though the driver added descriptors.
2. Link and PHY are fine; the stall occurs in the DMA/off-load logic.
3. The kernel notices the delay (≥ 5 s) and logs “Detected Hardware Unit Hang”; a soft adapter reset usually follows. [Proxmox Support Forum](https://forum.proxmox.com/threads/e1000e-eno1-detected-hardware-unit-hang.59928/)

---

### 8. Likely causes and next steps


|Most common cause|Quick work-arounds|
|---|---|
|Off-load bug (TSO/GSO/GRO) in certain 82579/219 revisions and the e1000e driver|`ethtool -K eno1 tso off gso off gro off`|
|EEE / ASPM combined with VLAN bridges|add kernel params `pcie_aspm=off e1000e.SmartPowerDownEnable=0`|
|Old NVM firmware|update BIOS/NVM (Intel Boot Util or OEM tool)|
|Card assigned to wrong NUMA node|load module with `modprobe e1000e Node=0`|

According to kernel.org reports, disabling **TSO** or all acceleration usually eliminates hangs, albeit at the expense of performance. If that doesn't help, a BIOS/firmware update or NIC replacement remains. [bugzilla.kernel.org](https://bugzilla.kernel.org/show_bug.cgi?id=118721)

The log breaks down into (1) header indicating the hang, (2) queue indicators showing that the card is not starting from descriptor 66, (3) lock details, (4-6) registers confirming that the physical link is good. The most likely culprit is a bug in the off-loads; the first diagnostic step is to disable them with the **ethtool -K** command or update the driver/firmware.

---
---

## **Fixing or Mitigating the e1000e Hang**

There are a few approaches to mitigate this issue.

---

### **1. Apply Module Parameters**

First, try adding a module parameter that’s known to reduce hangs.
Edit (or create) the file:

```
/etc/modprobe.d/e1000e.conf
```

Add:

```
options e1000e InterruptThrottleRate=0,0 TxIntDelay=0 RxIntDelay=0
```

Then rebuild initramfs and reload the module:

```
update-initramfs -u modprobe -r e1000e modprobe e1000e
```

Or simply **reboot**.

---

### **2️. Force Speed and Duplex**

Sometimes, negotiation issues can cause hangs. You can **force the speed** and **duplex** (make sure the switch side matches):

```
ethtool -s eno1 speed 1000 duplex full autoneg off
```

⚠ **Note:** This may disable Wake-on-LAN (WOL), since WOL often relies on autonegotiation.

---

### **3. Update Drivers or Kernel**

Make sure you’re on the **latest kernel** or install the **Intel out-of-tree drivers** from:

- [Intel Download Center](https://downloadcenter.intel.com)
  
The in-kernel driver sometimes lags behind Intel’s latest releases.

---

### **4. Disable Power Saving Features**

In BIOS/UEFI:
- Disable **Extended Idle Power States**,
- Disable **PCIe ASPM** (Active State Power Management).

This reduces the chance of power-saving interactions freezing the NIC.
I disabled all power saving features in BIOS.
In Linux, you can also disable Energy-Efficient Ethernet:

```
ethtool --set-eee eno1 eee off
```

---

### **5. Hardware Solution**

If none of the above works, **consider adding a dedicated PCIe network card or USB adapter** — they have better separation from the main chipset and are more robust under load.

---

### **6. Disable NIC Offloads**

Below I will explain what offloads are and show how to improve network card stability by disabling offloads.

---
---

## **Understanding Network Offloads**

Network interface cards (NICs) are not just “dumb pipes” passing data — they have hardware capabilities designed to **reduce CPU workload** by handling some tasks in hardware.  These are called **offloads**.

---

### **What Are Offloads?**

Offloads are hardware-assisted features that offload work from the CPU to the NIC, such as:

- calculating checksums (for TCP, UDP, IP),
- segmenting large packets into smaller ones (TSO, GSO),
- combining small received packets into larger buffers (GRO, LRO),
- inserting or stripping VLAN tags.

By default, most Linux systems enable these to **maximize performance**.

---

### **Why Disable Offloads?**

There are specific situations where you might want to **turn off offloads**:  
- When doing network debugging or packet captures (e.g., tcpdump),  
- When using virtualized networking (e.g., Proxmox, KVM bridges), 
- When testing for hardware or driver-related problems,  
- When dealing with non-standard protocols that offloads can’t handle.

Turning off offloads forces the **CPU** to handle all packet processing, making sure you get an exact view of the traffic and reducing the chance of offload-related bugs.

---

### **Example Command to Disable Offloads**

You can use the following command to disable key offloads on a network interface (replace `eno1` with your interface name):

```
ethtool -K eno1 gso off gro off tso off tx off rx off rxvlan off txvlan off sg off
```

|Flag|Meaning|
|---|---|
|gso off|Disable Generic Segmentation Offload|
|gro off|Disable Generic Receive Offload|
|tso off|Disable TCP Segmentation Offload|
|tx off|Disable transmit checksum offload|
|rx off|Disable receive checksum offload|
|rxvlan off|Disable VLAN RX tag offload|
|txvlan off|Disable VLAN TX tag offload|
|sg off|Disable scatter-gather I/O offload|

### Important Note: Persisting Changes

The `ethtool` command only applies changes at runtime — **they are not persistent across reboots**. To make sure the settings apply after every restart, you should:

1. Create a systemd service (example `/etc/systemd/system/ethtool-offload.service`):

```
[Unit]
Description=Disable offloads on eno1
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ethtool -K eno1 gso off gro off tso off tx off rx off rxvlan off txvlan off sg off

[Install]
WantedBy=multi-user.target
```

2. Enable the service:

```
sudo systemctl daemon-reexec && sudo systemctl daemon-reload && sudo systemctl enable ethtool-offload.service
```

This way, every time the system boots, the offload settings will be applied automatically.

### Automation Script

There is also a script that automates the process of disabling network interface card (NIC) offloading features specifically for Intel e1000e network interfaces on Linux systems. You can find an example script here:

[Proxmox VE Helper-Scripts Community - NIC Offloading Fix](https://community-scripts.github.io/ProxmoxVED/scripts?id=nic-offloading-fix)

---

## **Final Notes**

The combination of:  
- disabling problematic offloads,  
- tuning driver parameters,  
- updating firmware,  
- adjusting BIOS settings

usually leads to a **stable e1000e NIC** under Linux.
