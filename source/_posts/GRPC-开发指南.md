---
title: GRPC 开发指南
date: 2021-05-30 20:35:25
tags:
---



## 分享目标

1. 开发阶段：如何本地调试多个相互调用的 GRPC 服务？
2. 开发进阶：不同的 grpc-js 版本会对开发造成什么影响?
3. 发布阶段：GRPC 服务出了问题怎么办?

## 前置知识

### RPC

- RPC 定义: `远程过程调用`（Remote Procedure Call）是一个计算机`通信协议`，简单的理解是一个节点请求另一个节点提供的服务, 像调用本地服务一样调用远程服务。
  ![](./images/rpc.png)

- Q1: RPC 只是一种设计(一种概念), 如何实现远程过程调用(RPC)？
- A1: 采用什么通信协议(TCP|UDP|HTTP)? 数据传输格式是什么？

### GRPC = Google + RPC（js 版本@grpc/grpc-js）

- 传输协议: 基于 HTTP/2 协议标准而设计

```
Path : /Service-Name/{method name}
Service-Name : ?( {proto package name} "." ) {service name}
Message-Type : {fully qualified proto message name}
Content-Type : "application/grpc+proto"
```

- 序列化协议: 基于 ProtoBuf(Protocol Buffers) 序列化协议开发

gRPC 的 service 接口是基于 protobuf 定义的

```js
syntax = "proto3";
package fe.xxx.review;
service ActionReviewService {
  rpc GetTickets(GetTicketsReq) returns (GetTicketsResp) {}
}
message GetTicketsReq {
   int32 page_no = 1;
   int32 page_size = 2;
}
message GetTicketsResp {
  int32 page_no = 1;
  int32 page_size = 2;
  int32 total = 3;
  repeated TicketInfoFE ticket_info_fe = 4;
}
```

- Q1: GRPC 本质是对 http2.0 的一层封装，那么相互关联调用服务是否可以像 HTTP 服务一样互相调用？
- A1: 可以，写清楚服务调用的 IP(标记网络设备)和端口(区分主机的不同应用)等配置信息

### web 请求模型架构

- web1.0 单体应用
  ![](https://pic1.zhimg.com/80/v2-825cbd87c71106c31747aac24f8d36c4_1440w.jpg)
- web2.0 集群应用
  ![](https://pic3.zhimg.com/80/v2-86b2ea00699d9733cbb5bcff7488658a_1440w.jpg)
- web3.0 分布式应用
  ![](https://pic3.zhimg.com/80/v2-a99ddce97f9066f7d3ebb62ec397cf8a_1440w.jpg)

- Q1: 多个相互调用的多个服务会带来什么问题？
- A1: 服务间调用需要在代码里写死调用方 IP 和端口信息，靠人工手写配置，日常变更配置非常麻烦。

### 如何管理多个服务

- 思路：引入一个中间代理，解决新增服务动态获取 IP 问题

- 服务注册|发现
  <div style="display:flex;">
  <img src="https://pic2.zhimg.com/80/v2-e0d57ebf56f9743a03f35e297934d9dd_1440w.jpg" alt="服务注册" width="45%" height="50%" align="bottom" style="margin-right:5%;"/>
  <img src="https://pic4.zhimg.com/80/v2-7e32eeb70fad22a91e7f8d97088d606b_1440w.jpg" alt="服务注册" width="45%" height="50%" align="bottom" />
  <div>

```js
//服务发现，获取 User 服务的列表
list = NameServer->getAllServer('User');
//list 的内容
[
    {
        "ip":"192.178.1.1",
        "port":3445
    },
    {
        "ip":"192.178.1.2",
        "port":3445
    },
    {
        "ip":"192.178.1.6",
        "port":3445
    }
]
```

### 服务注册和发现- etcd

- etcd 定义: Etcd is a distributed, consistent `key-value store` for `shared configuration` and `service discovery` 是一个分布式的，一致的 key-value 存储，主要用途是共享配置和服务发现

* Q1: 服务注册和服务发现除了可以管理 IP 地址，还有其他功能吗？
  A1: 健康检查-服务器的存活状态、数据一致性管理等。

* Q2: 服务注册和服务发现有什么难点?
  A2: 集群、数据同步、强一致性、高并发高可用、选举机制、分布式

* Q3: 业界的解决方案有哪些?
  A3: zookeeper 和 consul 以及 etcd

* Q4: 我们 FE 部门内部是怎么解决的?
  Q4: etcd

###  微服务架构

![](./images/gateway.png)

## 问题一：如何本地调试多个相互调用的 GRPC 服务？

### A 服务调用 B 服务 Demo

```js
import { getClients } from 'xxx-protobuf';
const clients = await getClients();
//这里的key是公共仓库里面的配置的key。
const res = await clients[packageName](functionName, actionParam, metadata);
```

- 先回顾 grpc 通信的全流程
  ![](./images/etcdService.png)

- 调试的关键点是，中间人 etcd 是否有正确的登记 AB 服务的信息(IP|PORT)。

在本地调试时候，可以把服务注册到本地 etcd `"ETCD_HOSTS": "127.0.0.1:2379"`，你也可以使用线上的 etcd，原则是保证你要调用的 AB 服务有被正确注册到同样的 etcd。

![](./images/etcd.png)
关于如何使用本地 etcd，推荐查看[文档](./etcd.md)

### 本地调试前后端全流程

| 协议 |        工具        |      涉及服务      |
| ---- | :----------------: | :----------------: |
| http | whistle、zan-proxy | gateway+rpcService |
| rpc  |      BloomRpc      |     rpcService     |

### 更多调试方案

- [SSH 隧道（tunnel)端口转发](./ssh.md)

## 问题二： grpc-js 在不同版本定义不兼容，会对开发造成什么影响？

### 问题介绍

- case1: 访问页面，请求返回如下错误`Channel credentials must be a ChannelCredentials object`
- case2: 本地启动服务失败，提示`ChannelCredentials type err`

反复重启重安装 node-modules，重启大法失效了。

### 基础库介绍

- grpclb = @grpc/grpc-js + etcd3
  封装了 node-grpc 的基础库，提供客户端和服务端的创建，ectd 的服务注册方法。

```js
import { instantiateClient } from '@xxx/grpclb';
import { Server, ServerCredentials } from '@grpc/grpc-js';
import { register } from '@xxx/grpclb';
```

- xxx-protobuffer = grpclb + proto
  将业务的 pb 文件定义做了统一汇聚

- gateway = xxx-protobuffer + fastify
  fe node 的网关，将 HTTP 协议转化成 GRPC 调用

### 基础库调用关系图

![](./images/grpclb.png)

### 解决思路

- 找到报错的代码

![](./images/initClient.png)

- 检查是否使用了多个不同版本的 @grpc/grpc-js

![](./images/grpc-js-xx.png)

- due to @grpc/grpc-js version incompatibility, we recommended to keep the version unified

| grpclb version | @grpc/grpc-js version |
| -------------- | --------------------- |
| 3.0.0-3        | 1.1.5                 |
| 3.0.0-4        | 1.3.0                 |

- 在 package.json 里面锁定版本(小圆老师锁版本)

```js
  "resolutions": {
    "@grpc/grpc-js": "1.1.5",
    "@xxx/grpclb/@grpc/grpc-js": "1.1.5",
    "@xxx/grpclb/etcd3/@grpc/grpc-js": "1.1.5",
    "xxx-protobuf/@grpc/grpc-js": "1.1.5"
  }
```

## 问题三： 服务部署失败，怎么办？

### 前置知识：kibana 和 k8s pod 看到的日志有什么区别

- kibana: 宿主机的全量日志
- k8s pod: 当前 Pod 内日志

### 场景 A: jenkins 部署是成功的，但代码未生效，可以通过 Pod 日志定位。

1. action-review k8s: https://dashboard.i.test.xxxx.in.th/#/overview?namespace=xxx-frontend-service-vn-staging

2. fe gateway: https://dashboard.i.test.xxx.in.th/#/pod?namespace=xxx-frontend-gateway-vn-staging

3. 进入对应的 pod shell，执行 `ls -tlr`确认 jenkins 发布生效否

4. 进入日志目录`/data/logs/*/*/flog.log /data/logs/*/*/rlog.log`(宿主机采集目录， mitra 的规范一般是/_/_/般是 namesxxxe/deployment，flog.log 可以理解为接口日志，上游服务调用我们的服务时，会把传入的参数打印到这个文件中；rlog 代码中输出的日志了)

5. 查看日志的方式 A. `tail -f xxx.log | grep 关键字例如POST` B. less 等。

6. 如果 review 服务出了 bug，首先打开 gateway 和 review 服务本身的 log，分别查看他们的 flog.log 观察日志的滚动。gateway 如果可以正常打印出日志，没有 err，则大多是 review 的问题。

### 场景 B: jenkins 部署失败，Pod 一直重启，但是启动不成功，查看宿主机日志。

- 现象: Jenkins 日志反馈 kubectl 部署失败

```
+ kubectl -n xxx-frontend-service-vn-staging rollout status deploy/rn-hotpatch-admin-service --kubeconfig=/home/jenkins/.kube/staging-vn-kubeconfig
error: deployment "rn-hotpatch-admin-service" exceeded its progress deadline
+ '[' 1 == 0 ']'
+ '[' 1 == 0 ']'
+ echo 'Deploy had some problems, please check the k8s dashboard for details.'
Deploy had some problems, please check the k8s dashboard for details.
```

- 处理方案
  如果 Pod 启动失败，去 kibana 查看宿主机错误日志 https://kibana.i.test.xxx.in.th/app/kibana#/discover?_g=h@44136fa&_a=h@84b9311

- 如何搜索有用的日志记录

  1. PodName：从 k8s 拷贝出当前服务的 PodName 按照条件检索。
     ![](./images/kibana-log.png)

  2. 也可以通过环境等其他参数搜索。
     ![](./images/k8s-log.png)

### 处理思路

1. 如果是 Pod 启动秒退的话，可能是初始化的问题了，可能是健康检查未通过，端口不对等。
   ![](./images/ng-log.png)
   如图显示 POD 已经成功启动，健康检查也通过了，但是程序中间退出了

2. 如果是 POD 成功启动，中间退出，基本是代码的问题，（比如 CRON 或其他功能）出问题了，导致服务挂了，POD 退出又开始重启。

3. 如果依然搞不定话，可以请求 SRE 帮忙查看更多日志信息。

4. 万一没有日志的话，本地启动 docker image，模拟同样的部署环境，要注意 配置/版本 要保持一致。
