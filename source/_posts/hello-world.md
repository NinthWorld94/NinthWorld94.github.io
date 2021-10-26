---
title: Hello World
---
## 一.数据类型

在es5之前总共有：*null* *undefined* *boolean* *object* *function* *number* 六种数据类型，es6新增了 *symbol* ,es11新增了 *bigint* 

所以截止es2021，目前总共有：*null* *undefined* *boolean* *object* *function* *number* *symbol* *bigint* 基本数据类型

ps：这些也是 `typeof` 能返回的所有值

### 数据类型判断的三个原生方法

#### 1.`typeof`
数值、字符串、布尔值分别返回number、string、boolean，函数返回function，undefined返回undefined，对象返回object，null返回object。

ps：`typeof null // object` 其实是历史原因,1995年的 JavaScript 语言第一版，只设计了五种数据类型（对象、整数、浮点数、字符串和布尔值），没考虑null，只把它当作object的一种特殊值。后来null独立出来，作为一种单独的数据类型，为了兼容以前的代码，typeof null返回object就没法改变了。

#### 2.`instanceof` 
`instanceof` 运算符返回一个布尔值，表示对象是否为某个构造函数的实例。

`instanceof` 是基于原型链查找，运算符的左边是实例对象，右边是构造函数。它会检查右边构造函数的原型对象（prototype），是否在左边对象的原型链上。因此，下面两种写法是等价的。
```
const v = new Vehicle()
v instanceof Vehicle
// 等同于
Vehicle.prototype.isPrototypeOf(v)
```
#### 3.`Object.prototype.toString`
因为大部分数据类型都自定义重写了toString方法，一次可以根据此方法的返回值去判断数据类型
```
数值：返回[object Number]。
字符串：返回[object String]。
布尔值：返回[object Boolean]。
undefined：返回[object Undefined]。
null：返回[object Null]。
数组：返回[object Array]。
arguments 对象：返回[object Arguments]。
函数：返回[object Function]。
Error 对象：返回[object Error]。
Date 对象：返回[object Date]。
RegExp 对象：返回[object RegExp]。
其他对象：返回[object Object]。
symbol对象：返回Symbol()
```

## 二.运算符
加法运算符：`x + y`   
减法运算符： `x - y`  
乘法运算符： `x * y`  
除法运算符：`x / y`  
指数运算符：`x ** y`  
余数运算符：`x % y`  
自增运算符：`++x 或者 x++`  
自减运算符：`--x 或者 x--`  
数值运算符： `+x`  
负数值运算符：`-x`  


运算符号值得说的其实就是 `+` 运算符，加法运算符是在运行时决定，到底是执行相加，还是执行连接。也就是说，运算子的不同，导致了不同的语法行为，这种现象称为“重载”（overload）。
比如：
```
1 + true // 2
'3' + 4 + 5 // "345"
3 + 4 + '5' // "75"
```
指数运算符 `**` 
```
2 ** 4 // 16
```
指数运算符需要注意的是，指数运算符是右结合，而不是左结合。即多个指数运算符连用时，先进行最右边的计算。
```
// 相当于 2 ** (3 ** 2)
2 ** 3 ** 2
// 512
```


### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
