---
layout: post
title: String,StringBuilder和StringBuffer的区别
date: 2021-04-19
Author: yufeng 
tags: [Java]
comments: true
toc: true
---

## String，StringBuilder和StringBuffer的区别

#### String

String类中使用final关键字修饰字符数组来保存字符串，在Java9之前采用char数组，9之后采用byte数组来存储字符串

![String继承体系](https://user-gold-cdn.xitu.io/2017/11/6/0e2d6ec2dab9d49cc3daebaa4360f501?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### CharSequence

```java
int length();
char charAt(int index);
CharSequence subSequence(int start, int end);
public String toString();
}
```

##### 成员变量

```java
private final char value[];//final字符数组，一旦赋值，不可更改
private int hash;  //缓存String的 hash Code，默认值为 0
private static final ObjectStreamField[] serialPersistentFields =new ObjectStreamField[0];//存储对象的序列化信息
```

##### 构造方法

```java
 public String(){
  this.value = "".value;
}
//将数组的值初始化为空串，此时在栈内存中创建了一个引用，在堆内存中创建了一个对象
//示例代码
String str = new String()
str = "hello";

public String(String original){
  this.value = original.value;
  this.hash = original.hash;
}
//代码示例
String str=new String("hello");

public String(char value[]){
//将传过来的char拷贝至value数组里面
    this.value = Arrays.copyOf(value, value.length);
}
```

之后再写，先学多线程