---
title: linux常用基础操作
date: 2018-10-09 20:57:58
categories: 
- tools
tags:
- linux
---

liunx有很多优秀的地方，用了mac后确实再也不想用windows了，简单记录下工作日常需要了解的一些基础知识。

## 一、文件权限和目录

1. 多用户、多任务环境，文件可存取访问的身份owner|group|other，3种身份各有read4r|write2w|execute1x
```
➜  man ls  // 详细的命令
➜  info ls 
➜  ll // 文件权限|连接数|文件所属者|文件所属用户组|文件大小|文件最后修改时间|文件名
➜  whoami
➜  id -a weiqian
➜  groups
➜  id -a root
➜  sudo chown root:wheel demo
➜  sudo chomd 777 demo
```
2. linux目录配置标准FHS
```
/ 根目录，与开机系统有关
/usr（UNIX software resource） 与软件安装执行有关
/var (variable) 与系统运作过程有关
/bin 重要的执行文件，主要有cat|chomod|chown|date|mv|mkdir|cp|bash
/sbin 重要的系统执行文件
/etc 系统配置文件，例如人员的账号密码配置，各种文件的起始文件
/home 用户的住文件夹
/opt 第三方软件的放置目录（非原本的distribution）

```
3. 目录相关操作
```
.  此层目录
.. 上层目录
-  前一个工作目录
~  目前用户身份所在的主文件夹
pwd|mkdir|rmdir //处理目录
cp|rm|mv        //复制删除移动
cat|tac|nl|more|less|head|tail //查看文件
touch //修改文件时间或者创建新文件
```
## 二、硬件、内核与shell
1. 什么是shell
```
我们必须通过shell讲我们输入的命令与内核通信，好让内核可以控制硬件来正确无误的工作。操作系统其实是一组软件，由于这组软件在控制整个硬件与管理系统的活动检测，如果这组软件能被用户随意操作，若用户使用不当，将会使得系统崩溃。因为操作系统就是管理整个硬件功能，所以不能随便被一些没有管理能里的终端用户随意使用。
shell的功能只是提供用户操作系统的一个接口，因此这个shell需要调用其他软件才好，例如man|chmod|vi等等，这些命令都是独立的应用程序，但是我们可以通过shell(就是命令行模式)来操作这些应用程序，让这些应用程序调用内核来运行所需的工作。
```
2. 我们常用的shell
```
（Bourne Again Shell）简称bash，这个shell是Bourne Shell的增强版本
/bin/sh(已经被/bin/bash替代)
/bin/bash(linux的默认的)
/bin/ksh
/bin/tcsh
/bin/csh
/bin/zsh(基于ksh发展出来的，功能更强大的的shell)
```
3. bash shell的功能
```
命令记忆功能 history
命令与文件补全功能 tab
命令别名设置功能 alias
```
4. shell的变量功能
```
影响你能不能在任何目录下执行某个命令与PATH这个变量有很大关系，例如你在执行ls这个命令时候系统就是通过PATH这个变量里面的内容所记录的路径顺序来查找命令的，如果PATH这个变量里面路径还找不到ls这个命令时候，就会在屏幕上显示 “command not found”
在linux下所有的执行都需要一个执行码，你真正以shell来跟linux通信，是在正确登陆linux之后，这个时候你就有一个bash的可执行程序，也才可以真正经由bash和系统通信。而在进入shell之前，由于系统需要一些变量来提供它的数据访问，所以就有了一些环境变量需要读入系统。例如PATH|HOME|MAIL|SHELL，为了区分和自定义变量不通，环境变量通常用大些表示。
```
5. 变量的显示和设置

```
// 变量显示echo，只要在变量名称前面加上$
➜  / echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin
// 若为了该变量增加变量内容时候，可用‘$变量’累加内容，例如 PATH=“$PATH":/home/bin
```
## 三、bash的环境配置文件
1. 我们知道查看文件属性的命令ls的完整文件名为:/bin/ls，问题来了，为什么我可以在任何地方执行/bin/ls这个命令呢？
```
当我们在执行一个命令时候，系统会按照PATH的设置去每个PATH定义的目录下查询文件名为ls的可执行文件，如果在PATH定义的目录中含有多个文件名字为ls的可执行文件，那么先查询到的同名命令会被执行
```
2. 我们什么都没做，但是已进入bash就取得一堆有用的变量
```
这是因为操作系统有一些环境配置文件的存在，让bash在启动的时候就直接读取了这些配置文件，以规划好bash的操作环境，这些环境又分为系统文件和个人偏好配置文件。
```
3. ~/.bash_profile (login shell才会读取)
```
bash在读完了整体环境变量设置的/etc/profile并借此调用其他的配置文件之后，接下来会读取用户的个人配置文件，在login shell的配置文件种，所读取的个人偏好文件有3个依次是 ~/.bash_profile | ~/.bash_login | ~/.profile，按照顺序读取其中的一个
```
4. source 读取环境配置文件的命令
```
由于/etc/profile与~/.bash_profile都是在取得login shell的时候才会读取的配置文件，所以如果你将自己的偏好设置写入上述文件后，通常都是注销在登陆后设置才能生效，想要不注销直接读取文件，就要利用source这个文件。
```




