---
layout:     post
title:      Learning Chisel and scala
subtitle:   Part I-Scala
date:       2018-11-24
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - Chisel
    - Scala
---

### I. Preface
NVDLA源码分析是个漫长的痛并快乐着的过程，所以忙里偷闲的加入到RISC-V的学习中，想通过阅读几个RV开源处理器的源码，深刻体会一下RV ISA和Chisel. 但当阅读Chisel的官方Cheatsheet时，感觉还是要学习一下`Scala`，一方面，scala作为chisel基础，要玩转chisel，scala必不可少，另一方面，官网的`"A Short Users Guide to Chisel "`，内容太简洁，缺少了语法的一般性定义，编写和调试可能会感觉无从下手，所以有了本文对Scala的介绍.

作者在学习过程中，主要参考了`Learning Scala`和`Programming in Scala, 3rd Edition`，将自认为的核心内容在本文进行介绍，内容编排上做了一些调整，对一些Scala特性会与C++/Java进行一些简单对比，辅助理解，不足与遗漏之处，请路过的小伙伴指出. (Scala的一些参考书在Blog的[books repo](https://github.com/VVViy/VVViy.github.io/tree/master/books/scala)可以找到，仅供个人参考，请勿用于商业用途)

文末作者将对一些容易造成混淆和存在关联性的内容进行简单总结，便于查找区分，如'=>'操作符可应用于Match控制逻辑，也可应用于函数字面量等等.

### II. Data type
1.常量/变量定义
* 常量定义

```scala
Syntax:

//val关键字，scala编译器具备根据<data>推断<type>的能力，所以可以选择省略<type>
val <identifier> [: <type>] = <data>      

Example:

//example 1
scala> val aval = "hello"
aval: String = hello

//example 2
scala> val bval: String = "world"
bval: String = world
```
* 惰性常量

```scala
Syntax:

//lazy关键字修饰val定义惰性常量，只有常量第一次被访问时才计算<data>进行初始化，而非定义时，将在介绍class属性时进一步说明
lazy val <identifier> [: <type>] = <data>
```
* 变量定义

```scala
Syntax:         

//var关键字
var <identifier> [: <type>] = <data>    

Example：

//example
scala> var avar = 1
avar: Int = 1

scala> avar = 2
avar: Int = 2
```

* 转型
  - 与其他强类型语言相似，scala支持低精度向高精度数值类型自动转换
  - 但不支持自动由高精度向低精度截断，需要使用to<Type>方法进行人工转型，如
    
```scala
scala> val cval: Byte = 1
cval: Byte = 1

scala> val dval: Double = cval
dval: Double = 1.0

scala> val eval: Int = dval
<console>: error: type mismatch;

scala> val eval: Int = dval.toInt
eval: Int = 1
```

* 命名规则
  - 构成字符: `字母`，`数字`，`下划线`，`特殊字符`(如*，+，π，φ等，但**不包括[]和.两个**)及`反引号`('ESC'键下面)五类 (注：对于非特殊字符中的周期符号(.)，在对象引用方法时需要留意，scala中例化对象时与java不同，若class没有定义初始化参数，那么例化时不加()，如val cli = new AClass，但在例化的同时引用其中的方法，则必须加括号，如val cli = new AClass().foo)
  - 开头字符: 只能以`字母`，`特殊字符`或\``，如
  
```scala
scala> val π = 3.14159
π: Double = 3.14159

scala> val $ = "USD currency symbol"
$: String = USD currency symbol

scala> val o_O = "Hmm"
o_O: String = Hmm

scala> val 50cent = "$0.50"
<console>:1: error: Invalid literal number val 50cent = "$0.50" ^

scala> val a.b = 25
<console>:7: error: not found: value a val a.b = 25

scala> val `a.b` = 4
a.b: Int = 4
```

2.数值类型
* Scala中包含以下6种数值类型，与其他高级编程语言不同的是，scala中没有`built-in`类型，全部都是`class`.

Table 1. Numeric types

| Name | Description | Size | Min | Max |
|------|-------------|------|-----|-----|
| Byte | Signed integer| 1 byte| -127 | 128 |
| Short | Signed integer | 2 bytes | -32768 | 32767 |
| Int | Signed integer | 4 bytes | -2<sup>31</sup> | 2<sup>31</sup> - 1 |
| Long | Signed integer | 8 bytes | -2<sup>63</sup> | 2<sup>63</sup> - 1 |
| Float | Signed floating point | 4 bytes | | |
| Double | Signed floating point | 8 bytes | | |

* 使用数值类型字符
  在定义变量/常量时，使用各数值类型对应的特殊字符等价于显式指定类型参数，即

Table 2. Numeric literals

| Literal | Type | Description |
|---------|------|-------------|
| 5 | Int | 默认整数类型 |
| 0x0f | Int | "0x"表示16进制，但无8进制字符 |
| 5l | Long | "l" |
| 5.0 | Double | 默认浮点数类型 |
| 5f | Float | "f" |
| 5d | Double | "d" |

```scala
//example
scala> val fval = 5d
fval: Double = 5.0
```

3.Scala核心类型继承体系
  Scala中核心的`Type`体系如Fig-1所示，各类型含义及所有类型支持的方法可简述如下

Table 3. Core nonnumeric types

| Name | Decription | Instantiable |
|--------|-------------|:--------------:|
| Any | Root | No |
| AnyVal | "值"类型抽象根类，运行时分配在heap或stack上 | No |
| AnyRef | 引用类型抽象根类，分配在heap上 | No |
| Nothing | 唯一一个可以作为Type使用的类型，一般在函数或方法异常退出时的返回值类型 | No |
| Null | "空"类型，如val a: String = null 或 val b：String = " ", 表明该a/b未指向内存中任何实例 | No |
| Char | 兼容数值类型 | Yes |
| Boolean | scala中的Boolean类型只有true和false两个值，不支持数值类型向true或false的自动转换 | Yes |
| String | 在scala中属于复合数据类型(collection)，是字符的集合 | Yes |
| Unit | 类似于C++中的void关键字，说明函数或表达式无返回值，可以为常量或变量赋Unit类型值，Unit的表示方法为空括号"()"，如 scala> val nov=(), nov: Unit=() | No |


Table 4. Common type operations

| Name | Example | Description |
|--------|-----------|--------------|
| asInstanceOf[\<type>] | 5.asInstanceOf[Long] | 将调用类型值转换为目标类型，若类型不兼容将报错 |
| getClass | (7.0 / 5).getClass | 返回调用值的类型 |
| isIstanceOf | (5.0).isInstanceOf[Float] | 根据调用类型值与目标类型值是否一致，返回相应布尔值 |
| hashCode | "A".hashCode | 计算特定值的哈希值 |
| to<type> | 20.toByte; 47.toFloat | 主要用于数值类型中高精度向低精度截断后的值 |
| toString | (3.0).toString | 继承自Java的toString方法，由于scala中没有内置类型，皆为class，所有类型值都有该方法 |
    
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%235-%231.jpg?raw=true" height="500" width="477">

Fig-1

</div>

4.Scala运算符：scala支持的运算符和优先级与其他语言相比没有特殊之处，只是**不支持三元运算符(? :)**，详细内容请参考相关材料.
    
### III. Expression and Built-in control structure
1.表达式
* 定义表达式
* 嵌套表达式(Nest)
* 语句(Statements)
2.控制语句
* if...else
* match
* loops
  - for
  
    1）Range复合数据类型
    
    2）iterator guard
    
    3）嵌套迭代器
    
    4）值绑定
    
  - while与do...while

### IV. Functions and Functional programming
1.函数

* 纯函数
* 函数的定义与调用
  - 一般定义形式
  - 无参数定义的两种形式
  - 参数列表
  
    1）参数默认值
    
    2）Vararg参数匹配
    
    3）参数分组
    
  - 函数调用
  
    1）命名参数与位置参数
  
    2）表达式块参数
    
  - 函数返回值
* Procedure
* 递归函数
* 嵌套函数
* 泛型函数

2.函数式编程与电路

3.First-class function
* 高阶函数
* 函数类型与值
* 函数字面量
  - 定义
  - 背后原理
* 占位符
* Partially Applied Functions and Currying
* By-Name Parameters
* Partial Functions
* 函数字面量块参数

### V. Advanced data type: Collections

<div align="center">
    
<img src="">

Fig-

</div>

1.Immutable collections
* String
  - 创建字符串
  - 转义字符与运算符
  - 字符串内插
* Tuple: 三种构造形式
* List
  - List构造
  - 
* Map
* Set

2.Mutable collections

### VI. Classes


### VII. Special classes: Objects, Case classes, and Traits


### VIII. Some tips
| Item | Description |
|------|-------------|
| =>操作符应用|  |
| 可嵌套类型 |  |
| 无名函数/类 |  |
| 函数类型的进化 |  |
| class扩展途径 |  |
| 类内定义的元素 |  |
| 被忽视的符号 |  |
    