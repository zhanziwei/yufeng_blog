---
layout: post
title: BIO、NIO和AIO
date: 2021-04-19
Author: yufeng 
tags: [Java]
comments: true
toc: true
---

## Java的各种IO模型

I/O模型可以理解为：使用什么样的通道进行数据的发送和接收，决定了程序通信的性能。

Java支持3种网络编程模型：BIO、BIO、AIO

#### BIO模型

`同步并阻塞`（传统阻塞型），服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不作任何事情会造成不必要的线程开销。

![BIO](https://segmentfault.com/img/remote/1460000037714809)

#### 编程简要流程

等后面再写