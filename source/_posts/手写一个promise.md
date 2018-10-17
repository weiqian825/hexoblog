---
title: 手写一个promise
date: 2018-10-14 10:44:09
tags:
- promise
---

## 一、概念
  promise在javascript早有实现，es6将其写进了语言标准[PromiseA+](https://promisesaplus.com/)，统一了用法，并原生提供了promise对象。
  promise就是一个对象，用来传递一步操作的消息。它代表了某个未来才知道结果的事件(通常是一个异步操作)，并且这个实践提供了统一的api，可供进一步处理。
  promise对象特点
  1. 对象状态不受外界影响。Pending|Resolved|Rejected。只有异步操作结果可以决定当前是哪一种状态，其他操作都无法改变这个状态。
  2. 一旦状态改变就不会再变。Pending->Rejected或者Pending->Rejected，只要其中之一发生，状态就会凝固，不会再变，一直保持这个结果。
  有了promise，就可以讲异步操作以同步操作的流程表达出来，避免了层层的嵌套的回调函数。
## 二、new Promise(executor)
```
function MyPromise(executor) {
    let self = this
    self.state = 'pending'
    self.value = undefined
    self.reason = undefined
    // resolve的作用是将Promise对象状态从 ‘未完成’ 变成 ‘成功’，在异步操作成功时调用，并将异步操作的结果作为参数传递出去
    function resolve(value){
        if(self.state === 'pending'){
            self.state = 'resolved'
            self.value = value
            console.log('resolved func -->',value)
        }
    }
    // reject函数的作用是，将promise对象从‘未完成’变成‘失败’，在异步操作失败时候调用，并将异步操作爆出的错误作为参数传递出去
    function reject(reason){
        if(self.state === 'pending'){
            self.state = 'rejected'
            self.reason = reason
            console.log('reject reason -->',reason)
        }
    }
    executor(resolve,reject)
}
//promise实栗生成时候，可以用then方法分别指定Resolved和Rejected状态的回调函数。
MyPromise.prototype.then = function (onFulfilled, onRejected){
    let self = this
    if(self.state === 'resolved'){
        onFulfilled(self.value)
    }
    if(self.state === 'rejected'){
        onRejected(self.reason)
    }
}

var promise = new MyPromise(function(resolve,reject){
    resolve('resolve hahah')
})
promise.then(function(value){
  console.log('success then -->',value)
},function(reason){
  console.log('failure then -->',reason)
})
```
## 三、基础版本
// 表示某一段时间以后才会发生的结果，过指定时间后，Promsise的实栗状态会变成resolved
function timeout(ms){
    return new MyPromise((resolve, reject) => {
        console.log(Date.now())
        setTimeout(resolve,ms,'resolve value')
    })
}
console.log(Date.now())
timeout(3000).then((value) => {
    console.log(value)
})
// 运行刚才的函数，执行了resolved func，却没有执行到then，原因是什么呢？ 
准确的说，应该是由于setTimeout先执行了then,此时的状态却是pending,相当于什么也不做。等状态在3秒后变成了resolved,onFulfilled因为已经执行过了，所以不在执行了。我们对函数进行如下改造。
```
function MyPromise(executor) {
    let self = this
    self.state = 'pending'
    self.value = undefined
    self.reason = undefined
    self.onResolvedCallbacks = []
    self.onRejectedCallbacks = []
    // resolve的作用是将Promise对象状态从 ‘未完成’ 变成 ‘成功’，在异步操作成功时调用，并将异步操作的结果作为参数传递出去
    function resolve(value){
        if(self.state === 'pending'){
            self.state = 'resolved'
            self.value = value
            console.log('resolved func -->',value)
            self.onResolvedCallbacks.forEach(function(fn){
                fn()
            })
        }
    }
    // reject函数的作用是，将promise对象从‘未完成’变成‘失败’，在异步操作失败时候调用，并将异步操作爆出的错误作为参数传递出去
    function reject(reason){
        if(self.state === 'pending'){
            self.state = 'rejected'
            self.reason = reason
            console.log('reject reason -->',reason)
            self.onRejectedCallbacks.forEach(function(fn){
                fn()
            })
        }
    }
    executor(resolve,reject)
}
//promise实栗生成时候，可以用then方法分别指定Resolved和Rejected状态的回调函数。
MyPromise.prototype.then = function (onFulfilled, onRejected){
    let self = this
    if(self.state === 'resolved'){
        onFulfilled(self.value)
    }
    if(self.state === 'rejected'){
        onRejected(self.reason)
    }
    if(self.state === 'pending'){
        self.onResolvedCallbacks.push(function(){
            onFulfilled(self.value)
        })
        self.onRejectedCallbacks.push(function(){
            onRejected(self.reason)
        })
    }
}
// 表示某一段时间以后才会发生的结果，过指定时间后，Promsise的实栗状态会变成resolved
function timeout(ms){
    return new MyPromise((resolve, reject) => {
        setTimeout(resolve,ms,'resolve value')
    })
}
console.log(Date.now())
timeout(3000).then((value) => {
    console.log(value)
})

```

## 四、promise then


