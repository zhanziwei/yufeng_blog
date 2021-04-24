---
layout: post
title: Java对象特性以及数据类型
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

   使用方法：使用访问权限修饰符：public，private，protected和default。default默认为包访问，可以被同一包中其他类的成员访问，并使用包的概念来进行封装。

   * 包的概念

     包内包含一组类，呗组织在一个单独的命名空间中。使用import来导入你要使用的类，使用导入是为了提供一种管理命名空间的机制，所有类名之间相互隔离，使用package和import将单一的全局命名空间分隔开，避免名称冲突。按照惯例，**package** 名称是类的创建者的反顺序的 Internet 域名。

     package首先找出环境变量CLASSPATH，CLASSPATH包含一个或多个目录，用于查找.class文件的根目录。从根目录开始，Java解释器获取包名并将每个句点替换成反斜杠，生成一个基于根目录的路径名，编译器会从CLASSPATH指定的目录中查找子目录。

   * 包访问权限
     1. public：对外部可见
     2. 默认包：不用加任何访问修饰符，其他类在相同包下可访问该成员
     3. private：除了包含该成员的类，其他任何类都无法访问该成员
     4. protected：protected提供包访问权限和子类访问权限。

2. 继承

   某个类型的对象获得另一个类型的对象的属性和方法。

   使用方法：可以通过实现继承和接口继承。

   final数据：对于基本类型，final使数值不变，对于对象引用，final使引用恒定不变。

3. 多态

   多态方法调用允许一种类型表现出与相似类型的区别，只要这些类型派生自一个基类；只有非private方法才能被重写。

   使用方法：将父类或者接口定义的引用变量指向子类或实现类的实例对象；修饰方法：给方法上锁，防止子类通过覆写改变方法；修饰类：不能被继承。
   
   

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

##### 自动装箱和自动拆箱的实现

在装箱的时候自动调用的是Integer的valueOf(int)方法，而在拆箱的时候自动调用的是Integer的intValue的方法。

##### 注意事项

在通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象

```java
public static Integer valueOf(int i) {
        if(i >= -128 && i <= IntegerCache.high)
            return IntegerCache.cache[i + 128];
        else
            return new Integer(i);
}

private static class IntegerCache {
        static final int high;
        static final Integer cache[];

        static {
            final int low = -128;

            // high value may be configured by property
            int h = 127;
            if (integerCacheHighPropValue != null) {
                // Use Long.decode here to avoid invoking methods that
                // require Integer's autoboxing cache to be initialized
                int i = Long.decode(integerCacheHighPropValue).intValue();
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - -low);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
}
```

**Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False。如果超出对应范围仍然会去创建新的对象。**

Double、Float的valueOf方法的实现是类似的。

至于Boolean类：

```java
public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
```

### BigInteger和BigDecimal

BigInteger支持任意精度的整数，可用于精确表示任意大小的整数值。BigDecimal支持任意精度的定点数字。装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的。（xxx代表对应的基本数据类型）。
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
