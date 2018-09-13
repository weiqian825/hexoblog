---
title: 'nginx 本地部署及问题'
date: 2018-09-07 20:15:35
tags:
---

admin项目的nginx配置需要更改的时候，怎么样验证配置是否ok呢？

## 一、nginx常用命令

```
// 1. start
nginx
// 2. 停止nginx
nginx -s stop
nginx -s quit
// 3. 重载nginx配置
nginx -s reload
nginx -s quit
// 4. 指定配置的文件
nginx -c /usr/local/nginx/nginx.conf
// 5. 检查配置文件是否正确
nginx -t 
// 6. 帮助信息
nginx -h
nginx -?
// 7. 查看nginx版本
nginx -v
nginx -V
// 8 使用信号
// 8.1 获得pid
ps aux | grep nginx
cat /path/to/nginx.pid
// 8.2从容停止nginx，等所有结束后关闭服务
ps aux | grep nginx
kill -QUIT pid
// 8.3 nginx快速停止命令，立刻关闭进程
ps aux | grep nginx
kill -TERM pid
// 8.4 强制停止命令
kill -9 pid
// 9. 查看占用端口的pid
lsof -i :8080
lsof -i :端口号| grep LISTEN | awk '{ print $2; }' | head -n 2 | grep -v PID
```

## 二、更改的配置文件

1. 更改前，有hub的配置
```
# envs: ["test", "uat", "staging", "live"]
# cids: ["id", "th"]

server {
    listen       80;
    server_name  c-api-dp.{{ domain_env_flag }}shopee.{{ domain_suffix }};
    access_log   off;

    location / {
        proxy_pass              http://dp_api_{{ env }}_{{ cid }};
        include                 django_proxy_params;
        include                 gzip_params;
        proxy_intercept_errors  on;
    }
}

{% if cid == 'id' %}

server {
    listen       80;
    server_name  dp-admin.{{ domain_env_flag }}shopee.io;

    location / {
        return   307  https://$host$request_uri;
    }
}

server {
    listen       443 ssl http2;
    server_name  dp-admin.{{ domain_env_flag }}shopee.io;

    ## default page ##

    location / {
      try_files     $uri      /digital-purchase/index.html;
    }

    ## end default page ##

    ## Start of hub_admin ##

    location /hub/static/ {
        alias                   /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/hub/static/;
        expires                 30d;
        access_log              off;

        include                 gzip_params;
    }

    location /hub {
        try_files     $uri      /hub/index.html;
    }

    location = /hub/index.html {
        alias                 /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/hub/index.html;
        expires               -1;
        etag                  on;
        include               gzip_params;
    }

    ## End of hub_admin ##

    ## Start of dp_admin_portal ##

    location /digital-purchase/static/ {
        alias                   /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/windrunner/static/;
        expires                 30d;
        access_log              off;

        include                 gzip_params;
    }

    location /digital-purchase {
        try_files     $uri      /digital-purchase/index.html;
    }

    location = /digital-purchase/index.html {
        alias                 /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/windrunner/index.html;
        expires               -1;
        etag                  on;
        include               gzip_params;
    }

    ## End of dp_admin_portal ##

    ## Start of dp_admin_api ##

    location /api/ {
        proxy_pass            http://dp_admin_{{ env }}_{{ cid }};
        include               django_proxy_params;
        include               gzip_params;
    }

    ## End of dp_admin ##
}

{% endif %}

```
2. 更改后，去掉hub的配置
```
# envs: ["test", "uat", "staging", "live"]
# cids: ["id", "th"]

server {
    listen       80;
    server_name  c-api-dp.{{ domain_env_flag }}shopee.{{ domain_suffix }};
    access_log   off;

    location / {
        proxy_pass              http://dp_api_{{ env }}_{{ cid }};
        include                 django_proxy_params;
        include                 gzip_params;
        proxy_intercept_errors  on;
    }
}

{% if cid == 'id' %}

server {
    listen       80;
    server_name  dp-admin.{{ domain_env_flag }}shopee.io;

    location / {
        return   307  https://$host$request_uri;
    }
}

server {
    listen       443 ssl http2;
    server_name  dp-admin.{{ domain_env_flag }}shopee.io;

    ## Start of hub_admin ##

    location /hub/static/ {
        alias                   /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/hub/static/;
        expires                 30d;
        access_log              off;

        include                 gzip_params;
    }

    location /hub {
        try_files     $uri      /hub/index.html;
    }

    location = /hub/index.html {
        alias                 /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/hub/index.html;
        expires               -1;
        etag                  on;
        include               gzip_params;
    }

    ## End of hub_admin ##

    ## Start of dp_admin_portal ##

    location /static/ {
        alias                   /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/windrunner/static/;
        expires                 30d;
        access_log              off;

        include                 gzip_params;
    }

    location / {
        try_files     $uri      /index.html;
    }

    location = /index.html {
        alias                 /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/windrunner/index.html;
        expires               -1;
        etag                  on;
        include               gzip_params;
    }

    ## End of dp_admin_portal ##

    ## Start of dp_admin_api ##

    location /api/ {
        proxy_pass            http://dp_admin_{{ env }}_{{ cid }};
        include               django_proxy_params;
        include               gzip_params;
    }

    ## End of dp_admin ##
}

{% endif %}

```
## 三、本地验证nginx配置是否正确

在项目的deploy文件下，新建 nginx.conf，内容如下
```
user  root staff;
worker_processes  1;

events {
  worker_connections  3003;  ## Default: 1024
}

http {
  server {
    listen 8080;

    ## Start of admin_api ##

    location ^~ /api/ {
        proxy_pass            http://localhost:3003;
    }

    ## End of admin_api ##

    ## Start of admin ##

    location /static{
        alias                    /Users/weiqian/Desktop/shopee/Windrunner/build/windrunner/static/;
        expires                 30d;
        access_log              off;

    }

    location / {
        try_files     $uri       /index.html;
    }
    ## End of admin ##

  }
}

```
执行如下脚本
```
Windrunner git:(feature-ui) lsof -i :8080
➜  Windrunner git:(feature-ui) nginx -c /Users/weiqian/Desktop/shopee/Windrunner/deploy/nginx.conf
➜  Windrunner git:(feature-ui) nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful

```
放问网页 http://localhost:8080 可以正常访问，也就是说我们本地的配置是ok的。

## 四、线上访问验证

线上访问页面还是出错，js路径不对，那么就是说打包出来的静态文件可能不对，查看jenkins的日志报错如下
```
> node scripts/build.js

{ Error: Command failed: npm run oldbuild

Warning: DISABLE_ESLINT is deprecated, use ESLINT=none instead has been deprecated.
TypeError: Invalid data, chunk must be a string or buffer, not undefined
    at Socket.write (net.js:701:11)
    at nodeDeprecate (/workspace/node_modules/deprecate/index.js:22:22)
    at deprecate (/workspace/node_modules/deprecate/index.js:11:3)
    at Object.<anonymous> (/workspace/node_modules/af-webpack/lib/getConfig.js:81:26)
    at Module._compile (module.js:635:30)
    at Object.Module._extensions..js (module.js:646:10)
    at Module.load (module.js:554:32)
    at tryModuleLoad (module.js:497:12)
    at Function.Module._load (module.js:489:3)
    at Module.require (module.js:579:17)
    at require (internal/module.js:11:18)
    at Object.<anonymous> (/workspace/node_modules/af-webpack/getConfig.js:1:18)
    at Module._compile (module.js:635:30)
    at Object.Module._extensions..js (module.js:646:10)
    at Module.load (module.js:554:32)
    at tryModuleLoad (module.js:497:12)
```
线上有报错，最后显示build成功了，如果是这样，我们本地的build会不会也有问题，验证了下ok
```
rm -rf node_modules
rm -rf build
npm i
npm run build
```
报错如下
```
> ant-design-pro@1.3.0 build /Users/weiqian/Desktop/shopee/Windrunner
> node scripts/build.js

{ Error: Command failed: npm run oldbuild

Warning: DISABLE_ESLINT is deprecated, use ESLINT=none instead has been deprecated.
TypeError [ERR_INVALID_ARG_TYPE]: The "chunk" argument must be one of type string or Buffer. Received type undefined
    at validChunk (_stream_writable.js:259:10)
    at Socket.Writable.write (_stream_writable.js:293:21)
    at nodeDeprecate (/Users/weiqian/Desktop/shopee/Windrunner/node_modules/deprecate/index.js:22:22)
    at deprecate (/Users/weiqian/Desktop/shopee/Windrunner/node_modules/deprecate/index.js:11:3)
    at Object.<anonymous> (/Users/weiqian/Desktop/shopee/Windrunner/node_modules/af-webpack/lib/getConfig.js:81:26)
    at Module._compile (internal/modules/cjs/loader.js:678:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:689:10)
    at Module.load (internal/modules/cjs/loader.js:589:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:528:12)
    at Function.Module._load (internal/modules/cjs/loader.js:520:3)
    at Module.require (internal/modules/cjs/loader.js:626:17)
    at require (internal/modules/cjs/helpers.js:20:18)
    at Object.<anonymous> (/Users/weiqian/Desktop/shopee/Windrunner/node_modules/af-webpack/getConfig.js:1:18)
    at Module._compile (internal/modules/cjs/loader.js:678:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:689:10)
    at Module.load (internal/modules/cjs/loader.js:589:32)
[graceful-process#4201] exit with code:1
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! ant-design-pro@1.3.0 oldbuild: `cross-env DISABLE_ESLINT=none roadhog build`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the ant-design-pro@1.3.0 oldbuild script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/weiqian/.npm/_logs/2018-09-07T10_31_10_452Z-debug.log

    at ChildProcess.exithandler (child_process.js:282:12)
    at ChildProcess.emit (events.js:182:13)
    at maybeClose (internal/child_process.js:947:16)
    at Socket.stream.socket.on (internal/child_process.js:368:11)
    at Socket.emit (events.js:182:13)
    at Pipe._handle.close [as _onclose] (net.js:598:12) killed: false, code: 1, signal: null, cmd: 'npm run oldbuild' }
```
npm的package.json配置
```
"scripts": {
    "oldbuild": "cross-env DISABLE_ESLINT=none roadhog build",
    "build": "node scripts/build.js"
}
```
更改配置如下
```
"scripts": {
    "oldbuild": "cross-env ESLINT=none roadhog build",
    "build": "node scripts/build.js"
}
```
编译如下成功
```
➜  Windrunner git:(feature-ui) ✗ npm run build

> ant-design-pro@1.3.0 build /Users/weiqian/Desktop/shopee/Windrunner
> node scripts/build.js


> ant-design-pro@1.3.0 oldbuild /Users/weiqian/Desktop/shopee/Windrunner
> cross-env ESLINT=none roadhog build

Compiled successfully.

File sizes after gzip:

  694.35 KB  windrunner/index.fccc1fd9.js
  44.58 KB   windrunner/index.4c60e904.css


Successful!
```
本地build成功了，我们放到了jenkins试了下果然编译成功了，访问线上页面是ok的。

## 五、其他案例记录
把域名 https://dp-admin.test.shopee.io/ 换成 https://dp-admin.test.shopee.io/id
```
user  root staff;
worker_processes  1;

events {
  worker_connections  3003;  ## Default: 1024
}

http {
  server {
    listen 8080;

    ## Start of admin_api ##

    location ^~ /api/ {
        proxy_pass            http://localhost:3003;
    }

    ## End of admin_api ##

    ## Start of admin ##

    location /static {
        alias                    /Users/weiqian/Desktop/shopee/Windrunner/build/windrunner/static/;
        expires                 30d;
        access_log              off;

    }

    location /id {
        try_files     $uri       /index.html;
    }


    location = /index.html {
        alias                 /Users/weiqian/Desktop/shopee/Windrunner/build/windrunner/index.html;
        expires               -1;
        etag                  on;
        include               gzip_params;
    }
    ## End of admin ##
  }
}
```

执行代码 

```
➜  Windrunner git:(test) lsof -i :8080
COMMAND     PID    USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Google    23206 weiqian  149u  IPv4 0xd18ff623edf3bb69      0t0  TCP localhost:59974->localhost:http-alt (ESTABLISHED)
Google    23206 weiqian  202u  IPv4 0xd18ff623e145a789      0t0  TCP localhost:59936->localhost:http-alt (ESTABLISHED)
nginx     35357 weiqian    8u  IPv4 0xd18ff623dafe64c9      0t0  TCP *:http-alt (LISTEN)
nginx     35555 weiqian    7u  IPv4 0xd18ff623d9afbb69      0t0  TCP localhost:http-alt->localhost:59936 (ESTABLISHED)
nginx     35555 weiqian    8u  IPv4 0xd18ff623dafe64c9      0t0  TCP *:http-alt (LISTEN)
nginx     35555 weiqian   10u  IPv4 0xd18ff623f9b5f789      0t0  TCP localhost:http-alt->localhost:59974 (ESTABLISHED)
➜  Windrunner git:(test) kill -9 23206 35357 35555
➜  Windrunner git:(test) nginx -c /Users/weiqian/Desktop/shopee/Windrunner/deploy/nginx.conf
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /Users/weiqian/Desktop/shopee/Windrunner/deploy/nginx.conf:1
nginx: [emerg] open() "/Users/weiqian/Desktop/shopee/Windrunner/deploy/gzip_params" failed (2: No such file or directory) in /Users/weiqian/Desktop/shopee/Windrunner/deploy/nginx.conf:38
```
emerg报错gzip_params改正下nginx配置，屏蔽掉gzip_params
```
 location = /index.html {
        alias                 /Users/weiqian/Desktop/shopee/Windrunner/build/windrunner/index.html;
        expires               -1;
        etag                  on;
        #include              gzip_params;
    }
```
重新起下服务试试
```
➜  Windrunner git:(test) nginx -s stop
nginx: [alert] kill(35357, 15) failed (3: No such process)
➜  Windrunner git:(test) nginx -c /Users/weiqian/Desktop/shopee/Windrunner/deploy/nginx.conf
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /Users/weiqian/Desktop/shopee/Windrunner/deploy/nginx.conf:1
```
也是ok的呢。









