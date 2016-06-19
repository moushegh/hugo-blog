---
title: changing disk in MD RAID 1
author: naut
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
# <span id="USED_TOOLS" class="mw-headline">USED TOOLS </span>

<pre>apt-get update
apt-get install hdparm gdisk parted
</pre>

#  <span id="DISABLING_DISK_BEFORE_CHANGE" class="mw-headline">DISABLING DISK BEFORE CHANGE </span>

<pre><span style="color: #ff0000;">(note that MD RAID mode must be 1, if it 0 you will kill system on this step)
</span></pre>

see which partition is binded to which MD device

<pre>cat /proc/mdstat  
  
</pre>

output must be like, where (F) indicates that partition is marked as faulty

<pre>Personalities : [raid1]
 md3 : active raid1 sda4[2](F) sdb4[1]
     1822442815 blocks super 1.2 [2/1] [_U]
 md2 : active raid1 sda3[2](F) sdb3[1]
     1073740664 blocks super 1.2 [2/1] [_U]
</pre>

set partition as faulty for particular MD device

<pre>mdadm --manage --set-faulty /dev/mdX /dev/sdX1
</pre>

output must be similar to

<pre>mdadm: set /dev/sda2 faulty in /dev/md1
</pre>

repeat same step for all partitions for particular drive

after removing all partitions from broken disk, mdstat output must be similar to

<pre>Personalities : [raid1]
md3 : active raid1 sdb4[1]
     1822442815 blocks super 1.2 [2/1] [_U]
md2 : active raid1 sdb3[1]
     1073740664 blocks super 1.2 [2/1] [_U]
md1 : active raid1 sdb2[1]
     524276 blocks super 1.2 [2/1] [_U]
md0 : active raid1 sdb1[1]
     33553336 blocks super 1.2 [2/1] [_U]
</pre>

now since we set all pratitions of broken disk to faulty &#8211; we can remove them from md raid

<pre>mdadm --manage /dev/mdX -r /dev/sdX1
</pre>

output must be similar to

<pre>mdadm: hot removed /dev/sdX1 from /dev/mdX
</pre>

repeat same setp for all partitions for particualr drive

mdstat must look similar to this

<pre>Personalities : [raid1]
md3 : active raid1 sdb4[1]
     1822442815 blocks super 1.2 [2/1] [_U]
md2 : active raid1 sdb3[1]
     1073740664 blocks super 1.2 [2/1] [_U]
md1 : active raid1 sdb2[1]
     524276 blocks super 1.2 [2/1] [_U]
md0 : active raid1 sdb1[1]
     33553336 blocks super 1.2 [2/1] [_U]
unused devices: &lt;none&gt;
</pre>

if all steps are kept correct, you can now force linux to remove hard drive (in case you want to do HOT swap)

<pre>echo 1 &gt; /sys/block/sdX/device/delete 
</pre>

where X is actual broken drive letter

<pre>dmsg
</pre>

output must be like this

<pre>[383287.786029] sd 0:0:0:0: [sda] Stopping disk
</pre>

congratz you manage to remove disk without killing system &#8211; go smoke now.

# <span id="ADDING_NEW_DISK_HOT_SWAP_SCENARIO_.28without_reboot.29" class="mw-headline">ADDING NEW DISK HOT SWAP SCENARIO (without reboot)</span>

in case HDD was HOT swaped (without system restart) you need to force system to rescan controller and detect new drive

<pre>echo "0 0 0" &gt;/sys/class/scsi_host/host&lt;n&gt;/scan
</pre>

where <n> is host controller number

<pre>[383444.439888] ata1: hard resetting link
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
</pre>

so now linux found your new replaced hard drive and its time for partitioning.

#  <span id="ADDING_NEW_DISK_COLD_SWAP_SCENARIO_.28with_reboot.29" class="mw-headline">ADDING NEW DISK COLD SWAP SCENARIO (with reboot)</span>

service providers like &#8220;hetzner&#8221; are not supporting hot swap, so they always need to reboot system in order to change HDD,since we deleted hdd from system. its now safe to shutdown and ask provider to swap disk. after swap they will switch on system back. disk will get same drive letter as old &#8220;broken&#8221; one.

# <span id="PREPARING_DISK_FOR_MD_RAID_MEMBERSHIP" class="mw-headline">PREPARING DISK FOR MD RAID MEMBERSHIP</span>

if disk is bigger then 2TB &#8211; it must use GPT. where Y is drive letter for new changed HDD

<pre>parted /dev/sdY
mklabel GPT
yes
quit
</pre>

we will need to recreate same partition table on newly chagned drive (VERY VERY CAREFULY)

<pre>sgdisk -R=/dev/sdX /dev/sdY
</pre>

where X is exisiting drive in MD array and Y is new changed drive

<pre>sgdisk -G /dev/sdY
</pre>

The second command randomizes the GUID on the disk and all the partitions (MUST for uninqness)

if everything went without errors on this stage, you are rady to add disk back to MD RAID.

# <span id="ADDING_DISK_BACK_TO_RUNNING_MDRAID" class="mw-headline">ADDING DISK BACK TO RUNNING MDRAID</span>

to see which partition belongs to which MD device

<pre>watch cat /proc/mdstat
</pre>

then

<pre>mdadm --manage /dev/mdX -a /dev/sdX1
</pre>

output

<pre>mdadm: added /dev/sdX1
</pre>

MDRAID will start to sync drives in order to have data on both drives in parallel, you can follow it by

<pre>cat /proc/mdstat
</pre>

output

<pre>Personalities : [raid1]
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
</pre>

where &#8220;resync=DELAYED&#8221; means that sync is in &#8220;wait state&#8221; &#8211; only 1 MD device can sync in same time.

#  <span id="ADDING_BOOT_LOADER" class="mw-headline">ADDING BOOT LOADER</span>

after all the steps you need to tell grub that new disk is bootlbe also

<pre>grub-install /dev/sdY
</pre>

output

<pre>Installation finished. No error reported
</pre>

have fun.

<div class="sharedaddy sd-sharing-enabled">
  <div class="robots-nocontent sd-block sd-social sd-social-icon-text sd-sharing">
    <h3 class="sd-title">
      Share this:
    </h3>
    
    <div class="sd-content">
      <ul>
        <li>
          <a href="#" class="sharing-anchor sd-button share-more"><span>Share</span></a>
        </li>
        <li class="share-end">
        </li>
      </ul>
      
      <div class="sharing-hidden">
        <div class="inner" style="display: none;">
          <ul>
            <li class="share-email">
              <a rel="nofollow" data-shared="" class="share-email sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=email" title="Click to email this to a friend"><span>Email</span></a>
            </li>
            <li class="share-facebook">
              <a rel="nofollow" data-shared="sharing-facebook-259" class="share-facebook sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=facebook" title="Share on Facebook"><span>Facebook</span></a>
            </li>
            <li class="share-end">
            </li>
            <li class="share-reddit">
              <a rel="nofollow" data-shared="" class="share-reddit sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=reddit" title="Click to share on Reddit"><span>Reddit</span></a>
            </li>
            <li class="share-twitter">
              <a rel="nofollow" data-shared="sharing-twitter-259" class="share-twitter sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=twitter" title="Click to share on Twitter"><span>Twitter</span></a>
            </li>
            <li class="share-end">
            </li>
            <li class="share-tumblr">
              <a rel="nofollow" data-shared="" class="share-tumblr sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=tumblr" title="Click to share on Tumblr"><span>Tumblr</span></a>
            </li>
            <li class="share-linkedin">
              <a rel="nofollow" data-shared="sharing-linkedin-259" class="share-linkedin sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=linkedin" title="Click to share on LinkedIn"><span>LinkedIn</span></a>
            </li>
            <li class="share-end">
            </li>
            <li class="share-stumbleupon">
              <a rel="nofollow" data-shared="" class="share-stumbleupon sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=stumbleupon" title="Click to share on StumbleUpon"><span>StumbleUpon</span></a>
            </li>
            <li class="share-pinterest">
              <a rel="nofollow" data-shared="sharing-pinterest-259" class="share-pinterest sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=pinterest" title="Click to share on Pinterest"><span>Pinterest</span></a>
            </li>
            <li class="share-end">
            </li>
            <li class="share-google-plus-1">
              <a rel="nofollow" data-shared="sharing-google-259" class="share-google-plus-1 sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/?share=google-plus-1" title="Click to share on Google+"><span>Google</span></a>
            </li>
            <li class="share-print">
              <a rel="nofollow" data-shared="" class="share-print sd-button share-icon" href="https://nazaretyan.com/changing-disk-in-md-raid-1/" title="Click to print"><span>Print</span></a>
            </li>
            <li class="share-end">
            </li>
            <li class="share-end">
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</div>

<div class='sharedaddy sd-block sd-like jetpack-likes-widget-wrapper jetpack-likes-widget-unloaded' id='like-post-wrapper-39401321-259-5766a03cef057' data-src='//widgets.wp.com/likes/#blog_id=39401321&post_id=259&origin=nazaretyan.com&obj_id=39401321-259-5766a03cef057' data-name='like-post-frame-39401321-259-5766a03cef057'>
  <h3 class='sd-title'>
    Like this:
  </h3>
  
  <div class='likes-widget-placeholder post-likes-widget-placeholder' style='height:55px'>
    <span class='button'><span>Like</span></span> <span class="loading">Loading...</span>
  </div>
  
  <span class='sd-text-color'></span><a class='sd-link-color'></a>
</div>