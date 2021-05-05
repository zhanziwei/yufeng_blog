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

BIO是传统的Java IO编程，相关的类和接口在Java.io包下。`同步并阻塞`（传统阻塞型），服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不作任何事情会造成不必要的线程开销。

![BIO](https://segmentfault.com/img/remote/1460000037714809)

#### BIO的底层流程

在阻塞式 I/O 模型中，应用程序在从IO系统调用开始，一直到到系统调用返回，这段时间是阻塞的。返回成功后，应用进程开始处理用户空间的缓存数据。

#### 编程简要流程

1. 服务器驱动一个 ServerSocket。
2. 客户端启动 Socket 对服务器进行通信，默认情况下服务器端需要对每一个客户端建立一个线程进行通信。
3. 客户端发出请求后，先咨询服务器时候否线程响应，如果没有则会等待，或者被拒绝。
4. 如果有响应，客户端线程会等待请求结束后，再继续执行。

#### BIO服务端代码案例

```java
public class Server {

    public static void main(String[] args) throws IOException {
        //创建线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        //创建serverSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        for (; ; ) {
            System.out.println("等待连接中...");
            //监听,等待客户端连接
            Socket socket = serverSocket.accept();
            System.out.println("连接到一个客户端");
            executorService.execute(() -> handler(socket));
        }
    }

    //编写一个handler方法,和客户端通讯
    public static void handler(Socket socket) {
        byte[] bytes = new byte[1024];
        System.out.println("当前线程信息: " + Thread.currentThread().getName());
        try {
            //通过socket获取输入流
            InputStream inputStream = socket.getInputStream();
            //循环读取客户端发送的数据
            while (inputStream.read(bytes) != -1) {
                System.out.println(Thread.currentThread().getName()+ " : 发送信息为 :"+ new String(bytes, 0, bytes.length));
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            System.out.println("关闭连接");
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

##### BIO的缺点

1. 每个请求都需要创建独立的线程，与对应的客户端进行数据处理
2. 当并发数大时，需要创建大量线程处理连接，系统资源占用大
3. 连接建立后，如果当前线程暂时没有数据可读，当前线程会一直阻塞在read操作上，造成线程资源浪费。

#### NIO模型

NIO指Non-blocking IO，从jdk的1.4开始，是同步非阻塞的；相关类都在java.NIO包下，并对原io包中很多类进行改写。

NIO有三大核心部分：channel、Buffer、Selector；NIO 是面向缓冲区编程的。数据读取到了一个它稍微处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞的高伸缩性网络。Java NIO 的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用数据，如果目前没有可用数据时，则说明都不会获取，而不是保持线程阻塞，所以直到数据变为可以读取之前，该线程可以做其他事情。非阻塞写入同理。

#### NIO和BIO的区别

1. BIO 以流的方式处理数据，而 NIO 以块的方式处理数据，块 I/O 的效率比流 I/O 高很多。
2. BIO 是阻塞的，而 NIO 是非阻塞的。
3. BIO 基于字节流和字符流进行操作，而 NIO 基于 Channel（通道）和 Buffer（缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector（选择器）用于监听多个通道事件（比如连接请求，数据到达等），因此`使用单个线程就可以监听多个客户端通道`。

![示意图](https://segmentfault.com/img/remote/1460000037714808)

1. 每个 Channel 对应一个 Buffer。
2. Selector 对应一个线程，一个线程对应多个 Channel。
3. 该图反应了有三个 Channel 注册到该 Selector。
4. 程序切换到那个 Channel 是由`事件`决定的（Event）。
5. Selector 会根据不同的事件，在各个通道上切换。
6. Buffer 就是一个内存块，底层是有一个数组。
7. 数据的读取和写入是通过 Buffer，但是需要`flip()`切换读写模式。而 BIO 是单向的，要么输入流要么输出流。

##### Buffer

Buffer本质上是一个可以读写数据的内存块，为一个容器对象（含数组），该对象提供了一组方法，可以轻松使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、网络读取数据的渠道，但是读取或者写入都必须经过 Buffer。在 Buffer 子类中维护着一个对应类型的数组，用来存放数据。

| Buffer 常用子类 |          描述          |
| :-------------: | :--------------------: |
|   ByteBuffer    |  存储字节数据到缓冲区  |
|   ShortBuffer   | 存储字符串数据到缓冲区 |
|   CharBuffer    |  存储字符数据到缓冲区  |
|    IntBuffer    | 存储整数数据据到缓冲区 |
|   LongBuffer    | 存储长整型数据到缓冲区 |
|  DoubleBuffer   | 存储浮点型数据到缓冲区 |
|   FloatBuffer   | 存储浮点型数据到缓冲区 |

| 属性     |                             描述                             |
| -------- | :----------------------------------------------------------: |
| capacity | 容量，即可以容纳的最大数据量；在缓冲区被创建时候就被指定，无法修改 |
| limit    | 表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作，但极限是可以修改的 |
| position | 当前位置，下一个要被读或者写的索引，每次读写缓冲区数据都会改变该值，为下次读写做准备 |
| Mark     |      标记当前 position 位置，当 reset 后回到标记位置。       |

##### Channel的基本介绍

NIO的通道类似于流，但有如下区别：

1. 通道是双向的可以进行读写，而流是单向的只能读，或者写。
2. 通道可以实现异步读写数据。
3. 通道可以从缓冲区读取数据，也可以写入数据到缓冲区。

##### 使用FileChannel写入文本文件和读取文本文件

```java
public class NIOFileChannel {

    public static void main(String[] args) throws IOException {
        String str = "Hello,Java菜鸟程序员";
        //创建一个输出流
        FileOutputStream fileOutputStream = new FileOutputStream("hello.txt");
        //获取通道
        FileChannel channel = fileOutputStream.getChannel();
        //创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate(100);
        //写入byteBuffer
        byteBuffer.put(str.getBytes());
        //切换模式
        byteBuffer.flip();
        //写入通道
        channel.write(byteBuffer);
        //关闭
        channel.close();
        fileOutputStream.close();
    }
}

public class NIOFileChannel {
    public static void main(String[] args) throws IOException {
      FileInputStream fileInputStream = new FileInputStream("hello.txt");
      FileChannel channel = fileInputStream.getChannel();
      ByteBuffer byteBuffer = ByteBuffer.allocate(100);
      channel.read(byteBuffer);
      System.out.println(new String(byteBuffer.array(), 0, byteBuffer.limit())); //Hello,Java菜鸟程序员
      channel.close();
      fileInputStream.close();
    }
}
```

##### 使用transferFrom 复制文件

```java
public class NIOFileChannel04 {

    public static void main(String[] args) throws IOException {
        FileInputStream fileInputStream = new FileInputStream("hello.txt");
        FileOutputStream fileOutputStream = new FileOutputStream("world.txt");
        FileChannel inChannel = fileInputStream.getChannel();
        FileChannel outChannel = fileOutputStream.getChannel();
        //从哪拷贝,从几开始到几结束 对应的还有transferTo()方法.
        outChannel.transferFrom(inChannel, 0, inChannel.size());
        outChannel.close();
        inChannel.close();
        fileOutputStream.close();
        fileInputStream.close();
    }
}
```

#### Selector的基本介绍

1. Java 的 NIO 使用了非阻塞的 I/O 方式。可以用一个线程处理若干个客户端连接，就会使用到 Selector（选择器）。
2. **Selector 能够检测到多个注册通道上是否有事件发生(多个 Channel 以事件的形式注册到同一个 selector)**，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。
3. 只有在连接真正有读写事件发生时，才会进行读写，减少了系统开销，并且不必为每个连接都创建一个线程，不用维护多个线程。
4. 避免了多线程之间上下文切换导致的开销。

#### Selector的优点

Netty的I/O线程NioEventLoop聚合了Selector，可以并发处理成百上千个客户端连接

当线程从某客户端Socket通道进行读写时，若没有数据可用，该线程可以进行其他任务

线程通常将非阻塞I/O的空闲时间用于其他通道上执行I/O操作，所以单独的线程可以管理多个输入输出通道。

由于读写操作都是非阻塞的，就可以充分提高 I/O 线程的运行效率，避免由于频繁 I/O 阻塞导致的线程挂起。

一个 I/O 线程可以并发处理 N 个客户端连接和读写操作，这从根本上解决了传统同步阻塞 I/O 一连接一线程模型，架构性能、弹性伸缩能力和可靠性都得到极大地提升。

#### NIO非阻塞网络编程过程分析

```java
public class NIOServer {
    public static void main(String[] args) throws IOException {
        NIOServer nioServer = new NIOServer();
        nioServer.initServer();
    }

    private void initServer() throws IOException {
        // 创建serverSocketChannel用来监听客户端的连接请求
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 将serverSocketChannel设置为非阻塞的模式
        serverSocketChannel.configureBlocking(false);
        // 绑定端口号
        serverSocketChannel.socket().bind(new InetSocketAddress(6666));
        // 使用try-with-resources方法创建通道管理器对象Selector
        try(Selector selector = Selector.open()) {
            // 将serverSocketChannel注册到selector上，并监听ACCEPT事件
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            // 这是一个阻塞方法，阻塞到至少有一个通道在注册的事件上就绪了，返回的int值为表示有多少通道已经就绪
            while(selector.select()>0) {
                // selectedKeys访问selected key set中的就绪通道
                Set<SelectionKey> keys = selector.selectedKeys();
                // 创建Iterator对象，进行迭代
                Iterator<SelectionKey> iterator = keys.iterator();
                while(iterator.hasNext()) {
                    // 得到selectionKey对象，该对象代表了注册到该Selector的通道
                    SelectionKey key = iterator.next();

                    if(key.isAcceptable()) {    // 检测此键的通道是否已准备好接受新的套接字连接
                        doAccept(key);
                    } else if(key.isReadable()) {   //检测此键的通道是否已准备好可读取。
                        doRead(key);
                    } else if(key.isWritable()) {   // 检测此键的通道是否准备好可写
                        doWrite(key);
                    } else if(key.isConnectable()) System.out.println("连接成功！"); //检测此键的通道是否已完成或未完成其套接字连接操作。
                    iterator.remove();
                }
            }
        }
    }

    private void doAccept(SelectionKey key) throws IOException {
        // 通过key得到server的channel
        ServerSocketChannel serverSocketChannel = (ServerSocketChannel)key.channel();
        System.out.println("ServerSocketChannel正在循环监听");
        // 接受到此通道套接字的连接
        SocketChannel socketChannel = serverSocketChannel.accept();
        socketChannel.configureBlocking(false);
        // 将key的selector注册到clientChannel中
        socketChannel.register(key.selector(), SelectionKey.OP_READ, ByteBuffer.allocate(1024));
    }

    private void doRead(SelectionKey key) throws IOException {

        SocketChannel channel = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();
        while(channel.read(buffer) != -1) {
            buffer.flip();
            System.out.println(new String(buffer.array(), 0, buffer.limit()));
            buffer.clear();
        }
    }

    private void doWrite(SelectionKey key) throws IOException {
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        byteBuffer.flip();
        SocketChannel clientChannel = (SocketChannel) key.channel();
        // 判断是否还有剩余的数据在缓冲区内
        while(byteBuffer.hasRemaining()) clientChannel.write(byteBuffer);
        // 压缩缓冲区
        byteBuffer.compact();
    }
}



public class NIOClient {
    public static void main(String[] args) throws IOException {
        NIOClient nioClient = new NIOClient();
        nioClient.initClient();
    }

    private void initClient() throws IOException {
        SocketChannel clientChannel = SocketChannel.open();
        clientChannel.configureBlocking(false);
        //连接服务器
        clientChannel.connect(new InetSocketAddress("127.0.0.1", 6666));
        if (!clientChannel.connect(new InetSocketAddress("127.0.0.1", 6666))) {
            while (!clientChannel.finishConnect()) {
                System.out.println("连接需要时间,客户端不会阻塞...先去吃个宵夜");
            }
        }
        String str = "hello,Java菜鸟程序员";
        ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
        clientChannel.write(byteBuffer);
        clientChannel.close();
        System.out.println("客户端退出");
    }
}
```

![img](https://upload-images.jianshu.io/upload_images/752311-7878d027956f7f83.png?imageMogr2/auto-orient/strip|imageView2/2/w/1186/format/webp)

SelectionKey 中定义了四个操作标志位：`OP_READ`表示通道中发生读事件；`OP_WRITE`—表示通道中发生写事件；`OP_CONNECT`—表示建立连接；`OP_ACCEPT`—请求新连接。

* 服务端的ServerSocketChannel用来监听客户端的连接请求，只有1个且端口固定，主要监听accept事件
* 服务端的SocketChannel用来和客户端建立数据读写操作通信，数量与客户端的连接数量一致，每个都分配一个随机的端口，主要监听read事件
* 客户端有一个SocketChannel，用来和服务端通信，主要监听connect和read事件
* 服务端和客户端各有一个Selector，用来监听所有的SocketChannel或者ServerSocketChannel中注册的事件，在没有事件发生时，Selector.select()会被阻塞住。

#### NIO的缺点

NIO需要不断的重复发起IO系统调用，不断的轮询数据是否准备好，会不断地询问CPU，占用大量的CPU事件，系统资源利用率低。

