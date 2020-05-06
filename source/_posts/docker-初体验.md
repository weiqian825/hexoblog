---
title: docker 初体验
date: 2020-04-27 00:20:20
tags:
---

### 一、相关文档

https://docs.docker.com/docker-for-mac/

https://yeasy.gitbooks.io/docker_practice/container/run.html

docker rm `docker ps -a | grep Exited | awk '{print $1}'`   删除异常停止的docker容器

docker rmi -f  `docker images | grep '<none>' | awk '{print $3}'`  删除名称或标签为none的镜像

一、如何查看python某个包的版本？
1、pip list // 全部，在里面找

2、pip freeze // 全部

3、pip show numpy // 单个

4、conda list numpy // 单个

二、列出全部outdated（可更新）的包
pip list --outdated --format=legacy

pip list --outdated --format=columns

这两个命令的区别是列表的方式不一样。且他们的命令执行时间都非常的长。

三、更新单个包如numpy
pip install --upgrade numpy

或者

pip install -U numpy

四、更新全部包
需要用到一个叫pip-review的执行程序

首先通过pip下载pip install pip-review

然后执行以下命令：pip-review --local --interactive

或者执行下面python程序

from pip._internal.utils.misc import get_installed_distributions
from subprocess import call
for dist in get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
1
2
3
4
:（ 不过更新全部包不是非常靠谱，毕竟更新一个包都经常出问题！何况是更新所有的。

五、卸载包的方法
和安装的方法类似

pip uninstall numpy
