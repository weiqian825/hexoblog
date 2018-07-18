---
title: Hexo 试用记录
date: 2018-07-12 18:56:07
tags:  
---

## 前言

   试用了[Hexo](https://hexo.io/)搭建博客，果然方便啊。文档友好、操作简单、极速安装，技术门槛极低。简单记录下使用中的一些文档资料，已便查阅。(orz先忽略蹭的git资源吧2333)    

  按照下面的文档步骤做完，基本使用功能就ok了，大概花费半个到一个钟,如果后续要设计主题和添加各种功能，选取自己喜欢的插件配置即可。

## 文档

>1. [github极速搭建博客参考教程](http://tengj.top/2016/02/22/hexo1/)
>2. [vscode markdown插件 -- Markdown Preview Enhanced](https://shd101wyy.github.io/markdown-preview-enhanced/#/)
>3. [markdown教程](http://www.markdown.cn/)
>4. [凹凸实验室搭建博客参考教程](https://aotu.io/notes/2015/10/08/aotu-blog-v1/)

## 日常操作
```
hexo n #hexo new命令简写，用于新建一篇文章
hexo g #hexo generate命令简写，用于生成静态文件
hexo s #hexo server命令简写，用于启动服务器进行本地预览
hexo d #hexo deploy命令简写，用于将本地文件发布到GitHub上
hexo clean #清楚缓存文件（db.json）和已生成的静态文件（public）
```

## markdown插入图片
```
markdown插入图片的方法 ![Alt text](image path)
```
>1. 网上图片，只要将图片链接地址写入image path即可
>2. 本地图片，需要将图片存入/source/images/文件夹中，再将相对地址/images/图片名称 写入image path
>3. 由于markdown没有控制图片大小的语法，所以控制图片大小要用```<img>``` 标签实现，例如``` <img src='/images/WechatIMG2.jpeg' style='width: 200px;'/>``` 








