---
title: etcd基础操作
date: 2021-06-30 23:14:07
tags:
---


## 本机安装 etcd（grpc 服务本地注册）

1. 安装 etcd

```
brew install etcd
```

2. 运行 etcd

```
brew services start etcd
# 其他 etcd 命令
brew services stop etcd
brew services list
```

## 项目安装和启动

1. 安装依赖

```
npm install
```

2. 启动项目

```
npm run dev:vn
```
