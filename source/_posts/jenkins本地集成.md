---
title: jenkins本地集成
date: 2018-10-13 11:14:54
categories: 
- tools
tags:
- macos
- jenkins
---
工作中，总能遇到一些奇葩的事情，例如jenkins在build的时候有时候会失败，qa在群里@开发，开发build下，又成功了，也是一脸蒙蔽。
### 一、本地安装jenkins
1. java安装，没有安装先去安装[java环境](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，安装的跳过。
```
java -version
```
2.  brew安装，没安装的执行命令，安装的跳过。
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
3. 安装jenkins
```sh
brew install jenkins     
```
4. 启动jenkins
```
jenkins     
```
5. 访问jenkins
```sh
http://localhost:8080/     
```
6. 账号设置、安装推荐插件、创建管理员用户，总之跟着提示做就好了
```
cat /Users/weiqian/.jenkins/secrets/initialAdminPassword    
```
### 二、实战操作
1. 安装插件
```
系统管理 - > 插件管理 - >available - > 过滤 - >选择插件 -> 直接安装
Xcode integration （必须）
Keychains and Provisioning Profiles Management （必须）
Email Extension Template
CocoaPods Jenkins Integration
GitLab
GitLab Hook
GitLab Authentication
```
2. 配置Keychains and Provisioning Profiles Management 
```
// step1. 系统管理 -> Keychains and Provisioning Profiles Management 
cp /Users/weiqian/Library/Keychains/login.keychain-db /Users/weiqian/Desktop/login.keychain
// step2. upload desktop login.keychain
// step3. Provisioning Profiles Directory Path fill path
/Users/weiqian/Library/MobileDevice/Provisioning Profiles
// step4. save
```
3. 新建任务
```
new Task -> 自由风格的软件项目 -> General -> 源码管理 -> 构建触发器
```
表面上看起来是成功了。
路漫漫，未完待续····


