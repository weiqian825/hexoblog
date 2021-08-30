---
title: Jenkins 开发指南
date: 2021-07-30 20:36:36
tags:
---

# Workflow 开发指南

## workflow 介绍资料

workflow: 在 jenkins 上通过 yaml 配置来定义构建部署流程

[jenkins workflow](./workflow.md)
[workflow code](./master/vars/workflowRunner.groovy)

## 已接入 workflow 的项目代码

[xxx-admin-gw](master/deploy/workflow.yml)
[xxx-platform](/projects/gateway/deploy/workflow.yml)
[xxx-service](projects/deploy/workflow.yml)

## 接入 workflow 步骤 

### 第 1 步：改动 deploy/Jenkinsfile

默认的 Jenkinsfile 配置（旧的）

```js
@Library('common-shared-library@xxx') _

node('slave') {
  fePipeline.startDeploy()
}

```

workflow 的 Jenkinsfile 配置（新的）

```js
@Library('common-shared-library@xxx') _

node('slave') {
  workflowRunner.executeWorkflow()
}

```

### 第 2 步： 根据文档编写 workflow 配置，可参考以往有 workflow 的项目

```yml
jobs:
  admin-server:
    env:
      APP_NAME: xxx-admin-service
    steps:
      - name: build image
        use: build-docker-image
        with:
          type: nodejs
          image_tag: xxx-admin-service-$APP_REGION-$APP_ENV
          dockerfile: deploy/Dockerfile
          build_args:
            env: $APP_ENV
            region: $APP_REGION
      - use: deploy-on-k8s
        env:
          env: $APP_ENV
          region: $APP_REGION
        with:
          type: nodejs
          namespace: xxx-frontend-service-$APP_REGION-$APP_ENV
          runtime_env:
            ENV: $APP_ENV
            REGION: $APP_REGION
```

### 第 3 步： 编写 Dockerfile

- case1: 如果是旧项目改造，可以直接从已有项目的 jenkins workspace 拷贝一份[已有的 Dockerfile](https://jenkins.i.xxx.com/job/xxx-frontend-offline-admin-service/620/execution/node/3/ws/deploy/) 到本地 deploy

- case2: 如果是新项目，可自行编写 Dockerfile

### 第 4 步： 确认[jenkins config](https://jenkins.i.xxx.com/job/xxx-frontend-offline-admin-service/configure)配置是正确的 build deploy 分支

## workflow 工作原理-jenkins pipline

pipline 分为 2 种

- 声明式 pipeline 配置
  ![](./images/jenkins-script-pipline2.png)
- 脚本式 pipline，基于 groovy 脚本
  ![](./images/jenkins-cms-pipline.png)
