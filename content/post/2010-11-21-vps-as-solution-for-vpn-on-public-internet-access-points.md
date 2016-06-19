---
title: vps as solution for VPN on public internet access points
author: naut
layout: post
date: 2010-11-21
url: /post/vps-as-solution-for-vpn-on-public-internet-access-points/
categories:
  - how to
  - interesting
tags:
  - amazon aws
  - free tunnel
  - Linux
  - OpenVPN
  - vpn

---
i think most that i want to write on this post is already tolden  in head  <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />but anyway

most people using laptops with &#8220;free&#8221; access points on fastfood points/ airports and other places.

most of this points have a limited traffic access to internet with lot of port filters.

but as a usual user with unsual ports  <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />i want always to access everyting from everywhere with same quality as from home

after searching in internet for solution trying my-freedom and other servies that giving only non full speed access i become to idea that its easier to have own VPN server running somewhere in cheep or almost free  hosting.

<!--more-->what we will need

1. we will need to register account on amazon AWS, for that we need to go to http://aws.amazon.com/free/ and register there, after we will get tear of amazon vps <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

2. we need to install virtual instance, i took the &#8220;US-East Micro Instance с Ubuntu 10.4&#8243; with 10gb hard drive , on community AMI&#8217;s you can find it with name _ami-c2a255ab._

_3._ when server is up and running we will need putty or (in my case just a terminal as i&#8217;m a mac user) for login to server.

4. we will need to have no-ip.com account as on every restart intance will get new ip address, how to configure no-ip.com for linux host you can find on this article https://help.ubuntu.com/community/DynamicDNS

at server we need to install OpenVPN. there is a lot of good manuals how to do it, but in sort words

we need to read http://library.linode.com/networking/openvpn/ubuntu-10.04-lucid this very good article about OpenVPN.

and try to start own one , with config file

<div id="_mcePaste">
  port 443
</div>

<div id="_mcePaste">
  proto udp
</div>

<div id="_mcePaste">
  dev tap
</div>

<div id="_mcePaste">
  ########################
</div>

<div id="_mcePaste">
  ca /etc/openvpn/ca.crt
</div>

<div id="_mcePaste">
  cert /etc/openvpn/server.crt
</div>

<div id="_mcePaste">
  key /etc/openvpn/server.key
</div>

<div id="_mcePaste">
  dh /etc/openvpn/dh1024.pem
</div>

<div id="_mcePaste">
  ##################################
</div>

<div id="_mcePaste">
  tls-server
</div>

<div id="_mcePaste">
  mode server
</div>

<div id="_mcePaste">
  duplicate-cn
</div>

<div id="_mcePaste">
  tun-mtu 1500
</div>

<div id="_mcePaste">
  tun-mtu-extra 32
</div>

<div id="_mcePaste">
  mssfix 1450
</div>

<div id="_mcePaste">
  ##################################
</div>

<div id="_mcePaste">
  server 172.0.1.0 255.255.255.0
</div>

<div id="_mcePaste">
  ifconfig-pool-persist clients.txt
</div>

<div id="_mcePaste">
  push &#8220;redirect-gateway&#8221;
</div>

<div id="_mcePaste">
  push &#8220;dhcp-option DNS 172.0.1.1&#8243;
</div>

<div id="_mcePaste">
  ############################
</div>

<div id="_mcePaste">
  client-to-client
</div>

<div id="_mcePaste">
  keepalive 10 120
</div>

<div id="_mcePaste">
  max-clients 100
</div>

<div id="_mcePaste">
  ###############################
</div>

<div id="_mcePaste">
  user root
</div>

<div id="_mcePaste">
  group root
</div>

<div id="_mcePaste">
  ##################################
</div>

<div id="_mcePaste">
  persist-key
</div>

<div id="_mcePaste">
  persist-tun
</div>

<div id="_mcePaste">
  status          /var/log/openvpn/status_server.log
</div>

<div id="_mcePaste">
  log             /var/log/openvpn/openvpn_server.log
</div>

<div id="_mcePaste">
  log-append      /var/log/openvpn/append_server.log
</div>

<div id="_mcePaste">
  verb 3
</div>

after the server is running, we need just to download windows/mac client and configure it on your laptop.

as you can see from config file my openvpn server is running on 443 (HTTPS) port, which is open almost everywhere,

if for some meters HTTPS is closed in point from where you want to connect you can try to use 53UDP (bind) port

from my own experience its open almost everywhere <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

ask questions/configure and enjoy your 15GB of free traffic from Amazon AWS. <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

y