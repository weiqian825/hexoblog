---
title: formily工作原理
date: 2020-03-16 21:16:30
tags:
---

### Outline

What is form

How would you make a form

Why formily

Demo

### What is form

##### form = validator + ui 

场景归纳：组件数据收集&校验&更新的流程是一致。

流程抽象：通过配置屏蔽组件间的差异性，对组件的配置规则统一管理。

常见form方案 youform (jsx 语法方式来声明表单)

![Alt text](../images/ayouform.png)

![Alt text](../images/qqschemaform.png)

mozilla  jsonSchemaForm(通过 json 描述来构建表单，支持了配置化方式来构建表单)

 https://github.com/rjsf-team/react-jsonschema-form

![Alt text](../images/mozschemaform.gif)

Ant form 

 https://github.com/ant-design/ant-design/tree/master/components/form (antform)

https://github.com/react-component/form （rc-form）

![](../images/antformdemo.png)

redux-form作者finalform: https://github.com/final-form/final-form







formily = JSON Schema(JSchema)** + **字段分布式管理** + **React EVA**

##### 或许formliy可以和这些有关系

![Alt text](../images/formcando.png)





