---
title: ssh基础原理及操作
date: 2021-08-30 22:46:32
tags:
---

## SSH登陆验证
- ssh验证登陆
![](./images/ssh_login.png)

- jenkins ssh原理
![](./images/ssh_jenkins_login.png)

## SSH 正向连接

ssh -L sourcePort:forwardToHost:onPort connectToHost
意思：连接主机 connectToHost，监听本地的端口 sourcePort，通过主机 connectToHost，把到本地端口 sourcePort 的连接转发到主机 forwardToHost 的端口 onPort。

- ssh -L 123:localhost:456 remotehost
  连接主机 remotehost，监听本地的端口 123，通过 remotehost，把到本地端口 123 的连接转发到本地主机 localhost 的端口 456

- ssh -L 123:farawayhost:456 remotehost
  连接主机 remotehost，监听本地的端口 123，通过 remotehost，把到本地端口 123 的连接转发到主机 farawayhost 的端口 456
  ![](./images/ssh1.png)

## SSH 反向连接

- ssh -R 123:localhost:456 remotehost
  连接主机 remotehost，监听主机 remotehost 的端口 123，通过本地主机，把到主机 remotehost 的端口 123 的连接转发到本地主机 localhost 的端口 456。

- ssh -R 123:nearhost:456 remotehost
  连接主机 remotehost，监听主机 remotehost 的端口 123，通过本地主机，把到主机 remotehost 的端口 123 的连接转发到主机 nearhost 的端口 456
  ![](./images/ssh2.png)

## ProxyJump 用法

```
Host target    # 代表目标机器的名字
    HostName 目标机器 IP    # 这个是目标机器的 IP
    Port 22    # 目标机器 ssh 的端口
    User username_target    # 目标机器的用户名
    ProxyJump username@跳板机IP:port

Host 10.10.0.*    # 使用通配符 * 代表 10.10.0.1 - 10.10.0.255
    Port 22    # 服务器端口
    User username    # 服务器用户名
    ProxyJump username@跳板机IP:port
```

配置例子

```
Host basition
HostName 203.116.x.x
IdentityFile ~/.ssh/id_rsa

Host airpay_bastion_id
HostName 103.70.xx.xx
IdentityFile ~/.ssh/id_rsa
```

## 完整配置脚本 ~/.ssh/config

```
# 跳板机 203.116.x.x ｜ 103.70.xx.xx
Host basition
HostName 203.116.x.x
IdentityFile ~/.ssh/id_rsa

Host airpay_bastion_id
HostName 103.70.xx.xx
IdentityFile ~/.ssh/id_rsa

# k8s节点网段，也可以配置其他需要跳板机的机器
Host 10.65.8.*
ProxyCommand ssh basition -W %h:%p

Host 10.65.230.*
ProxyCommand ssh basition -W %h:%p

Host 10.62.197.*
ProxyCommand ssh airpay_bastion_id -W %h:%p

Host *
# 对于任何不明的host使用ld-qian.wei用户登陆
User ld-qian.wei
```

## 连接 python 脚本

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os
import sys

address = sys.argv[1].split(':')
ip = address[0]
port = address[1]

try:
  ssh_commond = 'ssh 10.65.x -vNL %s:%s:%s'%(port, ip, port)
  os.system(ssh_commond)
except OSError:
  sys.exit(1)
```
