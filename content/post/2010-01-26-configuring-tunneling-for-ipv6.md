---
title: configuring tunneling for ipv6
author: "Moushegh Nazaretyan"
layout: post
date: 2010-01-26
url: /post/configuring-tunneling-for-ipv6/
categories:
  - how to
  - interesting
tags:
  - iproute
  - iproute2
  - ipv4
  - ipv6
  - Linux
  - net-tools
  - route
  - tunnel

---
hey <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

again thanks for he.net for this wonderful appartunity to use IPv6 without own zone &#8230; blablabla <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

and here is some examples how to configure IPv6 tunnel.

lats start.

<!--more-->1.

<span style="color: #ff0000;">Linux with iproute </span>

<pre>modprobe ipv6
ip tunnel add new-ipv6 mode sit remote IP_ADDRESS local IP_ADDRESS ttl 255
ip link set new-ipv6 up
ip addr add IPv6_ADDRESS dev new-ipv6
ip route add ::/0 dev new-ipv6
ip -f inet6 addr

2.linux with net-tools



<pre>ifconfig sit0 up
ifconfig sit0 inet6 tunnel ::IPv4_LIKE_IPv6_ADDRESS
ifconfig sit1 up
ifconfig sit1 inet6 add IPv6_ADDRESS
route -A inet6 add ::/0 dev sit1</pre>


<p>
  3.Sun Solaris
</p>


<pre>ifconfig ip.tun0 inet6 plumb
ifconfig ip.tun0 inet6 tsrc IP_LOCAL tdst IP_REMOTE up
ifconfig ip.tun0 inet6 addif TUN_IPv6_ADDRESS_LOCAL TUN_IPv6_ADDRESS_REMOTE up
route add -inet6 default REMOTE_TUN_IPv6_ADDRESS</pre>


