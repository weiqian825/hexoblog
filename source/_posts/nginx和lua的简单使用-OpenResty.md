---
title: nginx和lua的简单使用-openresty
date: 2018-10-11 00:09:58
tags:
---
[OpenResty](https://openresty.org/cn/) 是一款基于 NGINX 和 LuaJIT 的 Web 平台

## 一、安装环境
1. 根据官网教程，macos我们只需要一行命令就可以开心的安装openresty (brew install openresty/brew/openresty)

```
➜  sshkey brew untap homebrew/nginx
Error: No available tap homebrew/nginx.
➜  sshkey brew install openresty/brew/openresty
Updating Homebrew...
==> Tapping openresty/brew
Cloning into '/usr/local/Homebrew/Library/Taps/openresty/homebrew-brew'...
remote: Enumerating objects: 72, done.
remote: Counting objects: 100% (72/72), done.
remote: Compressing objects: 100% (71/71), done.
remote: Total 72 (delta 1), reused 32 (delta 1), pack-reused 0
Unpacking objects: 100% (72/72), done.
Tapped 62 formulae (162 files, 132.9KB).
==> Installing openresty from openresty/brew
==> Installing dependencies for openresty/brew/openresty: openresty/brew/openresty-openssl and geoip
==> Installing openresty/brew/openresty dependency: openresty/brew/openresty-openssl
==> Downloading https://www.openssl.org/source/openssl-1.1.0h.tar.gz
######################################################################## 100.0%
==> Downloading https://raw.githubusercontent.com/openresty/openresty/master/patches/openssl-1.1.0d-s
######################################################################## 100.0%
==> Patching
==> Applying openssl-1.1.0d-sess_set_get_cb_yield.patch
patching file include/openssl/bio.h
Hunk #1 succeeded at 217 (offset -3 lines).
patching file include/openssl/ssl.h
Hunk #1 succeeded at 812 (offset 21 lines).
Hunk #2 succeeded at 821 (offset 21 lines).
Hunk #3 succeeded at 1054 (offset 21 lines).
Hunk #4 succeeded at 1461 with fuzz 1 (offset 31 lines).
patching file ssl/bio_ssl.c
Hunk #1 succeeded at 139 (offset 1 line).
Hunk #2 succeeded at 215 (offset 1 line).
Hunk #3 succeeded at 372 (offset 1 line).
patching file ssl/ssl_lib.c
Hunk #1 succeeded at 3177 (offset 195 lines).
patching file ssl/ssl_sess.c
Hunk #2 succeeded at 512 (offset 8 lines).
Hunk #3 succeeded at 901 (offset 9 lines).
patching file ssl/statem/statem.c
Hunk #1 succeeded at 591 (offset 8 lines).
Hunk #2 succeeded at 622 (offset 8 lines).
patching file ssl/statem/statem.h
patching file ssl/statem/statem_srvr.c
Hunk #1 succeeded at 1147 (offset -18 lines).
patching file util/libssl.num
==> perl ./Configure --prefix=/usr/local/Cellar/openresty-openssl/1.1.0h_1 --openssldir=/usr/local/et
==> make
==> make install MANDIR=/usr/local/Cellar/openresty-openssl/1.1.0h_1/share/man MANSUFFIX=ssl
==> Caveats
openresty-openssl is keg-only, which means it was not symlinked into /usr/local,
because only for use with OpenResty.

If you need to have openresty-openssl first in your PATH run:
  echo 'export PATH="/usr/local/opt/openresty-openssl/bin:$PATH"' >> ~/.zshrc

For compilers to find openresty-openssl you may need to set:
  export LDFLAGS="-L/usr/local/opt/openresty-openssl/lib"
  export CPPFLAGS="-I/usr/local/opt/openresty-openssl/include"

For pkg-config to find openresty-openssl you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/openresty-openssl/lib/pkgconfig"

==> Summary
🍺  /usr/local/Cellar/openresty-openssl/1.1.0h_1: 6,583 files, 15.6MB, built in 5 minutes 43 seconds
==> Installing openresty/brew/openresty dependency: geoip
==> Downloading https://homebrew.bintray.com/bottles/geoip-1.6.12.high_sierra.bottle.tar.gz
######################################################################## 100.0%
==> Pouring geoip-1.6.12.high_sierra.bottle.tar.gz
🍺  /usr/local/Cellar/geoip/1.6.12: 18 files, 548.9KB
==> Installing openresty/brew/openresty
==> Downloading https://openresty.org/download/openresty-1.13.6.2.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/openresty/1.13.6.2 --pid-path=/usr/local/var/run/openresty
==> make
==> make install
==> Caveats
To have launchd start openresty/brew/openresty now and restart at login:
  brew services start openresty/brew/openresty
Or, if you don't want/need a background service you can just run:
  openresty
==> Summary
🍺  /usr/local/Cellar/openresty/1.13.6.2: 293 files, 6.3MB, built in 1 minute 26 seconds
==> Caveats
==> openresty-openssl
openresty-openssl is keg-only, which means it was not symlinked into /usr/local,
because only for use with OpenResty.

If you need to have openresty-openssl first in your PATH run:
  echo 'export PATH="/usr/local/opt/openresty-openssl/bin:$PATH"' >> ~/.zshrc

For compilers to find openresty-openssl you may need to set:
  export LDFLAGS="-L/usr/local/opt/openresty-openssl/lib"
  export CPPFLAGS="-I/usr/local/opt/openresty-openssl/include"

For pkg-config to find openresty-openssl you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/openresty-openssl/lib/pkgconfig"

==> openresty
To have launchd start openresty/brew/openresty now and restart at login:
  brew services start openresty/brew/openresty
Or, if you don't want/need a background service you can just run:
  openresty
➜  sshkey brew services start openresty/brew/openresty
==> Tapping homebrew/services
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-services'...
remote: Enumerating objects: 14, done.
remote: Counting objects: 100% (14/14), done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 14 (delta 0), reused 9 (delta 0), pack-reused 0
Unpacking objects: 100% (14/14), done.
Tapped 1 command (43 files, 55.6KB).
==> Successfully started `openresty` (label: homebrew.mxcl.openresty)
➜  sshkey sudo find / -name nginx
Password:
/usr/local/bin/nginx
/usr/local/etc/nginx
/usr/local/var/homebrew/linked/nginx
/usr/local/var/log/nginx
/usr/local/var/run/nginx
/usr/local/opt/nginx
/usr/local/Cellar/openresty/1.13.6.2/pod/nginx
/usr/local/Cellar/openresty/1.13.6.2/nginx
/usr/local/Cellar/openresty/1.13.6.2/nginx/sbin/nginx=====> this is you need add nginx path
/usr/local/Cellar/nginx
/usr/local/Cellar/nginx/1.15.5/bin/nginx
/usr/local/Cellar/nginx/1.15.5/.bottle/etc/nginx
/usr/local/Cellar/nginx/1.15.5/.bottle/var/log/nginx
^C
sudo vim ~/.bash_profile
➜  sshkey nginx -v
nginx version: nginx/1.15.5
➜  sshkey brew uninstall nginx
Uninstalling /usr/local/Cellar/nginx/1.15.5... (23 files, 1.4MB)
➜  sshkey nginx
zsh: command not found: nginx
➜  sshkey source ~/.bash_profile
➜  sshkey nginx -v
nginx version: openresty/1.13.6.2
➜  sshkey history | grep  nginx-https.conf
 2821* touch nginx-https.conf
 2822* vim nginx-https.conf
 2825* sudo  nginx -c  /Users/weiqian/sshkey/nginx-https.conf
 2831* sudo  nginx -c  /Users/weiqian/sshkey/nginx-https.conf
 2833* sudo  nginx -c  /Users/weiqian/sshkey/nginx-https.conf
 2834* vim nginx-https.conf
 2836* sudo  nginx -c  /Users/weiqian/sshkey/nginx-https.conf
 2847* open nginx-https.conf
➜  sshkey history | grep stop
 2594  nginx -s stop -c /usr/local/etc/nginx/nginx.conf
 2719* nginx -s stop
 2824* sudo nginx -s stop
 2832* sudo nginx -s stop
 2835* sudo nginx -s stop
➜  sshkey sudo nginx -s stop
nginx: [error] invalid PID number "" in "/usr/local/var/run/openresty.pid"
➜  sshkey sudo  nginx -t -c  /Users/weiqian/sshkey/nginx-https.conf
nginx: the configuration file /Users/weiqian/sshkey/nginx-https.conf syntax is ok
nginx: configuration file /Users/weiqian/sshkey/nginx-https.conf test is successful
➜  sshkey sudo  nginx  -c  /Users/weiqian/sshkey/nginx-https.conf
Password:
nginx: [emerg] bind() to 0.0.0.0:80 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:443 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:443 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:443 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:443 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (48: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:443 failed (48: Address already in use)
nginx: [emerg] still could not bind()
➜  sshkey sudo lsof -i 4tcp:8080
➜  sshkey sudo ps aux | grep nginx
weiqian          12641   1.8  0.0  4286184    920 s004  S+   12:59上午   0:00.01 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn nginx
root             90055   0.0  0.0  4299904   1404   ??  S    11:49下午   0:00.02 nginx: worker process
root             90054   0.0  0.0  4291212    200   ??  Ss   11:49下午   0:00.00 nginx: master process nginx -c /Users/weiqian/sshkey/nginx-https.conf
➜  sshkey sudo kill -9 90054
➜  sshkey sudo kill -9 90055
➜  sshkey sudo ps aux | grep nginx
weiqian          12677   0.0  0.0  4276968    900 s004  S+   12:59上午   0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn nginx
➜  sshkey sudo nginx -s stop
nginx: [error] invalid PID number "" in "/usr/local/var/run/openresty.pid"
➜  sshkey sudo  nginx  -c  /Users/weiqian/sshkey/nginx-https.conf
```
测试的nginx文件配置
```
user root owner;
worker_processes 1;
events {
 worker_connections 1024;
}
http{
  server{
    listen 80;
    server_name www.example.com;
    root /Users/weiqian/sshkey;
    location / {
      return   301  https://$host$request_uri;
    }
    location /test-lua {
      default_type text/html;
      content_by_lua '
        ngx.say("<p>hello, world</p>")
      ';
    }
  }

 server {
    
    listen 443 ssl;
    server_name www.example.com;
    root /Users/weiqian/sshkey;
    ssl_certificate       /Users/weiqian/sshkey/demoCA/www.example.com.crt; 
    ssl_certificate_key   /Users/weiqian/sshkey/demoCA/www.example.com.key; 
    location / {
      index index.html;
    }  
    location /produk-digital/m {
        try_files     $uri      /digital-product/m/index.html;
    }
    location = /digital-product/m/index.html {
        alias       /Users/weiqian/sshkey/index.html;
    }
  } 
}
```
访问http://www.example.com/test-lua页面显示hello world说明我们lua环境配置正确可以，开始编写lua脚本了

## 二、lua脚本配置关闭http链接，所有链接跳转到https
1. lua的nginx模块具体代码如下
```
access_by_lua_block {
    if ngx.var.scheme == 'http' then
        return ngx.redirect("https://" .. ngx.var.host .. ngx.var.request_uri)
    end
}
```
2. 修改后的本地测试实栗
```
user root owner;

worker_processes 1;

events {
 worker_connections 1024;
}

http{
 server {
    listen       80;
    listen       443 ssl http2; 
    server_name www.example.com;
    ssl_certificate       /Users/weiqian/sshkey/demoCA/www.example.com.crt; 
    ssl_certificate_key   /Users/weiqian/sshkey/demoCA/www.example.com.key; 

    ## begin of digital purchase ##

    location / {
        try_files     $uri      /digital-product/m/index.html;
    }
    location /produk-digital/m {
        try_files     $uri      /digital-product/m/index.html;
    }
    location /digital-product/m {
        try_files     $uri      /digital-product/m/index.html;
    }
    access_by_lua_block {
        if ngx.var.scheme == 'http' then
            return ngx.redirect("https://" .. ngx.var.host .. ngx.var.request_uri)
        end
    }
    location = /digital-product/m/index.html {
        alias                   /Users/weiqian/sshkey/index.html;
        access_log              off;
        add_header              Cache-Control "no-cache, no-store";
    }
    location /digital-product/static/ {
        alias                   /Users/weiqian/sshkey/;
        access_log              off;
        expires                 30d;
    }
    ## end of digital purchase #
  } 
}

```
3. 存在问题，我们刚才的配置将所有的域名都转到了https实际上我们只想改自己配置，不要影响别人的业务

```
user root owner;

worker_processes 1;

events {
 worker_connections 1024;
}

http{
 server {
    listen       80;
    listen       443 ssl http2; 
    server_name www.example.com;
    ssl_certificate       /Users/weiqian/sshkey/demoCA/www.example.com.crt; 
    ssl_certificate_key   /Users/weiqian/sshkey/demoCA/www.example.com.key; 

    ## begin of digital purchase ##

    location /produk-digital/m {
        try_files     $uri      /digital-product/m/index.html;
    }
    location /digital-product/m {
        try_files     $uri      /digital-product/m/index.html;
    }
   
    location = /digital-product/m/index.html {
        access_by_lua_block {
          if ngx.var.scheme == 'http' then
            return ngx.redirect("https://" .. ngx.var.host .. ngx.var.request_uri)
          end
        }
        alias                   /Users/weiqian/sshkey/index.html;
        access_log              off;
        add_header              Cache-Control "no-cache, no-store";
    }
    location /digital-product/static/ {
        alias                   /Users/weiqian/sshkey/;
        access_log              off;
        expires                 30d;
    }
    location /testlua/ {
       alias        /Users/weiqian/sshkey/testlua/;
       expires      30d;
    }
    ## end of digital purchase #

  } 
}
```






