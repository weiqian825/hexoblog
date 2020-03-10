---
title: 正则表达式RegExp
date: 2018-10-18 08:36:12
categories: 
- web
tags:
- regExp
---

### 一、概念
[RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp) 
[正则表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions)
1. 正则表达式字面量/patterns/flags
```js
// 正则表达式的字面量提供了正则表达式的编译，当正则表达式不变时候，这种方法提供了更好的性能
const regex = /^[a-zA-Z]+[0-9]*\W?_$/gi
```
2. RegExp对象的构造函数,使用构造函数提供正则表达式的运行时编译 new RegExp(pattern [, flags])
```js
let regex = new RegExp(/^[a-zA-Z]+[0-9]*\W?_$/, "gi");
// 当使用构造函数创造正则对象时，需要常规的字符转义规则（在前面加反斜杠 \）
// new RegExp("pattern") 的时候不要忘记将 \ 进行转义，因为 \ 在字符串里面也是一个转义字符
let regex = new RegExp("^[a-zA-Z]+[0-9]*\\W?_$", "gi");
```
3. 特殊字符
```js
/  a. 在非特殊字符之前的反斜杠表示下一个字符是特殊的，不能从字面上解释。 b. 反斜杠也可以将其后的特殊字符，转义为字面量。
^   匹配输入的开始
$   匹配输入的结束
*   匹配前一个表达式0次或多次。等价于 {0,}
+   匹配前面一个表达式1次或者多次
?   匹配前面一个表达式0次或者1次。等价于 {0,1}
.   （小数点）匹配除换行符之外的任何单个字符
(x)     匹配 'x' 并且记住匹配项，就像下面的例子展示的那样。括号被称为 捕获括号
(?:x)   匹配 'x' 但是不记住匹配项。这种叫作非捕获括号
```
### 二、
