---
title: treeform表单设计
date: 2019-01-22 14:32:01
categories: 
- tools
tags:
- 表单
- treeform
---

### 一、用法示例

1. 组件调用方式
```js
  Props     | Type         | use
  --------- | ------------ | ---------  
  treeData  | object|array | 业务数据来源
  schemaData| object       | UI形态设计(表单)
  onError   | function     | 类型校验错误处理函数，回传错误info
  onChange  | function     | 回传treeData的最新状态，业务里需要写一个action改变store
  onSubmit  | function     | 回传当前提交表单下的数据

  <TreeForm
    treeData={treeData}
    schemaData={noticeSettingSchema}
    onError={this.onError}
    onChange={this.onChange}
    onSubmit={this.onSubmit} 
  />
```
2. const/notice业务的schema，主要设计的式表单相关
此处的设计遵循 [JSON schema](http://json-schema.org/)
有2个作用 A.自动生成表单  B.利用jsonschema校验基础类型
```js

  showField  | Type    | use
  ---------- | ------- | -----------------------------------------------  
  type       | string  | 数据的类型object|array|string 
  field      | string  | UI的展示形式 [text|select|time|textarea ]
  properties | object  | object类型的表单数据组织
  title      | string  | 表单的title
  placeholder| string  | input的默认placeholder
  
  
  validator | Type     | use
  --------- | -------- | -----------------------------------------------  
  required  | array    | 设置当前表单的必填字段
  maxItems  | number   | 超过最大长度会触发onError,不触发onSubmit
  maxLength | number   | 超过最大长度会触发onError,不触发onSubmit
  minLength | number   | 小与最小长度会触发onError,不触发onSubmit
  pattern   | RegExp   | 校验数据类型，会触发onError,不触发onSubmit
  
  
  action    | Type     | use
  --------- | -------- | ----------------------------------------------- 
  canSubmit | boolean  | 表单页面是否出现submit按钮，默认false，不出现
  canDel    | boolean  | 当前表单列表是否允许删除，默认false，不删除
  canAdd    | boolean  | 当前表单列表是否允许新增，默认false，不新增
  

  const noticeSettingSchema = {
    type: 'object',
    field: 'object',
    canSubmit: true,
    properties: {
      children: {
        type: 'array',
        field: 'array',
        maxItems: 50,
        defaultItemName: 'Notice',
        canAdd: true,
        canDel: true,
        items: {
          type: 'object',
          field: 'object',
          required: [ 'name', 'status', 'startTime', 'endTime', 'cycleType',   'text' ],
          properties: {
            name: {
              type: 'string',
              field: 'text',
              title: 'Name',
              maxLength: 20,
              pattern: /.*\S+.*/
            },
            status: {
              type: 'string',
              field: 'select',
              title: 'Status',
              data: [{
                title: 'Shown',
                value: 'shown'
              }, {
                title: 'Hidden',
                value: 'hidden'
              }],
              defaultValue: 'shown'
            },
            startTime: {
              type: 'string',
              field: 'time',
              title: 'Start Time',
              defaultValue: '00:00'
            },
            endTime: {
              type: 'string',
              field: 'time',
              title: 'End Time',
              defaultValue: '23:59'
            },
            cycleType: {
              type: 'string',
              field: 'select',
              title: 'Cycle',
              data: [{
                title: 'Every Day',
                value: 'EveryDay'
              }],
              defaultValue: 'EveryDay'
            },
            startDate: {
              type: 'string',
              field: 'datePicker',
              title: 'Start Date'
            },
            endDate: {
              type: 'string',
              field: 'datePicker',
              title: 'End Date'
            },
            text: {
              type: 'string',
              field: 'textarea',
              title: 'Text',
              maxLength: 70,
              placeholder: 'Please input text',
              pattern: /.*\S+.*/
            }
          }
        }
      }
    }
  }
```
3. notice的treeData数据
```js
  showField  | Type                | use
  ---------- | ------------------- | ----------------------------------------  
  id         | string              | 必填，节点的唯一标识 
  name       | string              | 必填，节点的展示标识
  toggled    | boolean             | 树节点是否展开，true为展开，false为关闭(默认)
  active     | boolean             | 展示的当前节点，对应节点的object的active为true
  childrenKey| string              | 当前树子节点的key
  data       | array|object|string | 当前数据为array时候，组件会自动增加childrenKey

  {
    id: 0,
    name: 'notice',
    toggled: true,
    children: [{
        name: "Notice 8",
        status: "shown",
        start_time : "00:00",
        end_time : "23:59",
        start_date : "",
        end_date : "",
        text : " 35",
        cycle_type : "EveryDay",
        create_time : "2019-01-22 13:55"
      },
      {
        name : "Notice 7",
        status : "shown",
        start_time : "00:00",
        end_time : "23:59",
        start_date : "",
        end_date : "",
        text : " grge",
        cycle_type : "EveryDay",
        create_time : "2019-01-22 13:55"
    }]
  }
```
4. [notice demo访问地址](dp-admin.test.xxxxx.io/id/notice/detail/1)

### 二、实现思路

[实现思路流程图](https://www.processon.com/view/link/5c47069ae4b03334b50c3f34)

### 三、待优化部分

1. id和name的设计

2. 数据同步问题

3. 关键函数的实现

   

### 四、问题

1.某某这个页面需要填写用户信息？

这几个输入框能填写正常信息就行

2.so easy？

Bug太磨人，这个光标，这个键盘，那个校验

3.听说你解过这个bug，我的邮箱校验好像漏了个啥

抄作业

🈶️

4.奋发图强，做一个通用方案，不让同事在踩坑？

sniper ota和nearby里的@comp/Common/DataEntry/Form

Windrunner 