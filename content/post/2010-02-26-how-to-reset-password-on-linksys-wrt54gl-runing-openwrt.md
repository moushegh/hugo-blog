---
title: how to reset password on Linksys WRT54GL running OpenWRT
author: "Moushegh Nazaretyan"
layout: post
date: 2010-02-26
url: /post/how-to-reset-password-on-linksys-wrt54gl-runing-openwrt/
categories:
  - how to
  - interesting
tags:
  - linksys
  - Linux
  - OpenWRT
  - root
  - router

---
post is mainly for people who just find out that cant remember the root password from OpenWRT operation system thats running on their Linksys WRT54GL.

what you need to do  <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />?

<!--more-->1. configure your static IP address on your ehternet to 192.168.1.3 netmask 255.255.255.0 , we dont need a gateway and dns servers,

2. connect Linksys LAN port N1 to your machine,

3. restart Linksys, and wait till DMZ light will become green. after it push the EasyConnect button (or reset button it doesn&#8217;t meter)

4. now your Linksys openWRT is runing in file_safe mode, you can telnet to it via 192.168.1.1 ( for telnet i&#8217;m recomending to use PuTTY (just google for it))

5. when youre inside , type mount_root, system will remount JFFS from RO to RW.

6.  now you can east type PASSWD and change root password.

hope this post helps you <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

i made same for one of my routers .. works perfect.

regards

