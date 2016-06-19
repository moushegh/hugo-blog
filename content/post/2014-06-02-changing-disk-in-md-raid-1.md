---
title: changing disk in MD RAID 1
author: "Moushegh Nazaretyan"
layout: post
date: 2014-06-02
url: /post/changing-disk-in-md-raid-1/
categories:
  - how to
tags:
  - gdisk
  - how to
  - linux raid
  - mdraid
  - parted
  - raid
  - software raid

---
Short Article about how to change disk on software raid, hot and cold swap.


<!--more-->

### USED TOOLS

```
apt-get update
apt-get install hdparm gdisk parted
```

### DISABLING DISK BEFORE CHANGE 

```(note that MD RAID mode must be 1, if it 0 you will kill system on this step)```

see which partition is binded to which MD device

```cat /proc/mdstat ```

output must be like, where (F) indicates that partition is marked as faulty


```
Personalities : [raid1]
 md3 : active raid1 sda4[2](F) sdb4[1]
     1822442815 blocks super 1.2 [2/1] [_U]
 md2 : active raid1 sda3[2](F) sdb3[1]
     1073740664 blocks super 1.2 [2/1] [_U]
```

set partition as faulty for particular MD device

```mdadm --manage --set-faulty /dev/mdX /dev/sdX1```

output must be similar to

```
mdadm: set /dev/sda2 faulty in /dev/md1
```

repeat same step for all partitions for particular drive

after removing all partitions from broken disk, mdstat output must be similar to

```
Personalities : [raid1]
md3 : active raid1 sdb4[1]
     1822442815 blocks super 1.2 [2/1] [_U]
md2 : active raid1 sdb3[1]
     1073740664 blocks super 1.2 [2/1] [_U]
md1 : active raid1 sdb2[1]
     524276 blocks super 1.2 [2/1] [_U]
md0 : active raid1 sdb1[1]
     33553336 blocks super 1.2 [2/1] [_U]
```

now since we set all pratitions of broken disk to faulty &#8211; we can remove them from md raid

```mdadm --manage /dev/mdX -r /dev/sdX1```

output must be similar to

```mdadm: hot removed /dev/sdX1 from /dev/mdX```

repeat same setp for all partitions for particualr drive

mdstat must look similar to this

```
Personalities : [raid1]
md3 : active raid1 sdb4[1]
     1822442815 blocks super 1.2 [2/1] [_U]
md2 : active raid1 sdb3[1]
     1073740664 blocks super 1.2 [2/1] [_U]
md1 : active raid1 sdb2[1]
     524276 blocks super 1.2 [2/1] [_U]
md0 : active raid1 sdb1[1]
     33553336 blocks super 1.2 [2/1] [_U]
unused devices: &lt;none&gt;
```

if all steps are kept correct, you can now force linux to remove hard drive (in case you want to do HOT swap)

```echo 1 > /sys/block/sdX/device/delete```

where X is actual broken drive letter

``` dmsg ```

output must be like this

```
[383287.786029] sd 0:0:0:0: [sda] Stopping disk
```

congratz you manage to remove disk without killing system :) go smoke now.

### ADDING NEW DISK HOT SWAP SCENARIO (without reboot)

in case HDD was HOT swaped (without system restart) you need to force system to rescan controller and detect new drive

```echo "0 0 0" > /sys/class/scsi_host/host<n>/scan```

where <n> is host controller number

```
[383444.439888] ata1: hard resetting link
[383444.756112] ata1: SATA link up 3.0 Gbps (SStatus 123 SControl 300)
[383445.042099] ata1.00: ATA-8: ST3000DM001-9YN166, CC4C, max UDMA/133
[383445.042139] ata1.00: 5860533168 sectors, multi 0: LBA48 NCQ (depth 31/32), AA
[383445.072228] ata1.00: configured for UDMA/133
[383445.072270] ata1: EH complete
[383445.072467] scsi 0:0:0:0: Direct-Access     ATA      ST3000DM001-9YN1 CC4C PQ: 0 ANSI: 5
[383445.072834] sd 0:0:0:0: [sda] 5860533168 512-byte logical blocks: (3.00 TB/2.72 TiB)
[383445.072846] sd 0:0:0:0: Attached scsi generic sg0 type 0
[383445.072924] sd 0:0:0:0: [sda] 4096-byte physical blocks
[383445.073364] sd 0:0:0:0: [sda] Write Protect is off
[383445.073402] sd 0:0:0:0: [sda] Mode Sense: 00 3a 00 00
[383445.073587] sd 0:0:0:0: [sda] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
[383445.122528]  sda: sda1 sda2 sda3 sda4 sda5
[383445.124355] sd 0:0:0:0: [sda] Attached SCSI disk
```

so now linux found your new replaced hard drive and its time for partitioning.

### ADDING NEW DISK COLD SWAP SCENARIO (with reboo)

service providers like "hetzner" are not supporting hot swap, so they always need to reboot system in order to change HDD,since we deleted hdd from system. 

its now safe to shutdown and ask provider to swap disk. after swap they will switch on system back. disk will get same drive letter as old &#8220;broken&#8221; one.

### PREPARING DISK FOR MD RAID MEMBERSHIP

if disk is bigger then 2TB &#8211; it must use GPT. where Y is drive letter for new changed HDD

```
parted /dev/sdY
mklabel GPT
yes
quit
```

we will need to recreate same partition table on newly chagned drive (VERY VERY CAREFULY)

```sgdisk -R=/dev/sdX /dev/sdY```

where X is exisiting drive in MD array and Y is new changed drive

```sgdisk -G /dev/sdY```

The second command randomizes the GUID on the disk and all the partitions (MUST for uninqness)

if everything went without errors on this stage, you are rady to add disk back to MD RAID.

### ADDING DISK BACK TO RUNNING MDRAID

to see which partition belongs to which MD device

```watch cat /proc/mdstat```

then

```mdadm --manage /dev/mdX -a /dev/sdX1```

output

```mdadm: added /dev/sdX1```

MDRAID will start to sync drives in order to have data on both drives in parallel, you can follow it by

```cat /proc/mdstat```

output

```
Personalities : [raid1]
md3 : active raid1 sda4[2] sdb4[1]
     1822442815 blocks super 1.2 [2/1] [_U]
     [===========&gt;.........]  recovery = 57.2% (1043526976/1822442815) finish=211.8min speed=61278K/sec
md2 : active raid1 sda3[2] sdb3[1]
     1073740664 blocks super 1.2 [2/1] [_U]
       resync=DELAYED
md1 : active raid1 sda2[2] sdb2[1]
     524276 blocks super 1.2 [2/1] [_U]
       resync=DELAYED
md0 : active raid1 sda1[2] sdb1[1]
     33553336 blocks super 1.2 [2/1] [_U]
       resync=DELAYED
unused devices: &lt;none&gt;
```

where ```resync=DELAYED``` means that sync is in ```wait state``` , only 1 MD device can sync in same time.

### ADDING BOOT LOADER

after all the steps you need to tell grub that new disk is bootlbe also

```grub-install /dev/sdY```

output

```
Installation finished. No error reported
```

