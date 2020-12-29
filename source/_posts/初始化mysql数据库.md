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
//设置密码
//a)为root用户设置密码
//b)删除匿名账号
//c)取消root用户远程登录
//d)删除test库和对test库的访问权限
//e)刷新授权表使修改生效
mysql_secure_installation

```

二、初始化数据库

1. 数据库操作语句

```
mysql -u root -p
//显示全部数据库
show databases;
//创建名为 new 的数据库
create database db1;
//删除名为 db1 的数据库
drop database db1;
//选择 db1 数据库
use db1;
```

2. 表操作语句

```
// 当前数据库存在什么表
show tables
```

3. 构造测试数据

```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for tab1
-- ----------------------------
DROP TABLE IF EXISTS `tab1`;
CREATE TABLE `tab1` (
  `col1` varchar(255) DEFAULT NULL,
  `col2` varchar(255) DEFAULT NULL,
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=latin1;

-- ----------------------------
-- Records of tab1
-- ----------------------------
BEGIN;
INSERT INTO `tab1` VALUES ('db1_tab1_col1_1', 'db1_tab1_col2_1', 1);
INSERT INTO `tab1` VALUES ('db1_tab1_col1_2', 'db1_tab1_col2_2', 2);
COMMIT;

-- ----------------------------
-- Table structure for tab2
-- ----------------------------
DROP TABLE IF EXISTS `tab2`;
CREATE TABLE `tab2` (
  `col1` varchar(255) DEFAULT NULL,
  `col2` varchar(255) DEFAULT NULL,
  `col3` varchar(255) DEFAULT NULL,
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=latin1;

-- ----------------------------
-- Records of tab2
-- ----------------------------
BEGIN;
INSERT INTO `tab2` VALUES ('db1_tab2_col1_1', 'db1_tab2_col2_1', 1, 'db1_tab2_col3_1');
INSERT INTO `tab2` VALUES ('db1_tab2_col1_2', 'db1_tab2_col2_2', 2, 'db1_tab2_col3_2');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

四、问题

1. 解决 MAC+ MySQL 8 错误：Authentication plugin 'caching_sha2_password' cannot be loaded

```
不是客户端Navicat的原因，是MySQL兼容问题，需要修改数据库的认证方式
MySQL8.0版本默认的认证方式是caching_sha2_password
MySQL5.7版本则为mysql_native_password。

官方文档：
https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password
```
