---
title: 部署-node+pm2部署流程
date: 2020-03-31 08:53:11
tags:
---


## 01​确认本机调试OK

```
 "scripts": {
    "prebuild": "echo I run before the build script",
    "build": "nest build",
    "postbuild": "echo I run after the build script"
    "start": "nest start",
    "debug": "nest start --debug --watch",
    "prod:node": "node dist/src/main.js",
    "prod:pm2": " sh scripts/start.sh live",
    "test:watch": "NODE_ENV=test jest --config ./test/jest-e2e.json --watch",
    "test:coverage": "NODE_ENV=test jest --config ./test/jest-e2e.json --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand"
  }
 
#确保本地测试用例ok
npm run test 
#确保本地编译OK
npm run build 
#确保本地node起程序OK
npm run prod:node  
#确保程序已经启动
curl -X GET http://localhost:3000/api/v1 
#确保本地PM2启动程序ok
npm run prod:pm2  
#确保程序已经启动
curl -X GET http://localhost:3000/api/v1
```
## 02追踪Jenkins执行
```
1.仔细看jenkins里面配置的脚本，不确定的话，多打印日志
2.查看jenkins log的每个步骤，分析ansible或者其他中控执行细节
3.直到jenkins部署成功(git 拉取、编译环境、拉取包是否OK、推送)
```
## 03排查部署机器的问题
```
#切换到正确用户
su kredit
#查看进程是否启动
pm2 list 
#找出运行在指定端口的进程
netstat -anop | grep ':20002'
#linux系统查看node进程
ps -ef|grep node
#关闭node程序
kill -9 pid
#验证程序是否启动
curl -X GET http://localhost:3000/api/v1
#查看pm2日志
pm2 logs id
#启动node服务
node dist/src/main.js
#查看node日志 
tail -200 xxx.log
#检查数据库是否可以连通
mysql -uroot -h localhost -P3306 -p
#检查各种环境变量是否传入正确
node_modules/@nestjs/cli/bin/nest.js start --debug
```
测试node脚本
```

// 导入http模块:
var http = require('http');
// 创建http server，并传入回调函数:
var server = http.createServer(function (request, response) {
    // 回调函数接收request和response对象,
    // 获得HTTP请求的method和url:
    console.log(request.method + ': ' + request.url);
    // 将HTTP响应200写入response, 同时设置Content-Type: text/html:
    response.writeHead(200, {'Content-Type': 'text/html'});
    // 将HTTP响应的HTML内容写入response:
    response.end('Hello world!');
});
// 让服务器监听8080端口:
server.listen(8080);
console.log('Server is running at http://127.0.0.1:8080/')
```
Node测试mysql是否连通的脚本
```
let mysql = require("mysql");
const db_config={
    host:"localhost",
    user:"root",
    password:"root",
    port:"3306",
    database:"testdb"
}
let connect=mysql.createConnection(db_config);
//开始链接数据库
connect.connect(function(err){
    if(err){
        console.log(`mysql连接失败: ${err}!`);
    }else{
        console.log("mysql连接成功!");
    }
});
//基本的查询语句
let sqlQuery="select * from tb_user";
connect.query(sqlQuery,function(err,result){
    if(err){
        console.log(`SQL error: ${err}!`);
    }else{
        console.log(result);
        closeMysql(connect);
    }
});
//查询成功后关闭mysql
function closeMysql(connect){
    connect.end((err)=>{
        if(err){
            console.log(`mysql关闭失败:${err}!`);
        }else{
            console.log('mysql关闭成功!');
        }
});
}
```
查看服务进程数要启动几个进程，可以通过服务器内核确定
```
#查看物理CPU个数
cat /proc/cpuinfo| grep "physical id" | sort| uniq | wc -l
#查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq
#查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```
进程CPU高占比查询
```
#通过top服务器，定位到PID为xx的node进程占用CPU到达100%
top
#显示线程列表，按照CPU占用高的线程排序
top -Hp PIDxxx
#找到最耗时线程的PID，然后将它转成16进制的格式
printf "%x\n"  PIDxxx
#通过jstack命令查看jvm，grep耗时线程的详细堆栈信息
jstack PIDxxx |  grep 77e0 -A  60
```
## 04其他笔记
```
fork与cluster启动模式
 
开发环境中多以fork启动，生产环境中多用cluster方式启动。
 
fork不支持socket端口复用，cluster支持地址端口复用，因为只有node的cluster模块支持socket选项SO_REUSEADDR
 
cluster是fork的派生，cluster支持所有cluster拥有的特性。
 
fork模式可以应用于其他语言，如php，python，perl，ruby，bash，coffee， 而cluster只能应用于node
 
fork不支持定时重启，cluster支持定时重启。定时重启也就是配置中的cron_restart配置项。
 
日志问题
 
pm2的相关文件默认存放于$HOME/.pm2/目录下，其日志主要有两类：
 
pm2自身的日志，存放于$HOME/.pm2/pm2.log；
 
pm2所管理的应用的日志，存放于$HOME/.pm2/logs/目录下，标准输出日志存放于${APP_NAME}_out.log，标准错误日志存放于${APP_NAME}_error.log；
```
```
npm install pm2 -g
pm2 list
pm2 start app_name(id|all)
pm2 restart app_name(id|all)
pm2 stop app_name(id|all)
pm2 delete app_name(id|all)
pm2 reload app_name(id|all)
pm2 startOrReload app_name(id|all)
pm2 logs app_name(id|all)
pm2 show app_name(id|all)
 
 
# Fork mode
pm2 start app.js --name my-api # Name process
# Cluster mode
pm2 start app.js -i 0        # Will start maximum processes with LB depending on available CPUs
pm2 start app.js -i max      # Same as above, but deprecated.
pm2 scale app +3             # Scales `app` up by 3 workers
pm2 scale app 2              # Scales `app` up or down to 2 workers total
# Listing
pm2 list               # Display all processes status
pm2 jlist              # Print process list in raw JSON
pm2 prettylist         # Print process list in beautified JSON
pm2 describe 0         # Display all informations about a specific process
pm2 monit              # Monitor all processes
# Logs
pm2 logs [--raw]       # Display all processes logs in streaming
pm2 flush              # Empty all log files
pm2 reloadLogs         # Reload all logs
# Actions
pm2 stop all           # Stop all processes
pm2 restart all        # Restart all processes
pm2 reload all         # Will 0s downtime reload (for NETWORKED apps)
pm2 stop 0             # Stop specific process id
pm2 restart 0          # Restart specific process id
pm2 delete 0           # Will remove process from pm2 list
pm2 delete all         # Will remove all processes from pm2 list
# Misc
pm2 reset <process>    # Reset meta data (restarted time...)
pm2 updatePM2          # Update in memory pm2
pm2 ping               # Ensure pm2 daemon has been launched
pm2 sendSignal SIGUSR2 my-app # Send system signal to script
pm2 start app.js --no-daemon
pm2 start app.js --no-vizion
pm2 start app.js --no-autorestart
```