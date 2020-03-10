---
title: csv问题记录
date: 2018-10-18 16:31:07
tags:
---

### 一、问题
1. 遇到逗号分列，用""号把字段包起来
```js
for (let key in item) {
    item[key] = '"' + item[key] + '"'
}
```
2. 长数字自动变成科学计数法，在字段末尾加上\t，强制分割列
```js
for (let key in item) {
    item[key] = item[key] + '\t'
}
```
所以后台的所有字段最好都统一做一次处理
```js
for (let key in item) {
    item[key] = '"' + item[key] + '\t"'
}
```
3. 编辑再保存csv，多列自动变成了一列，挤到一行
```js
csv编辑之后在保存，需要选择保存格式。【选中第一列，点击菜单栏–数据–分列–分隔符号】
```
4. upload的时候记得设置文件格式(xxxx).csv
   

