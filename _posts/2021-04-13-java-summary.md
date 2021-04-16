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

若希望在堆内存里表示基本类型的数据，则用到它们的包装类，会进行自动装箱和自动拆箱

### BigInteger和BigDecimal

BigInteger支持任意精度的整数，可用于精确表示任意大小的整数值。BigDecimal支持任意精度的定点数字。
### JVM的垃圾回收

引用s在作用域终点结束，但指向的字符串对象依然存在于内存中，Java的垃圾收集器检查所有new出来的对象并判断不可达，来释放那些被占用的内存。

#### 反向URL的作用

所有文件都存在于自己的命名空间中，文件中的每个类都具有唯一标识符。

#### static关键字

1. 修饰静态变量：多个对象共享相同的变量
2. 修饰静态方法：类名.方法名访问方法
3. 修饰静态代码块：类初次加载时候进行初始化

#### 测试对象等价

System.out.println(n1==n2)和System.out.println(n1 != n2);，对于两个Integer对象相等，所以先输出true，后输出false。因为Integer内部维护着一个IntegerCache的缓存，[-128,127]之间的值用==和！=比较也能得到正确结果。

#### 字面值常量

```java
int i1 = ox2f;
int i3 = 0177;
char c = 0xffff;
byte b = 0x7f;
short s = 0x7fff;
long n1 = 200L;
byte blb = (byte)0b00110101;
float f2 = 1f;
double d1 = 1d;
```

