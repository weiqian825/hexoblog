---
title: electron做go爬虫的gui
date: 2018-08-30 15:20:45
categories: 
- web
tags:
- electron
- go
- gui
---
一期要做的事情很简单，用electron给一个GO爬虫脚本做一个前端的GUI，顺便学习下[electron](https://electronjs.org/)
### 一、运行go爬虫
[go爬虫](https://github.com/weiqian825/jira-auto/tree/master)先要简单配置下[go](https://golang.org/)的环境
```js
// 1. install go
brew install go -g
// 2. config gopath
vim ~/.bash_profile
// my simple config
export PATH=$PATH:/Users/weiqian/go/bin
export GOPATH=/Users/weiqian/go
// 3. go env 
source ~/.bash_profile
// 4. clone code in go/src 
// method 1 use go get clone the code 
go get github.com/weiqian825/jira-auto
//method 2  
cd  /Users/weiqian/go/src/jira-auto
git clone  https://github.com/weiqian825/jira-auto.git
// 5. run program
go run main.go
// 如果程序报错缺少库 go get 去安装就好了

```

### 二、eletron 部分

1. electron由一个主进程和若干个渲染进程组成，主进程和渲染进程通信用ipcMain，在调试或者在功能开发的时候这个太需要了

```sh
// 主进程的调试信息
//开发时候打印的日志可以在bash里面看到。
//打包成可执行程序之后，可以打开jira-auto.app/Contents/MacOS/jira-auto，同样可以看到
//主进程接收由渲染进程发来的消息
ipcMain.on('exec:querybtn', function (e, command) {
  // 主进程向渲染进程发消息，可以在界面console看到
  mainWindow.webContents.send('exec:querybtn', '进入函数：exec')
  exec(path.resolve( command, (error, stdout, stderr) => {
    mainWindow.webContents.send('exec:querybtn', '结束执行', error, stdout, stderr)
    error && console.log(error)
    stdout && console.log(stdout)
    stderr && console.log(stderr)
  })
})

//渲染进程，发送用户的动作到主进程
ipcRenderer.send('exec:querybtn',run_exec)

// 渲染进程接受消息，可以在界面console看到
ipcRenderer.on('exec:querybtn', function (e, tips, error, stdout, stderr) {
    console.log(tips);
    error && console.log("error_____",error);
    stdout && console.log("stdout_____",stdout);
    stderr && console.log("stderr_____",stderr);
});

```
2. electron里面如何调用go脚本？ go程序提供的是cli接口，不是http接口。查询相关资料后，分别尝试了shelljs和node.js里面child_process模块的spawn方法。

```sh

// method 1: try use shelljs ---error
ipcMain.on('shelljs:querybtn', function (e, command) {
  mainWindow.webContents.send('shelljs:querybtn', command)
  // need execPath /usr/local/bin/node ?
  // // https://github.com/shelljs/shelljs/issues/704
  // shell.config.execPath = shell.which('node').stdout
  // shell.config.execPath = process.env.PATH
  console.log('shelljs command---> ', command)
  shell.exec(command)
})

// method 2: try use child_process  exec
ipcMain.on('exec:querybtn', function (e, command) {
  console.log('exec command---> ', command)
  mainWindow.webContents.send('exec:querybtn', '进入函数：exec')

  exec(path.resolve( command, (error, stdout, stderr) => {
    console.log('执行结束：exec')
    mainWindow.webContents.send('exec:querybtn', '结束执行', error, stdout, stderr)
    error && console.log(error)
    stdout && console.log(stdout)
    stderr && console.log(stderr)
  })
})

// method 3: try use child_process  spawn
ipcMain.on('spawn:querybtn', function (e, command) {
  console.log('spawn command---> ', command)
  const _process = spawn( command[0], command[1])
  mainWindow.webContents.send('spawn:querybtn', '进入函数：spawn')

  _process.stdout.on('data', function (data) {
    console.log('stderr执行结束：spawn')
    mainWindow.webContents.send('spawn:querybtn', 'stderr结束执行', data)
    console.log(data.toString())
  })

  _process.stderr.on('data', function (data) {
    console.log('stderr执行结束：spawn')
    mainWindow.webContents.send('spawn:querybtn', 'stderr结束执行', data)
    console.log(data.toString())
  })
})

```
3. 经过1，2小问题的解决后，开心的准备打包  electron-packager
```  json
  "scripts": {
       "start": "electron main.js", // 不要用node main.js electron里面的app等会找不到
       "test": "echo \"Error: no test specified\" && exit 1",
       "pack": "electron-packager --out ../outApp  --overwrite  .",
       "standard": "standard"
  }
```
npm run pack 打包成功了，个性化配置有很多参数可以查询，运行程序发现一直报错并不能运行。
4. 查看文件有没有被打包进去，右击exe=>Show Package Content=>Contents=>Resources=>app=>。。。。可以看见整个工程目录被打包进去了
5. 打印日志，发现报错是一直找不到go文件，也就是electron的部分执行了，go的部分不能正常运行
``` go
_process = spawn('go',['run','main.go','--cookie="xxxx"'])
```
6. 目前的go是源码呈现的，并没有被编译成可执行文件，这样就会依赖gopath的环境，让go脱离gopath环境的以来，需要把go编译成可以执行文件，go build 之后command部分变成 ./xxx 
```  go
_process = spawn('./main',['--cookie="xxxx"'])
_process.stdout.on('data',function(data){
    console.log(data.toString())
})
```
7. Show Package Content显示go可执行文件已经被打包进去了，然而报错路径找不到，猜想是相对路径的问题，添加__dirname
 ```  go
_process = spawn(path.resolve(__dirname)+'/'+'./main',['--cookie="xxxx"'])
_process.stdout.on('data',function(data){
    console.log(data.toString())
})
 ```
8. 函数是可以正常执行了，又报错open data.csv permission denied，先试试选择没有权限要求的文件去写文件，例如desktop
```go
    import (
        "os/user"
    )
    myself, error := user.Current()
    if error != nil {
        panic(error)
    }
    homedir := myself.HomeDir
    desktop := homedir + "/Desktop/data.csv"
```
9. 在体验输入账户时候，发现macos的input部分不能粘贴等操作，解决如下
```js
if (process.platform === 'darwin') {
  mainMenuTemplate.unshift({
    label: 'Edit',
    submenu: [{
      role: 'undo'
    }, {
      role: 'redo'
    }, {
      type: 'separator'
    }, {
      role: 'cut'
    }, {
      role: 'copy'
    }, {
      role: 'paste'
    }, {
      role: 'pasteandmatchstyle'
    }, {
      role: 'delete'
    }, {
      role: 'selectall'
    }]
  })
}

```

### 三、目录结构调整

1. spider放一起
2. client放一起
3. build写好脚本后，chmod 755 xxx.sh
```js
   // build_spider.sh
   cd ./client && npm install && npm run pack 
   // build_client.sh
   go build -o ./client/jira_spider ./spider &&  cd ./client && npm install && npm run pack
   // build.sh 
   ./build_spider.sh && ./build_client.sh
```

  



