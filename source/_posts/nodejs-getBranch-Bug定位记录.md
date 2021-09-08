---
title: nodejs-getBranch Bug定位记录
date: 2021-09-07 23:11:19
tags:
---

## 问题

请求接口，返回异常

```
➜  ~ curl 'https://fe.xxx.vn/api/admin/v2/fe.xxxGitApiService/GetBranch' \
  -H 'authority: fe.xxx.vn' \
  -H 'pragma: no-cache' \
  -H 'cache-control: no-cache' \
  -H 'sec-ch-ua: "Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"' \
  -H 'accept: application/json, text/plain, */*' \
  -H 'a-path: .' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36' \
  -H 'content-type: application/json' \
  -H 'origin: https://admin.xxx.io' \
  -H 'sec-fetch-site: cross-site' \
  -H 'sec-fetch-mode: cors' \
  -H 'sec-fetch-dest: empty' \
  -H 'referer: https://admin.xxx.io/' \
  -H 'accept-language: zh-CN,zh;q=0.9,en;q=0.8' \
  --data-raw '{"id":27296,"version":1}' \
  --compressed
{"msg":"No connection established","ret":14,"timestamp":1631027817041,"data":{}}
```

## 查看业务部署和 POD 状况

1. 确认服务的调用链路： 网关服务 fe-gateway 通过域名 fe.xxx.com 调用了 git-api-service 服务
2. fe-gateway 和 git-api-service 没有发布记录
3. Pod 运行正常

## 顺着调用链路查看日志

1. 先查看网关日志

```


2021-09-07 22:26:51.363000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|{"level":30,"time":1631028411361,"pid":25,"hostname":"fe-admin-gateway-8489bc4645-2xt4f","reqId":1611,"req":{"method":"OPTIONS","url":"/api/admin/v2/fe.xxx.GitApiService/GetBranch","hostname":"fe.test.xxx.vn","remoteAddress":"100.98.44.0","remotePort":58099},"msg":"incoming request","v":1}\n

2021-09-07 22:26:51.411000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|{"level":30,"time":1631028411410,"pid":25,"hostname":"fe-admin-gateway-8489bc4645-2xt4f","reqId":1611,"res":{"statusCode":204},"responseTime":49.50215768814087,"msg":"request completed","v":1}\n

2021-09-07 22:26:51.935000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|{"level":30,"time":1631028411934,"pid":25,"hostname":"fe-admin-gateway-8489bc4645-2xt4f","reqId":1612,"req":{"method":"POST","url":"/api/admin/v2/fe.xxx.GitApiService/GetBranch","hostname":"fe.test.xxx.vn","remoteAddress":"100.98.44.0","remotePort":58114},"msg":"incoming request","v":1}\n
2021-09-07 22:26:51.935000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|---request headers {\n  host: 'fe.test.xxx.vn',\n  'x-request-id': '6dbb93ff9b3d4d2351f49e2ef8bad2c7',\n  'x-real-ip': '103.201.26.18',\n  'x-forwarded-for': '103.201.26.18, 10.65.8.130',\n  'x-forwarded-host': 'fe.test.xxx.vn',\n  'x-forwarded-port': '80',\n  'x-forwarded-proto': 'http',\n  'x-original-uri': '/api/admin/v2/fe.xxx.GitApiService/GetBranch',\n  'x-scheme': 'http',\n  'x-original-forwarded-for': '103.201.26.18',\n  'content-length': '24',\n  pragma: 'no-cache',\n  'cache-control': 'no-cache',\n  'sec-ch-ua': '"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"',\n  accept: 'application/json, text/plain, */*',\n  'a-path': '.',\n  'sec-ch-ua-mobile': '?0',\n  'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36',\n  'content-type': 'application/json',\n  origin: 'https://admin.codepush.i.test.xxx.io',\n  'sec-fetch-site': 'cross-site',\n  'sec-fetch-mode': 'cors',\n  'sec-fetch-dest': 'empty',\n  referer: 'https://admin.codepush.i.test.xxx.io/',\n  'accept-encoding': 'gzip, deflate, br',\n  'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8'\n}\n--request.params['*'] undefined\nrequest.params['*']==='upload' false\nmeta Metadata {\n  internalRepr: Map {\n    'operator' => [ 'jenkins' ],\n    'email' => [ 'jenkins@xxx.com' ]\n  },\n  options: {}\n}\n

```

2. 在查看 git-api-service 服务日志，没有日志记录

由 1,2 可分析网关请求没有发送到服务，有可能由很多原因，大概率是运维问题。

3. 从网关的 pod ping 服务的 pod，不通

```
bash-4.4# ping 100.98.164.174
PING 100.98.164.174 (100.98.164.174): 56 data bytes
```

如何获取一个 pod 的 ip 呢？ 在 k8s 的服务 pod 面板上，会有一个服务 pod 的 ip 信息

4. 当网关 pod ping 通了服务 pod 之后，网关的日志正常了

```
2021-09-08 08:35:37.302000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|{"level":30,"time":1631064937302,"pid":23,"hostname":"fe-admin-gateway-8489bc4645-2xt4f","reqId":1723,"req":{"method":"OPTIONS","url":"/api/admin/v2/fe.apc.gitapi.GitApiService/GetBranch","hostname":"fe.test.apc.daily.airpay.vn","remoteAddress":"100.98.44.0","remotePort":25201},"msg":"incoming request","v":1}\n
2021-09-08 08:35:37.303000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|{"level":30,"time":1631064937303,"pid":23,"hostname":"fe-admin-gateway-8489bc4645-2xt4f","reqId":1723,"res":{"statusCode":204},"responseTime":0.740415096282959,"msg":"request completed","v":1}\n
2021-09-08 08:35:37.438000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|{"level":30,"time":1631064937438,"pid":25,"hostname":"fe-admin-gateway-8489bc4645-2xt4f","reqId":1615,"req":{"method":"POST","url":"/api/admin/v2/fe.apc.gitapi.GitApiService/GetBranch","hostname":"fe.test.apc.daily.airpay.vn","remoteAddress":"100.98.44.0","remotePort":25203},"msg":"incoming request","v":1}\n
2021-09-08 08:35:37.439000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|---request headers {\n  host: 'fe.test.apc.daily.airpay.vn',\n  'x-request-id': '6b389cd317850eeb9446c35d9752269a',\n  'x-real-ip': '103.201.26.13',\n  'x-forwarded-for': '103.201.26.13, 10.65.8.130',\n  'x-forwarded-host': 'fe.test.apc.daily.airpay.vn',\n  'x-forwarded-port': '80',\n  'x-forwarded-proto': 'http',\n  'x-original-uri': '/api/admin/v2/fe.apc.gitapi.GitApiService/GetBranch',\n  'x-scheme': 'http',\n  'x-original-forwarded-for': '103.201.26.13',\n  'content-length': '24',\n  pragma: 'no-cache',\n  'cache-control': 'no-cache',\n  'sec-ch-ua': '"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"',\n  accept: 'application/json, text/plain, */*',\n  'a-path': '.',\n  'sec-ch-ua-mobile': '?0',\n  'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36',\n  'content-type': 'application/json',\n  origin: 'https://admin.codepush.i.test.xxx.io',\n  'sec-fetch-site': 'cross-site',\n  'sec-fetch-mode': 'cors',\n  'sec-fetch-dest': 'empty',\n  referer: 'https://admin.codepush.i.test.xxx.io/',\n  'accept-encoding': 'gzip, deflate, br',\n  'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8'\n}\n
2021-09-08 08:35:37.439000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|--request.params['*'] undefined\nrequest.params['*']==='upload' false\nmeta Metadata {\n  internalRepr: Map {\n    'operator' => [ 'jenkins' ],\n    'email' => [ 'jenkins@xxx.com' ]\n  },\n  options: {}\n}\n
2021-09-08 08:35:37.461000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|{"level":30,"time":1631064937461,"pid":25,"hostname":"fe-admin-gateway-8489bc4645-2xt4f","reqId":1615,"res":{"statusCode":200},"responseTime":22.45934772491455,"msg":"request completed","v":1}\n
```

5. git-api 服务开始有日志了，但是日志里面请求不对

```
2021-09-08 08:37:46.546000+07:00 info|grpcMethod:node-agent.worker|traceId:|uid:|file:|func:|Error: Client network socket disconnected before secure TLS connection was established\n    at connResetException (internal/errors.js:570:14)\n    at TLSSocket.onConnectEnd (_tls_wrap.js:1361:19)\n    at Object.onceWrapper (events.js:299:28)\n    at TLSSocket.emit (events.js:215:7)\n    at endReadableNT (_stream_readable.js:1184:12)\n    at processTicksAndRejections (internal/process/task_queues.js:80:21) {\n  code: 'ECONNRESET',\n  path: null,\n  host: 'git.xxx.com',\n  port: 443,\n  localAddress: undefined,\n  config: {\n    url: 'https://git.xxx.com/api/v4/projects/xxx/repository/branches?search=',\n    method: 'get',\n    headers: {\n      Accept: 'application/json, text/plain, */*',\n      'PRIVATE-TOKEN': 'xxx',\n      'User-Agent': 'ax

```

6. 开始在 git-api-service curl git 服务，发现时好时坏

```

bash-4.4# curl --header "PRIVATE-TOKEN: xxx" https://git.xxx.com/api/v4/projects/xxx/repository/branches?search=
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to git.xxx.com:443
bash-4.4#
bash-4.4# curl --header "PRIVATE-TOKEN: xxx" https://git.xxx.com/api/v4/projects/xxx/repository/branches?search=
[{"name":"feature-decentralized","commit":{"id":"4700b1aa85c7bfbe026ccbd7ea97df4478a2dbf2","short_id":"4700b1aa","created_at":"2021-09-01T10:59:25.000+08:00","parent_ids":null,"title":"feat: bump otp-ss to 1.0.1-slieder.4","message":"feat: bump otp-ss to 1.0.1-slieder.4","author_name":"xxx","author_email":"xxx.wilianto@xxx.com","authored_date":"2021-09-01T10:59:25.000+08:00","committer_name":"xxx","committer_email":"xxx.wilianto@xxx.com","committed_date":"2021-09-01T10:59:25.000+08:00","web_url":"https://git.xxx.com/xxx/app-fe/otp-ss-standalone/-/commit/4700b1aa85c7bfbe026ccbd7ea97df4478a2dbf2"},"merged":false,"protected":false,"developers_can_push":false,"developers_can_merge":false,"can_push":true,"default":false,"web_url":"https://git.xxx.com/xxx/app-fe/otp-ss-standalone/-/tree/feature-decentralized"}
```

7. 总结下一共发现了 2 个问题，需要反馈给运维

Q1：网关有时候 ping 不通服务 pod
Q2：git 服务 pod curl git 请求时好时坏

## SRE 解决问题

1. 将 2 个问题及 k8s 的服务信息反馈给 SRE
2. 先使用 dig gitlab.xxx.com，多运行几次， git-api-service 调用 gitlab 的域名会变
3. 进一步定位 coredns 被绑定了别的 ip
4. sre 修复配置
