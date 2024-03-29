---
title: '[极客时间]typescript'
date: 2019-10-20 15:59:36
tags:
---
### 一、类型基础
```sh
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
### 二、类型基础
```ts
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
### 三、枚举类型
```ts
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
### 四、接口
```ts
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
### 五、泛型
```ts
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
### 六、类型检查机制
```ts
1. 类型推断：不需要指定变量的类型，typescript可以根据某些规则自动的为其推断出一个类型。 基础类型推断、最佳通用类型推断、上下文类型推断onkeydown
避免滥用类型断言
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
### 七、ES6与commonJS的模块系统
- es6的模块化系统： 导出export，导入import
- node模块化：导出modules.exports(exports是modules.exports的引用)
```ts
tsconfig
   target: es5
   module: 'commonjs'  amd umd

命名空间
理解声明合并
编写声明文件(umd global lib及插件(模块全局))

```
### 八、tsconfig
```js
1. 与文件相关的选项
files: ['./src/a.ts']//编译器需要编译的文件列表
include: ['src'] //编辑器要编辑的目录，支持通配符
exclude: []//编译器需要排除的文件夹

extends: './tsconfig.base' //继承配置
compileOnSave: true //vscode暂时不支持
2. 与编译相关的选项
compilerOptions: {
  incremental: true, //增量编译
  tsBuildInfoFile: //增量编译文件的存储位置
  diagnostics: 打印诊断信息

  target: //目标语言版本
  module: //生成代码的模块标准
  outFile: //将多个相互以来的文件生成一个文件，可以使用在AMD模块中

  lib: //ts需要引入的库，即声明信息
  allowJs: //允许先变异JS文件（.js, jsx）
  checkJs //允许在JS文件中报错，通常与allowJS一起使用
  outDir: //指定输出目录
  rootDir: //指定输入文件目录

  declaration //自动生成声明文件
  declarationDir //声明文件的路径
  emitDeclarationOnly //只生成声明文件
  sourceMap://生成目标文件的sourceMap
  inlineSourceMap: //生成目标文件的inlineSourceMap
  declarationMap: //生成目标文件的sourceMap
  typeRoots //生命文件的目录，例如默认 node_modules/@types
  types //需要加载的声明的文件包
  removeComments //删除注释
  noEmit //不输出文件
  noEmitOnError //发生错误时候不输出文件
  removeComments //删除注释
  downlevelIteration //降级遍历器的生成
  noEmitHelpers //不生成helper函数，需要额为安装ts-helpers
  importantHelper //通过tslib引入help函数， 文件必须是模块
  strict //开启严格模式
  alwaysStrict //
  noImplicitAny //不允许any类型
  strictNullChecks //不允许把null undefined赋值给其他变量类型
  strictFunctionTypes //不允许函数的双向协变
  strictPropertyInitalization //类的实例必须初始化
  strictBindCallApply //严格的bind/call/apply检查
  noImplicitThis //不允许this有隐式的any类型

  noUnusedLoacls //检查之生命，未使用的局部变量
  noUnusedParameters //检查未使用的函数参数
  noFallthroughCasesInSwitch //防止switch语句贯穿
  noImplicitReturns //每个分支都要又返回值
  esModuleInterop //允许export = 导出， 由important from 导入
  allowUmdGlobalAccess //允许在模块中访问UMD全局变量
  moduleResolution //模块解析策略
  baseUrl //解析非相对模块的基地址
  paths: {
      jquery: []
  }
  rootDirs //将多个目录放在一个虚拟目录下，用于运行时
  listEmittedFiles //打印输出的文件
  listFiles //打印编译的文件
}
3. 工程引用

```
### 九、编译工具
```js
1. 编译工具: ts-loader awesome-typescript-loader
2. 代码检查工具： eslint
typescript=> 类型检查+语言转换
3. jest单元测试
4. 函数与类组件
```
### 十、ts-js共存，对于旧项目的改造迁移
```js
1. 共存策略<ts和js并存>
2. 宽松策略<将js重命名魏tsx, 修改ts规则让编译通过>
3. 严格策略<改成真正的ts>
```
### 十一、weather demo
1. 初始化项目
```
mkdir projectDemo
code projectDemo
touch index
yarn init
yarn add ts-node typescript -d
//配置tsconfig
yarn tsc --init
//配置tslint(js-eslint, ruby-rubocop)
yarn add tslint -d
yarn tslint --init
//配置git
touch .gitignore
//配置husky
yarn add husky -d
添加husky的hooks
  "husky": {
    "hooks": {
      "pre-commit": "yarn tslint -c tslint.json 'src/**/*.ts'"
    }
  }
//
```
2. Commander.js
```
yarn add commander
yarn add @types/node
```
3. 添加色彩
```
yarn add color
yarn add axios
```


