---
title: How to shutdown how if init6, reboot or other shutdown method do not work because of hdd, or other system problems
author: naut
layout: post
date: 2010-04-09
url: /post/how-to-shutdown-how-if-init6-reboot-or-other-shutdown-method-do-not-work-because-of-hdd-or-other-system-problems/
categories:
  - how to
tags:
  - debain
  - IT support
  - Linux
  - reset
  - restart

---
If you have ever had a hard drive fail on a remote server you may remember the feeling you had after trying to issue the following commands:

<!--more-->

\# reboot
  
bash: /sbin/reboot: Input/output error
  
\# shutdown -r now
  
bash: /sbin/shutdown: Input/output error

Obviously, there is a problem with your drive. These commands are failing because the kernel is unable to load the /sbin/reboot and /sbin/shutdown binaries from the disk so that it can execute them.

A fsck on the next boot might be able to correct whatever is wrong with the disk, but first you need to get the system to reboot. If your machine is located at a managed hosting provider then you could submit a reboot ticket, but you&#8217;ll have to wait for someone to take responsibility.

Wouldn&#8217;t it be nice if there was a way to ask the kernel to reboot without needing to access the failing drive? Well, there is a way, and it is remarkably simple.

The “magic SysRq key” provides a way to send commands directly to the kernel through the /proc filesystem. It is enabled via a kernel compile time option, CONFIG\_MAGIC\_SYSRQ, which seems to be standard on most distributions. First you must activate the magic SysRq option:

echo 1 > /proc/sys/kernel/sysrq

When you are ready to reboot the machine simply run the following:

echo b > /proc/sysrq-trigger

This does not attempt to unmount or sync filesystems, so it should only be used when absolutely necessary, but if your drive is already failing then that may not be a concern.

In addition to rebooting the system the sysrq trick can be used to dump memory information to the console, sync all filesystems, remount all filesystems in read-only mode, send SIGTERM or SIGKILL to all processes except init, or power off the machine entirely, among other things.

Also, instead of echoing into /proc/sys/kernel/sysrq each time you can activate the magic SysRq key at system boot time using sysctl, where supported:

echo &#8220;kernel.sysrq = 1&#8243; >> /etc/sysctl.conf

If you would like to learn more about magic SysRq you can read the sysrq.txt file in the kernel documentation.

