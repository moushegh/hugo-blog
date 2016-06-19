---
title: how to change hostname in linux without restart
author: naut
layout: post
date: 2010-04-09
url: /post/how-to-change-hostname-in-linux-without-restart/
categories:
  - how to
tags:
  - Linux

---
its quite simple ..

you need just to echo new hostname into /proc/sys/kernel/hostname

for exmaple &#8211;

echo $NEWHOSTNAME > /proc/sys/kernel/hostname

logoff and login again to host <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

but remeber to add the new hostname to /etc/hosts as well

