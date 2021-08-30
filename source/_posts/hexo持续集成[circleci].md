---
title: 'hexo持续集成[circleci]'
date: 2019-08-23 18:22:43
tags:
---

### 一、相关文档
1. [circleci官网](https://circleci.com)
2. [github](https://github.com)

### 二、配置
1. hexo的配置文件
```deploy
version: 2
jobs:
  build:
    docker:
      - image: circleci/node:8
    steps:
      - add_ssh_keys:
          fingerprints:
            - "65:91:8d:7d:61:20:56:02:37:20:72:fb:7d:d5:4e:49"
      - checkout
      - run: echo "==================== start build =============================="
      - run: mkdir -p my_workspace
      - run: echo "Trying out workspaces" > my_workspace/echo-output
      - persist_to_workspace:
          # Must be an absolute path, or relative path from working_directory
          root: my_workspace
          # Must be relative path from root
          paths:
            - echo-output 
      - run:
          command: |
            git clone https://github.com/theme-next/hexo-theme-next themes/next
      - restore_cache:
          keys:
          - dependencies-{{ checksum "package.json" }}
          - dependencies-
      - run:
          command: yarn
      - save_cache:
          paths:
            - node_modules
          key: dependencies-{{ checksum "package.json" }}
      - run:
          command: |
            yarn build
            git config --global user.name "weiqian825"
            git config --global user.email "540518665@qq.com"
            yarn deploy
      - run: echo "==================== end build =============================="


workflows:
  version: 2
  build-and-deploy:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
```
2. 在github里面添加新的[tokens](https://github.com/settings/tokens),把所有权限都勾选上 
3. [添加sshkey](https://circleci.com/docs/2.0/add-ssh-key/)
   
```js
1. generate the key with ssh-keygen -m PEM -t rsa -C "your_email@example.com"

2.https://circleci.com/gh/[username]/[project]/edit#ssh.添加刚才生产的ssh-keygen xxx

3. https://github.com/[username]/[project]/settings/keys 添加刚才生产的xxx.pub
记得勾选写权限
```
4. 在config.yml里面添加
```js
- add_ssh_keys:
        fingerprints:
          - "65:91:8d:7d:61:20:56:02:37:20:72:fb:7d:d5:4e:49"
```
5. 设置checkout ssh key
```js
在https://circleci.com/gh/[username]/[project]/edit#checkout 
add Deploy key and user key
```

### 三、原理