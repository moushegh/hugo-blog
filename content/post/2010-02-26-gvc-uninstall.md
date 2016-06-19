---
title: GVC uninstall
author: "Moushegh Nazaretyan"
layout: post
date: 2010-02-26
url: /post/gvc-uninstall/
categories:
  - how to
tags:
  - GVC
  - Sonicwall
  - vpn

---
sometimes when youre trying to uninstall sonicwall global VPN client youre getting error that is not installed on the machine, but after that when youre trying to install the new one its gives error that old version is intalled,

<!--more-->here is easy way to remove it.

<span style="color: #ffffff;">In a DOS command window type the following: </span>

<span style="color: black;"><span style="color: #ffffff;">C:\> </span><strong><span style="color: #ffffff;">net stop rcfox</span></strong></span>

<span style="color: black;"><span style="color: #ffffff;">Then go into the Windows\System32\drivers directory and rename </span><strong><span style="color: #ffffff;">rcfox.sys</span></strong><span style="color: #ffffff;"> to </span><strong><span style="color: #ffffff;">rcfox.sys.old</span></strong></span>

<span style="color: black;"><span style="color: #ffffff;">Run the installer</span><br /> </span>

<span style="color: #ffffff;">hope it helps.</span>

y