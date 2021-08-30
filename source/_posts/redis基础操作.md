---
title: redis基础操作
date: 2021-08-30 22:43:32
tags:
---


## Redis 基本操作

### 方案一： 登陆 test 机器，使用 redis-cli 操作 redis

1.登陆跳板机

```
ssh 10.65.x.x
```

2.命令行连接 redis redis-cli -h host -p port

```
ld-qian_wei@node1:~$ redis-cli -h 10.65.x.x -p 30002 -a password
```

3.redis-cli 命令操作 redis

- keys key

```
10.65.x.x:30002> keys codepush*
 1) "codepush_test_sg_ios{conf}"
```

- hgetall key

```
10.65.x.x:30002> HGETALL codepush_test_sg_ios{conf}
  1) "60_24.0.0"
  2) "{\"plugin merchant-food\":{\"device_id\":null,\"whitelist\":null}}"
  3) "59_21.0.0"
  4) "{\"plugin partner\":{\"device_id\":null,\"whitelist\":null}}"
```

- hmget key field

```
10.65.x.x:30002> hmget codepush_test_sg_ios{conf} 60_24.0.0
1) "{\"plugin merchant-food\":{\"device_id\":null,\"whitelist\":null}}"
```

- hkeys key

```
10.65.x.x:30002> hkeys codepush_staging_vn_ios{conf}
1) "40_3.0.0"
2) "41_3.0.0"
```

- 其他指令

```
//redis返回哈希表key的所有的value值
hvals key
//redis删除哈希表key的某一个field值和对应的value值
hdel key
//redis设置key的过期时间
expire key time | expireat key time
//redis查看key的到期时间或剩余的剩余的生存时间
ttl key
```

- 查看 proj redis 数据

```
ld-qian_wei@node1:~$ redis-cli -h 10.65.8.174 -p 30001 -a pass
10.65.8.174:30001> keys codepush*
 1) "codepush_test_android_test12333{proj}"
```

### 方案二（推荐）：使用 redis 客户端，建隧道连接 test 环境

1.安装 redis 客户端

```
brew install --cask another-redis-desktop-manager
```

2.申请机器 10.65.x.x 等权限

3.隧道连接 test redis

```
./ssht 10.65.x.x:30002
```

4. 打开 another-redis-desktop-manager 客户端，配置 redis 连接
