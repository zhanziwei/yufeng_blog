---
layout: post
title: Java编程思想的一些总结
date: 2021-04-14
Author: yufeng 
tags: [Java]
comments: true
toc: true
---

### 对象的三大特性

1. 封装

   仅向应用程序员公开必要的内容，并隐藏内部实现的细节。

   作用：有效地避免该工具类被错误的使用和更改，从而减少程序出错的可能。彼此职责划分清晰，相互协作。

   使用方法：使用访问权限修饰符：public，private，protected和default。default默认为包访问，可以被同一包中其他类的成员访问。

2. 继承

   某个类型的对象获得另一个类型的对象的属性和方法。

   使用方法：可以通过实现继承和接口继承。

3. 多态

   一个类实例的相同方法在不同情形有不同表现形式。

   使用方法：将父类或者接口定义的引用变量指向子类或实现类的实例对象。

### 基本数据类型存储

| 基本类型 | 大小    | 最小值    | 最大值         | 包装类型  |
| -------- | ------- | --------- | -------------- | --------- |
| boolean  | —       | —         | —              | Boolean   |
| char     | 16 bits | Unicode 0 | Unicode 216 -1 | Character |
| byte     | 8 bits  | -128      | +127           | Byte      |
| short    | 16 bits | - 215     | + 215 -1       | Short     |
| int      | 32 bits | - 231     | + 231 -1       | Integer   |
| long     | 64 bits | - 263     | + 263 -1       | Long      |
| float    | 32 bits | IEEE754   | IEEE754        | Float     |
| double   | 64 bits | IEEE754   | IEEE754        | Double    |
| void     | —       | —         | —              | Void      |

### BigInteger和BigDecimal

BigInteger支持任意精度的整数，可用于精确表示任意大小的整数值。BigDecimal支持任意精度的定点数字。
### test

