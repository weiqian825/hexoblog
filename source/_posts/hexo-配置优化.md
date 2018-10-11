---
title: hexo 添加categories|tags|文章预览|搜索等
date: 2018-10-11 16:19:26
categories: 
- tools
tags:
- hexo
---

## 一、添加categories

1. 执行命令 hexo new page categories
```
➜  hexo git:(master) hexo new page categories
INFO  Created: ~/Desktop/shopee/hexo/source/categories/index.md
➜  hexo git:(master) ✗ cat source/categories/index.md
---
title: categories
date: 2018-10-11 15:38:12
type: "categories"
---

```
2. 具体文章添加categories
```
title: hexo 添加categories和tags
date: 2018-10-11 16:19:26
categories: 
- tools
```
3. 修改themes/next/_config.yml里面的配置，去掉categories的屏蔽
```
menu:
  home: / || home
  #about: /about/ || user
  #tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

## 二、添加tags 

1. 执行命令 hexo new page tags
```
➜  hexo git:(master) ✗ hexo new page tags
INFO  Created: ~/Desktop/shopee/hexo/source/tags/index.md
➜  hexo git:(master) ✗ cat  source/tags/index.md
---
title: tags
date: 2018-10-11 16:01:45
type: "tags"
---
```
2. 具体文章添加tags
```
title: hexo 添加categories和tags
date: 2018-10-11 16:19:26
categories: 
- tools
tags:
- hexo
```
3. 修改themes/next/_config.yml里面的配置，去掉categories的屏蔽
```
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```
## 三、文章预览 
1. 修改themes/next/_config.yml文件里面auto_excerpt的enable属性改为true
```
auto_excerpt:
  enable: true
  length: 150
```







