---
title: NIO 与 Reactor模式
date: 2020-07-07 18:54:25
tags: Java基础
categories: Java
---

不喜欢太多高大上的理论，记不住……

## 〇、前置知识

#### 1、什么是IO？
答：io ， input，output 。 一句话，**IO就是为了完成数据传输**。 指的是，完成程序与其他载体的数据传输，可以是硬盘、进程、缓存等。 因为完成程序与其他载体， 程序为本体，往程序里面写是in， 往外部写是out。
（其实，进程缓冲区 和 内核缓冲区分别“挡”在内存和硬盘前面， IO操作先写缓冲区。再由内核将缓冲区写入硬盘…）

#### 2、什么Socket？
答 ： 哎，我一直说不清 。 只知道， Socket是两个进程之间双向通信的方式，支持不同的通信协议TCP UDP，表示方式 ： IP地址 + 端口号。 说是可以看成两个进程通信的端点，然后里面还能放入数据！ 

#### 3、什么是IO模型？
答：指的是Linux系统IO模型。分为阻塞IO、非阻塞IO、多路复用IO、信号驱动IO以及异步IO。这里的同步/异步、阻塞/非阻塞针对的都是用户程序。（[此文](https://www.cnblogs.com/crazymakercircle/p/10225159.html)对于同步异步的解释是有偏差的，我更倾向于下面这种解释，程序缓冲区是由 *程序接收系统内核数据，由程序写入完成IO操作就应该是同步IO；如果是系统内核准备好数据写进程序缓冲区，再通知程序，那么这次IO是由系统完成的，程序线程可以做其他操作，应该算异步IO* ）

> 实际IO操作由应用程序本身执行，会阻塞线程就是同步IO。反之由操作系统完成就是异步IO。（《疯狂java》）

简述一下，linux系统的虚拟内存空间被划分为两部分： **内核内存空间与用户内存空间**。这里的用户空间就是运行的程序进程， 由于**内核与用户空间的数据交换非常费时**， 所以在两部分分别设置了缓冲区。用户的缓冲区是IO库用buffer优化的，而内核的buffer是自带的，并且内核向硬盘的写入操作是我们不能控制的。所谓的Linux IO模型就是在内核空间与用户空间的数据交换时起作用。
<!-- more -->
无论什么样的IO模型，`硬盘 - （准备数据耗时）-内核缓冲区 - （复制耗时） - 用户缓冲区` 这个过程都是无法优化的。所谓的同步模型，是指用户线程在这个等待阶段阻塞，什么都干不了只能等结果！异步模型是指等待的过程中，用户进程可以干别的；结果好了， 内核再通知你。至于，阻塞/非阻塞指的是用户进程这段时间有没有干别的事情，还是挂起等在原地。

（进程指应用， 线程指单个socket，下文用线程来说明了。并且，有的应用代表举例，使用的是原理一致的WEB IO 用例 ）

同步阻塞IO ：应用代表---Java默认的socket都是阻塞的。等待的时候，用户线程挂起不吃CPU资源。线程之间切换吃内存资源，高并发要开好多线程维护连接，用户线程此时还啥也干不了，不适用高并发。

同步非阻塞IO ：None Blocking IO ， 不是 New IO！！这种IO像智障一样，用户线程采用“轮询”的方式，一遍一遍问系统buffer：“数据准备好了吗？”  确实是不阻塞了，但用户线程也没闲置下来！本质跟阻塞差不多。可以通过设置socket使其变为non-blocking实现这种NIO。

多路复用IO ：应用代表---  Java NIO（New IO）、 Redis 采用IO多路复用对于Socket的监听。用户线程准备完毕就更改状态，说 ： “自己准备请求数据，准备获得连接” ，进入请求连接模式。系统内核会单开一条线程，不断轮询Socket 列表， 看看哪一个socket状态为准备完毕，跟它连接上，连接后用户线程开始阻塞！内核处理请求将数据写入用户缓冲区并通知用户。 我们发现，一旦建立连接，就变为上一种NIO！

异步非阻塞IO ：应用代表--- Nodejs （事件驱动非阻塞IO模型）。 用户发个请求，然后就可以干别的了。内核处理好了，再通知用户来取结果。


#### 4、热身： 遍历文件夹下所有文件（尽管 ，这在WEB开发中没什么用 2333）
常规递归 ：
```
    public void print1(String path){
        travel(new File(path));
    }
    private void travel(File input){
        if (!input.exists()) return;
        Arrays.stream(input.listFiles()).forEach(file -> {
            if (file.isDirectory()) {
                travel(file);
            }else {
                System.out.println(file.getPath());
            }
        });
    }
```
NIO提供新接口
```
    public void print2(String path) throws Exception{
        Files.walkFileTree(Paths.get(path),new SimpleFileVisitor<Path>(){
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                System.out.println("正在访问文件：" + file);
                return super.visitFile(file, attrs);
            }

            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
                System.out.println("正在访问路径：" + dir);
                return super.preVisitDirectory(dir, attrs);
            }
        });
    }
```

## 一、 传统io区别

1、NIO：面向 buffer 缓冲区 ，传统IO : 面向流

2、NIO：缓冲区双向的 ， 传统IO : 单向的 

3、NIO ： 通道Channel  仅仅用于连接，需要由缓冲区buffer来承载数据
传统IO：建立管道pipe 可以承载二进制流数据，输入输出流 

4、NIO ： 非阻塞式 io ， 传统IO：阻塞式io  

5、NIO有选择器Selector。

（4、5 网络编程时使用）


## 二、NIO核心

1、通道Channel   

Channel 模拟 传统Io的流操作，只不过这次Channel内不是Stream而是一块区域Buffer。


2、缓冲区 Buffer

Buffer 是一个容器，底层是数组。额外说明一点，NIO允许将内存的一部分映射map到buffer上。

## 三、 缓冲区 操作

#### 1、简介

底层就是数组 ， 不同基本类型都有不同的buffer对应。最常用的是 ByteBuffer，还有CharBuffer，LongBuffer等继承自Buffer抽象类。


#### 2、管理方式 
1) 分配缓冲区大小
eg :   CharBuffer.allocate()

2) 存取方式 
put()
get()


3) 核心属性

capacity   :  存储数据容量  ， 申请后不可修改

limit ：limit 后面的数据不可进行读写

position ： 正在操作数据的位置

mark() : 标记  position 位置，position可以直接定位到Mark处。

>满足关系 ：0<= mark <= position <= limit <= capacity

4) 读、写数据模式切换

flip()
>Flips this buffer. The limit is set to the current position and then the position is set to zero. If the mark is defined then it is discarded.

可见，flip将limit设置到position，将position设置到0。可以理解为是“ 读写数据模式切换 ”， 将position归位，limit限制读到null。 

rewind()
重读

clear()
清空缓冲区，注 数据还在，但数据处于被遗忘状态 （通过调整limit 指针，后面的读不到）


reset()  
恢复到 mark标记的位置

（还有很多其他方法   ）


#### 3、直接缓冲区与非直接缓冲区

1) allocate()  
开辟空间在 jvm 堆内存中。


2) allocateDirect() 

直接建立在磁盘中，提高效率，只支持 byteBuffer。通过物理内存映射文件 ，再由操作系统控制写入硬盘，一定程度上提高效率。对于大量数据，消耗资源。

注： isDirect() 判断buffer是否是直接缓冲区

## 四、通道（Channel）

本质跟流没啥区别， 直观上看输入流与输出流合二为一，可以节约cpu 的io资源。

#### 1、常用实现类

java.nio.channels.Channel 
父类接口

1.1 文件IO

FileChannel
文件通道

1.2 网络IO ：

SocketChannel  
 TCP 客户端

ServerSocketChannel
 TCP 服务器

DatagramChannel
 UDP客户端

#### 2、获取通道  

支持通道的类，即可调用getChannel()获取通道
getChannel()

注 ：支持通道的类举例 
本地 ：
FileInputStream/FileOutputStream
RandomAccessFile

网络 :
Socket
ServerSocket
DatagramSocket

其他方式获取Channel ： 
jdk1.7之后通过  open()  
Files工具类 newByteChannel()

#### 3、“通道 + 缓冲区” 完成文件复制 (例： 直接缓冲区)

```
    public void copy()  {
        FileChannel inChannel = null;
        FileChannel outChannel = null;
        try {
             inChannel = FileChannel.open(Paths.get("C:\\Users\\Alex\\Desktop\\集合框架.gif"), StandardOpenOption.READ);
             outChannel = FileChannel.open(Paths.get("C:\\Users\\Alex\\Desktop\\集合框架2.gif"), StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE_NEW);

            MappedByteBuffer inMappedBuf = inChannel.map(FileChannel.MapMode.READ_ONLY,0,inChannel.size());
            MappedByteBuffer outMappedBuf = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());

            byte[] dst = new byte[inMappedBuf.limit()];
            inMappedBuf.get(dst);
            outMappedBuf.put(dst);
            
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            if (inChannel != null){
                try {
                    inChannel.close();
                }catch (IOException e){
                    e.printStackTrace();
                }
            }
            if (outChannel != null){
                try {
                    outChannel.close();
                }catch (IOException e){
                    e.printStackTrace();
                }
            }
        }
    }
```
#### 4、使用通道之间的数据传递 ，简化操作

transferFrom()
transferTo()

```
   public void copy2() throws IOException {
        FileChannel inChannel = FileChannel.open(Paths.get("C:\\Users\\Alex\\Desktop\\集合框架.gif"), StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get("C:\\Users\\Alex\\Desktop\\集合框架2.gif"), StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE_NEW);

        inChannel.transferTo(0, inChannel.size(), outChannel);

        inChannel.close();
        outChannel.close();
    }

```



#### 5、分散读取与聚集写入

channel 可以通过 read()  、write() 方法来读取写入缓冲区数据。当接受参数 Buffer[] 时， 则可实现分散读取/聚集写入功能。



## 五、 网络通信的阻塞与非阻塞

#### 1、阻塞 IO

通过多线程 “缓解” 线程阻塞问题

#### 2、非阻塞 NIO

提出概念  ： 选择器 Selector ，即前置知识提到的“IO多路复用”的落地实现。

`client --> selector --> server`

每个希望用非阻塞方式通信的client 都应将channel 注册到selector上，由selector 监控 client 的 channel 状态， 当channel 准备就绪后， 再由selector将请求分配到一个或者多个线程上。

使用 SelectionKey 称为“选择键” 来**表示注册到选择器的channel的状态** ：
1  read
4 write
8 connect
16 accept

#### 3、 使用NIO完成网络通信 ： 阻塞式 、 非阻塞式

Channel  +  Buffer  +  Client 演示 ， 环境为本机 + 虚拟机 ，端口信息如下：
```
Client  ip: 192.168.31.90
Server ip : 192.168.31.134
server port: 9999
```

1) 阻塞式
```
//-----------client--------------
/**
     * 阻塞式IO
     * 发送 C:\Users\Alex\Desktop\集合框架.gif
     * 到 server
     */
    public void client() throws IOException {
        SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("192.168.31.134",9999));

        FileChannel inChannel = FileChannel.open(Paths.get("C:\\Users\\Alex\\Desktop\\集合框架.gif"), StandardOpenOption.READ);

        ByteBuffer buf = ByteBuffer.allocate(1024);

        while (inChannel.read(buf) != -1){
            buf.flip();
            sChannel.write(buf);
            buf.clear();
        }

        inChannel.close();
        sChannel.close();
    }

//-----------server--------------

public void server() throws IOException {
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        FileChannel outChannel = FileChannel.open(Paths.get("copy.gif"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);
        //bind server port
        ssChannel.bind(new InetSocketAddress(9999));

        ByteBuffer buf = ByteBuffer.allocate(1024);

        //get client Channel
        SocketChannel sChannel = ssChannel.accept();

        while (sChannel.read(buf) != -1){
            buf.flip();
            outChannel.write(buf);
            buf.clear();
        }

        ssChannel.close();
        sChannel.close();
        outChannel.close();
    }

```

2) 非阻塞式（这个demo可以作为聊天的模型框架，朴素Reactor模式）

```
//----------client----------------
/**
     * 非阻塞式IO
     * 发送 C:\Users\Alex\Desktop\集合框架.gif
     * 到 server
     */

    public void client2() throws IOException {
        SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("192.168.31.134",9999));

        // 切换到非阻塞模式
        sChannel.configureBlocking(false);
        ByteBuffer buf = ByteBuffer.allocate(1024);

        buf.put(new Date().toString().getBytes());
        buf.flip();
        sChannel.write(buf);
        buf.clear();

        sChannel.close();
    }



//----------server----------------

    public void server2() throws IOException{
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        //每次得到channel ， 都需要将阻塞关闭
        ssChannel.configureBlocking(false);
        ssChannel.bind(new InetSocketAddress(9999));

        //!! 得到 selector , then 注册 channel 到选择器 
        Selector selector = Selector.open();
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);

        //轮询
        while (selector.select() > 0){ //通过Selector的select（）方法可以选择已经准备就绪的通道 
            Iterator<SelectionKey> it = selector.selectedKeys().iterator();
            while (it.hasNext()){
                SelectionKey sk = it.next();
                if (sk.isAcceptable()){
                    SocketChannel sChannel = ssChannel.accept();
                    //!!
                    sChannel.configureBlocking(false);
                    sChannel.register(selector, SelectionKey.OP_READ);
                }else if (sk.isReadable()){
                    SocketChannel sChannel = (SocketChannel) sk.channel();

                    ByteBuffer buf = ByteBuffer.allocate(1024);
                    int len = 0;
                    while ((len = sChannel.read(buf)) > 0){
                        buf.flip();
                        System.out.println(new String(buf.array(), 0, len));
                        buf.clear();
                    }
                }
                //remove SelectionKey
                it.remove();
            }
        }

    }


```
以上都是同步IO ， 异步IO使用AIO （Asynchronous IO）。

## 六、管道 pipe 

与channel类似 只不过 pipe 是单向的


## 七、Reactor模式

#### 1、简介

Reactor模式(反应器模式)是一种处理一个或多个客户端并发交付服务请求的事件设计模式。当请求抵达后，服务处理程序使用I/O多路复用策略，然后同步地派发这些请求至相关的请求处理程序。

应用 Redis ， 以及上文的 非阻塞式 Demo。

#### 2、优点

1）响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的；

2）编程相对简单，可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销；

3）可扩展性，可以方便的通过增加Reactor实例个数来充分利用CPU资源；

4）可复用性，reactor框架本身与具体事件处理逻辑无关，具有很高的复用性；



----

参考 ： 
视频教程 ： [https://www.bilibili.com/video/BV14W411u7](https://www.bilibili.com/video/BV14W411u7)
selector ：[https://www.cnblogs.com/snailclimb/p/9086334.html](https://www.cnblogs.com/snailclimb/p/9086334.html)

reactor模式 ： [Reactor模式](https://www.cnblogs.com/crazymakercircle/p/9833847.html)
NIO原理 ： [https://www.cnblogs.com/crazymakercircle/p/10225159.html](https://www.cnblogs.com/crazymakercircle/p/10225159.html)

