---
title: nginx + php-fpm + php-apc + mysql + wordpress perfect way to run a blog
author: naut
layout: post
date: 2013-09-27
url: /post/nginx-php-fpm-php-apc-mysql-wordpress-perfect-way-to-run-a-blog/
categories:
  - how to
  - interesting
tags:
  - debian
  - Linux
  - nginx
  - php-fpm
  - wp nginx

---
i recently moved from apache2 to nginx on all my web servers. and feeling happy about that.

i would like to share configurations since , before and after the move i did load test on my blog and difference on performance was amazing.

<!--more-->

lets consider that we already have a server (VM or not) with running Debian linux. and LAMP with WP on board.

so all we need to do its to migrate from apache2 to nginx.

so simple steps are starting of moving apache from port 80 to 81 , to have it somewhere , in case of emergency roll back.

check your http.conf and vhosts configs and do not forget about ports.conf. move everything to 81 restart apache, check if 80 port is free by netstat -nlptu |grep 80, check that apache is on 81 by netstat -nlptu |grep 81, try to open your WP with http://domain:81 and if everything works, we can go forward and start to configure nginx.

so for nginx we will need to use precompiled package from nginx.org or dotdeb or to compile it ourselves. I did compilation in my case, but here we will go with package from dotdeb (more options compiled).

edit your /etc/apt/source.list and add

    deb http://www.deb-multimedia.org stable main non-free
    deb http://packages.dotdeb.org squeeze all
    deb-src http://packages.dotdeb.org squeeze all
    deb http://nginx.org/packages/debian/ squeeze nginx
    deb-src http://nginx.org/packages/debian/ squeeze nginx

i&#8217;m using debian 6 (did not have time to upgrade to 7 :)) ) so consider of changing squeeze to wheezy in case of you&#8217;re using debian 7, version can be found here cat /etc/debian_version

so after just usual steps of adding key from foreign repository

    apt-get update && apt-get install deb-multimedia-keyring
    wget http://www.dotdeb.org/dotdeb.gpg -O- |apt-key add -
    gpg --keyserver hkp://keys.gnupg.net --recv-keys ABF5BD827BD9BF62
    gpg -a --export 7BD9BF62 | apt-key add -
    

so now dotdeb is trusted and we can do apt-get update and apt-get upgrade, i&#8217;m sure that some php/mysql and apache packages will get updated since debian 6 have old version.

so now we are ready to install php5-fpm , memcache (to keep php sessions in it), php-apc for caching, nginx itself <img src="http://i1.wp.com/nazaretyan.com/wp-includes/images/smilies/icon_smile.gif?w=625" alt=":)" class="wp-smiley" data-recalc-dims="1" />

    apt-get install php5-fpm nginx php5-apc memcached php5-memcached php5-dev 

so we can test our nginx installation by trying to open the http://domain/
  
if nginx default page is in place so we are on the right way.

first thing that we will need to fix its a bug in the file /etc/php5/fpm/php.ini ,

so open it with your favorite editor and fine the line cgi.fix_pathinfo and change value to 0 .

after this part is done, we can move on.

now all we need is to have nginx to work with WP (so php will work via php5-fpm).

    server {
    listen       80;
    server_name  domane.name wwwdomainname etc;
    root /var/www/domain/;
    
    access_log  /var/log/nginx/domain.access.log ;
    
    location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass   unix:/tmp/wwwpool.sock;
    fastcgi_cache  one;
    fastcgi_cache_min_uses 3;
    fastcgi_cache_valid 200 301 302 304 5m;
    fastcgi_cache_key "$request_method|$host|$request_uri";
    fastcgi_index  index.php;
    include fastcgi_params;
    fastcgi_param       SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    fastcgi_ignore_client_abort     off;
    }
    
    location / {
    root   /var/www/domain/;
    index  index.php;
    
    try_files $uri $uri/ /index.php?$args;
    }
    
    }

&nbsp;

here you will need to be careful for

     fastcgi_pass   unix:/tmp/wwwpool.sock 

since this you need to define in php5-fpm config. in /etc/php5/fpm


