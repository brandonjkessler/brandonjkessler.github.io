---
title: McAfee Corrupted EFI Partition
Author: Brandon J. Kessler
tags: DevOps SysAdmin Windows
category: Technology
published: true
---

<h1>{{ page.title }}</h1>

## The Problem

We just migrated from McAfee Endpoint Encryption to Bitlocker at work, and a few of our devices had the following error popup and failed to boot:

```
Failed to deactivate [0xEE00000C] Failed to remove EPE bootloader, 0xEE00000C   Unexpected exception in EpeUefiBootcode.cpp: [0xEE12000F] Failed to find EE Partition.
```

What we discovered is that McAfee corrupted the EFI partition. So we booted in to WinPE to fix the issue.
<!--more-->
## The Fix

NOTE: Before running any of these, confirm what DISK and PARTITION your EFI partition is on.

1.  Boot to WinPE
2.  Type the Following:
    1.  `diskpart`
        1.  `select disk 0`
        2.  `select partition 1`
        3.  `delete partition override`
        4.  `create partition efi`
        5.  `format quick fs=fat32 label="System"`
        6.  `assign letter=S`
        7.  `exit`
    2.  `bcdboot C:\Windows`
    3.  `exit`

The device should now boot to Windows and you can continue encrypting with Bitlocker! We created a batch file that pulls these commands from a .txt file that we run off of a thumb drive in WinPE. We titled the Batch File and the Text File “Fix-McAfeeEFI” and you can find their contents below.

### Batch File

```
diskpart /s %~dp0Fix-McAfeeEFI.txt

bcdboot C:\Windows
```

### Text File

```
select disk 0
select partition 1
delete partition override
create partition efi
format quick fs=fat32 label="System"
assign letter=S
```