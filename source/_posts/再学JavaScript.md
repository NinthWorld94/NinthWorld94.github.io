---
title: 再学JavaScript系列--值类型跟引用类型到底有什么区别？值类型存在栈上面上一定正确吗？
date: 2021-02-23 10:37:17
categories: ['技术探讨']
tag: ['JavaScript',V8,堆栈模型]
comments: true
---
分析造成引用类型跟值类型的行为差异的底层原因，V8堆栈模型浅析，以及对一些数据类型的存储机制解析
<!-- more -->

## 值类型和引用类型的特点
根据JS的定义，或者说ECMAScript的定义总共有八种语言类型：`null`,`undefined`,`number`,`boolean`,`string`,`symbol`,`bigint`,`object`
其中除了 `object` 其他都是值类型，我们用一段代码来概括其区别:  
```
let a = 1;
let b = a;
a = 2 
console.log(b); // 1
let c = {x: 1};
let d = c;
c.x = 2
console.log(c.x); // 2
let e = 2 
let f = 2
e === f // true 
let g = {x: 1};
let h = {x: 1};
g === h // false
```
## 数据在堆栈中的存储

对于以上现象，先用通俗的解释：  
对于值类型，其在内存中存储在一个栈当中其中key是变量名，value是具体的值。因此`b = a`成立因为value完全一致嘛。  
而对于引用类型，`let c = {x: 1}` 这行代码先在堆上面开辟一块内存空间,其中存储着 `{x:1}`这条数据，然后对于变量`c`其value中存储着这个空间的地址，因此`d = c`的赋值操作其实是复制了一份地址，而不是数据，二者都指向堆里面的`{x: 1}`数据，因此当修改二者其中之一时，另一个变量随之改变。对于`g`跟`h`两个变量，二者虽然数据一摸一样，但是二者不是同一块内存空间，修改`g`中的`x：1`不会改变`h`中的`x`因此不相等。  
图解如下：  
![](/source/images/srack&heap.jpg "堆栈")
## 堆栈的空间结构
栈的空间结构没什么好讲的，就是一块连续的存储空间，再看看堆的存储结构张图:
![](/source/images/heap.jpg "堆内存结构")
其中：  
* 新生代内存区（new space）  
新生代内存区会被划分为两个semispace，每个semispace大小默认为16MB也就是说新生代内存区通常只有32MB大小（64位），而这两个semispace分别是from space 和 to space（具体有什么用下文会说），通常新创建的对象会先放入这两个semispace中的一个。

* 老生代内存区（old space）  
通常会较为持久的保存对象,也分为两个区域 old pointer space 和 old data space分别用来存放GC后还存活的指针信息和数据信息。

* 大对象区（large object space）  
这里存放体积超越其他区大小的对象，主要为了避免大对象的拷贝，使用该空间专门存储大对象。

* 单元区、属性单元区、Map区（Cell space、property cell space、map space）  
Map空间存放对象的Map信息也就是隐藏类(Hiden Class）最大限制为8MB；每个Map对象固定大小，为了快速定位，所以将该空间单独出来。

* 代码区 (code Space)  
主要存放代码对象，最大限制为512MB，也是唯一拥有执行权限的内存
  

## V8引擎中的基本类型存储机制

### 一个疑点

根据上文所说，对于基本类型应该是存储在栈中的，但是V8引擎中一个栈区的大小为984kib 理论上一个字符串的大小不会超过这个数字，但是实际操作中却能声明一个上百mib大小的字符串，如图：  

![](/source/images/bigString.jpeg "堆内存结构")

### V8引擎`string`的真正存储机制
因此实际的V8引擎中数据绝不仅仅是简单的栈存储。
先说结论，实际上对于 `string`，先从内存中（哈希表）查找是否有已经创建的完全一致的字符串，如果存在，直接复用。如果不存在，则开辟一块新的内存空间存进这个字符串，然后把地址赋到变量中。同时，这也解释了为啥V8里字符串不能通过下标修改字符串，因为人家本来就是不可变的。
我们直接来看一下一段V8的源码验证下： 

![](/source/images/stringTable.png "V8 StringTable源码")
![](/source/images/getName.jpeg "V8 StringTable源码")
![](/source/images/newString.jpeg "V8 StringTable源码")

因此其实我们声明两个相同的字符串时，两个变量的地址（hash）其实是一样的
```
const BasicVarGen = function () {
    this.s0 = 'IAmString'
    this.s1 = 'IAmString'
}


let a = new BasicVarGen()
let b = new BasicVarGen()
//a b中的s0 s1是同一个内存地址（hash）
```
### V8引擎`number`的存储机制

`number`在V8中分为 `smi` 和 `heapNumber`,  
`smi` 直接存进内存,范围为 ： -2³¹ 到 2³¹-1（2³¹≈2*10⁹）的整数  
`heapNumber` 类似字符串,不可变,范围为 ：所有非smi的数字，存进堆里面  

ECMAScript 标准约定number数字需要被当成 64 位双精度浮点数处理，由于一直使用 64 位去存储任何数字实际是非常低效的（空间低效，计算时间低效 smi大量使用位运算），所以 JavaScript 引擎并不总会使用 64 位去存储数字，引擎在内部可以采用其他内存表示方式（如 32 位），只要保证数字外部所有能被监测到的特性对齐 64 位的表现就行。

## 总结
值类型跟引用类型在实际使用中需要注意一些使用上的差异，这种差异根据JavaScript引擎会有不同的实现，在V8中并不是所有的值类型都存储在栈内存中，对于V8引擎来说:  
* `string`： 存在堆里，栈中为引用地址，如果存在相同字符串，则引用地址相同。

* `number`： 小整数存在栈中，其他类型存在堆中。

* 其他类型：引擎初始化时分配唯一地址，栈中的变量存的是唯一的引用。

(参考链接：https://juejin.cn/post/6844904175868837901)  
(参考链接：https://www.zhihu.com/question/482433315/answer/2083349992?utm_source=wechat_session&utm_medium=social&utm_oi=1118537826173702144&utm_content=group1_Answer&utm_campaign=shareopn)








