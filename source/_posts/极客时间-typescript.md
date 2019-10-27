---
title: '[极客时间]typescript'
date: 2019-10-20 15:59:36
tags:
---
## 一、类型基础
```
1. 强类型与弱类型
   强类型语言： 不允许改变变量的数据类型，除非进行强制类型转换
   弱类型语言： 变量可以被赋予不同的数据类型
2. 静态类型与动态类型
   静态类型： 在编译的阶段确定所有变量的类型
      编译阶段确定属性偏移量
      用偏移量访问代替属性名访问
      偏移量信息共享
   动态类型： 在执行的阶段去定所有变量的类型
   JS 在程序运行时，动态计算程序的偏移量
      需要额外空间存储属性名
      所有对象的偏移量信息各存一份

3. ex
       mkdir ts_in_action && cd ts_in_action
    && npm init -y 
    && npm i typescript -g
    && tsc --init
    && mkdir src && touch ./src/index.ts [let: string = 'hello ts']
    && tsc ./src/index.ts
    && tsc ./src/index.ts
```
## 二、类型基础
```
ES6        TS的数据类型
Boolean    Boolean   void
Number     Number    any
String     String    never
Array      Array     元组
Function   Function  枚举
Object     Object    高级类型
Symbol     Symbol
undefined  undefined
null       null

类型注解
  作用： 相当于强类型语言中的类型生命
  语法： (变量/函数):type
  //原始类型
  let bool: boolean = true
  let num:number = 123
  let str:string = 'abc'
  //数组
  let arr1: number[] = [1,2,3]
  let arr2: Array<number | string> = [1, 2, 3, '4']
  // 元组
  let tuple: [number, string] = [0, '1']
  // 函数
  let add = (x:number, y:number): number => x + y
  // 对象
  let obj:{x: number, y:number} = {x: 1, y: 2}
  //symbol
  let s1: symbol = Symbol()
  let s2 = Symbol()
  //undefined,null
  let un: undefined = undefined
  let un: null = null
  //void
  let noReturn = () => {}
  //any
  let x
  // never
  let error = () => {
      throw new Error('error')
  }
  let endless = () => {
      while(true){}
  }
```
## 三、枚举类型
```
枚举： 一组有名字的常量集合

//数字枚举
enum Role {
    Reporter,
    Developer,
    Maintainer,
    Owner,
    Guest
}

//字符串枚举
enum Message{
    Success = '恭喜'，
    Fail = '抱歉'
}
//异构枚举
enum Answer {
    N,
    Y='Yes'
}
//枚举成员
enum Char{
    a,
    b=Char.a,
    c=1+3,
}
//常量枚举
const enum Month{
    Jan,
    Feb,
    Mar
}
let Month = [Month.Jan, Month.Feb, Month.Mar]
```
## 四、接口
```
1. 对象类型的接口
2. 函数类型接口
let add:(x:number,y:number) => number
等价
interface Add{
    (x: number, y: number): number
}
等价类型别名
type Add = (x: number, y:number) => number

let add:Add = (a, b) => a+b
interface Lib {
    (): voide;
    version: string;
    doSomething(): void;
}
function getLib(){
  let lib: Lib = (() => {}) as Lib
  lib.version = '1.0'
  lib.doSomething = () => {}
}
let lib1 = getLib();
lib1();
lib1.doSomething();
let lib2 = getLib()
lib2.doSomething();
3. 与函数相关的知识点
函数重载
function add1(...rest:number[]): number;
function add1(...rest: string[]): string;
function add1(...rest: any[]):string;
4. 类
A继承和成员修饰
B抽象类与多态
C类与接口的关系-接口只能约束类的公有方法
```
## 五、泛型
```
1. 泛型函数
函数重载|联合类型|any型 => 丢失了约束关系
一个函数或者一个类可以支持多种数据类型
泛型：不预先确定的数据类型，具体的类型在使用的时候才能确定
function log<T>(value: T):T{return value}
log<string[]>(['a','b'])
log(['a','b'])
--------------
type Log = <T>(value:T) => T
let myLog:Log = log
--------------
interface Log{
    <T>(value: T): T
} 
let myLog: Log = log
myLog('hahha')

2. 泛型类和泛型约束
interface Length{
    length: number
}
function log()<T extends Length>(value: T): T{
    console.log(value)
    return value
}
```
## 六、类型检查机制
```
1. 类型推断
2. 类型兼容： X兼容Y：X（目标类型） = Y（源类型）
  //接口兼容性： 成员少的兼容成员多的
  interface X{a:any, b:any}
  interface Y{a:any, b:any, c: any}
  let x:X = {a:1,b:2}
  let y:Y = {a:2,b: 2,c:3}
  x = y
  //函数兼容性
  type Handler = (a:number, b:number) = void
  function hof(handler: Handler){
      return handler
  }
  1>参数个数
  let handler1 = (a: number) => {}
  hof(handler1)
  let handler2 = (a:number, b:number, c:number) => {}
  //hof(handler2)
  可选参数和剩余参数

  2>参数类型
  let handler3 = (a: string) = {}
  hof(handle3)

  interface Point3D{x: number; y:number;z:number}
  interface Point2D{x:number;y:number}
  let p3d = (point:Point3D) => {
  }
  let p2d = (point:Point2D) => {}
  p3d = p2d
  p2d = p3d
  函数参数的双向协变

  3>返回值类型
  let f=() => ({name:'Alice'})
  let g= () => ({name:'Alice', location: 'Beijing'})
  f = g
  
  //枚举兼容性
  enum Fruit{Apple,Banana}
  enum Color{Red,Yellow}
  let fruit: Fruit.Apple = 3
  let no: number = Fruit.Apple 

  //类的兼容性 比较结构
  //泛型的兼容性

  当一个类型Y可以被赋值给另外一个类型X时，我们就可以说类型X兼容类型Y 
  结构之间兼容： 成员少的兼容成员多的
  函数之间兼容： 参数多的兼容参数少的

3.TS类型保护机制
  //instanceof
  if(lang instanceof Java){A}else{B}

  // in 
  if('java' in lang){A}
  else{B}

  //typeof
  if(typeof x==='string') {x.length}else{x.toFixed(2)}

  // xxx as

4. 高级类型
   交叉类型-> & 取类型的并集 对象混用
   联合类型-> | 取类成员的交集 灵活使用
   可区分的联合类型
   索引类型-> keyof T
     function getValues<T, K extends keyof T>(obj: T, keys:K[]): T[K][]{
         return keys.map(key => obj[key])
     }
     // keyof T
     interface Obj{
         a: number
         b: string
     }
     let key: keyof Obj

     //T[K]
     let value: Obj['a']
     //T extends U
   映射类型
     interface Obj{
         a: string
         b: number
         c: boolean
     }
     同态
     type ReadonlyObj = Readonly<Obj>
     type PartialObj = Partial<Obj>
     type PickObj = Pick<Obj, 'a'|'b'>
     
     type RecordObj = Record<'x'|'y', Obj>

     条件类型

     type TypeName<T> = 
       T extends string ? 'string':
       T extends number ? 'number':
       T extends boolean ? 'boolean':
       T extends undefined ? 'undefined':
       T extends function ? 'function': "object"

    type T1 = TypeName<string>
    type T2 = TypeName<string | string[]>

    type Diff<T, U> = T extends U ? never: T
    type T4 = Diff<"a"|"b"|"c", "a"|"e">
    type NotNull<T> = Diff<T, undefined | null>

    Exclude
    NonNullable
    Extract

    type T6 = Extract<'a'|'b'|'c', 'a'|'e'>
    //ReturnType<T>
    type T7 = ReturnType<()=> string>
```
## 七、ES6与commonJS的模块系统
```
tsconfig
   target: es5
   module: 'commonjs'  amd umd
 
命名空间
理解声明合并
编写声明文件(umd global lib及插件(模块全局))

```

