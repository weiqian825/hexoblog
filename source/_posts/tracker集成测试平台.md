---
title: tracker集成测试平台
date: 2020-06-11 11:50:27
tags:
---

### 一、权限申请

```js
https://xxxx/ttp/tree/master/api

http://xxxx/web/ttp/tools/dbsyn

https://xxxx/ttp_web

for i in {1..10}; do curl http://checkip.amazonaws.com/; done

https://aws.amazon.com/cn/premiumsupport/knowledge-center/anonymous-not-authorized-elasticsearch/

rm test.txt; for i in {1..100}; do curl http://checkip.amazonaws.com/; done >> test.txt; cat test.txt | sort| uniq

```

### 二、python

```js
workon
mkvirtualenv ttp
workon ttp
pip install -r requirements.txt
```

### 三、测试平台部署

web 项目操作流程：

```
 1.必须要先申请物理机访问权限https://docs.google.com/document/d/1TZ7KyBPyBMRB_bbnsF1GWh9jIc0depIjWtGGc5irLuQ/edit
 2.在本地拉取TCP项目的masterhttps://xxxx/tcp
 3.本地build项目npm run build 或者 npm run build:prod
4.会生成 dist 文件夹，打包 cd disttar -zcvf testcase.tar ./\*
5.上传到远程物理机 scp testcase.tar xxx
6.在远程物理机上进入项目文件夹，并删除文件夹内所有文件 testcase 项目：cd /opt/testcase_fettp 项目：cd /opt/ttp_web
sudo rm -r static 等等
7.移动本地上传的文件到项目 testcase 项目：sudo mv ~/testcase.tar /opt/testcase_fettp 项目：sudo mv ~/testcase.tar /opt/ttp_web8.解压 sudo tar -zxvf testcase.tar
以上即可完成部署，可以不用重启 Nginx
```

BE 项目操作流程：

```
1.本地打包 tar -zcvf ttp.tar ./\*
2.上传到远程物理机上 scp ttp.tar xxx
3.将文件解压到/opt/ttp/文件夹下 sudo mv ~/ttp.tar /opt/ttpcd /opt/ttpsudo tar -zxvf ttp.tar
4.因为 ttp 项目是在xxx 的虚拟 python 环境中运行的，需要首次打包后端代码，需要修改自己的配置文件 vim ~/.bashrc 然后加上 export WORKON_HOME=/home/ld-sgdev/xxx/.virtualenvsexport VIRTUALENVWRAPPER_SCRIPT=/usr/local/bin/virtualenvwrapper.shexport VIRTUALENVWRAPPER_PYTHON=/usr/bin/pythonexport VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenvexport VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages'source /usr/local/bin/virtualenvwrapper.sh

然后source 一下配置文件 source ~/.bashrc

启动项目虚拟环境 workon ttp

5.查看当前是否有正在运行的进程 ps -ef | grep run.py
6.杀死当前正在运行的进程 kill -9 `lsof -i:8002 | head -2 | tail -1 | awk '{print $2}'` 或者 kill -9 [线程号]
7.后台运行项目 nohup python run.py&
8.再次查看是否拉起进程 ps -ef | grep run.py

如果没有拉起的话，可以直接运行 python run.py，查看出错地方 note：要把 conf/config.py 中的 DEBUG 改成 False，否则无法连接物理机数据库
```
