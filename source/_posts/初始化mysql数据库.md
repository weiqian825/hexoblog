---
title: 初始化mysql数据库
date: 2020-12-29 13:29:30
tags:
---

一、安装 mysql

```
//安装mysql
brew install mysql
//启动mysql
mysql.server start
//mysql安全设置（密码等）
mysql_secure_installation
//a)为root用户设置密码，建议设置成：hahaha@2020
//b)删除匿名账号
//c)取消root用户远程登录
//d)删除test库和对test库的访问权限
//e)刷新授权表使修改生效
//登陆mysql
mysql -u root -p
//显示全部数据库
show databases;

```

二、安装 Navicat，运行 SQL 文件

1. 下载 Mysql For Navicat： https://www.navicat.com.cn/manual/online_manual/cn/navicat/mac_manual/#/installation
2. [导入初始化 mysql 的文件](./20201204/db_portal_1204.sql)

三、修改 DB Portal 配置

```
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=hahaha@2020
DB_DATABASE=db_portal_dev
DB_HOST_MASTER=localhost
DB_HOST_SLAVE=localhost

```

四、常见问题

1.cmd 中遇到 MAC+ MySQL 8 错误：Authentication plugin 'caching_sha2_password' cannot be loaded
不是客户端 Navicat 的原因，是 MySQL 兼容问题，需要修改数据库的认证方式

```
//查看mysql的版本
SELECT VERSION();
//降级mysql的认证方式
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'hahaha@2020';
//刷新权限
flush privileges;
```

MySQL8.0 版本默认的认证方式是 caching_sha2_password
MySQL5.7 版本则为 mysql_native_password。

官方文档：
https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password

解决方案：降级 mysql 的认证方式为 mysql_native_password，因为当前版本的 node 不支持 caching_sha2_password 该认证方式。

https://stackoverflow.com/questions/50093144/mysql-8-0-client-does-not-support-authentication-protocol-requested-by-server

2. cmd 执行 show database 不生效，检查下没有有添加“ ；”
