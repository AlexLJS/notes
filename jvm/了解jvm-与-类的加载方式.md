---
title: 了解jvm 与 类的加载方式
date: 2020-07-07 18:55:14
tags: Jvm
categories: Jvm
---
注： 
本系列jvm笔记分三线记录 ： 正常文字笔记，穿插面试问题，再穿插jvm命令简介。尽量少说废话。

----
## 一、jvm简介

### 1. 开启与停止

jvm指的是 java 虚拟机 ， 每个jvm 启动时就开启一个进程。

>问题1：jvm什么时间退出？
答 ： 程序正常结束；程序异常或者错误；系统崩了；调用 Runtime.exit() 或者 System.exit() ， 或者 Runtime.halt() 。

>命令简介 ： 
jps ： 打印当前程序执行中的进程。
```
public class Test {
    public static void main(String[] args) {
        System.out.println( "hello world");
        try {
            Thread.sleep(6000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

> jps

7090 Jps
7048 Test
2927 Main
7039 Launcher

```
<!-- more -->
### 2. 一段java代码执行过程 ：

jvm先将java代码编译成字节码. class，jvm再经过类的加载器将class文件加载到内存 （即将字节码文件.class解释成二进制文件）。针对不同的操作系统，由不同的jvm调整解释过程。所以，** java是跨平台语言，一次编译到处执行。**

>问题2：为何说java 是个半解释半编译型语言？
答：并不是因为编译成字节码，再解释成二进制文件执行。 而是， 因为jvm的解释过程（这里jvm指的是 Hotspot虚拟机） ，JIT（即时编译器）与解释器协同工作。JIT专门用来将常用的代码，编译成二进制码写入“方法区”的缓存，CodeCache。jit 相当于c语言的编译器，为了提升效率。 所以，java称为半解释半编译语言。

###  3. 虚拟机简介 ：

HotSpot虚拟机 ， 目前是性能最好的虚拟机。在响应时间与执行时间取得平衡，通过本地方法区缓存热点代码来实现的。 但 JRockit 是世界上最快的虚拟机，只专注服务器端， 放弃了响应时间较快的解释器，全部代码编译后执行。此外还有J9 虚拟机， 专门给IBM自己的服务器产品中使用。

还有很多虚拟机根据不同的硬件平台定制的，速度非常快，不是前三个通用jvm虚拟机。例如TaobaoJVM ， 跟Intelcpu深度耦合， 定制了自己的jvm 对gc优化、对heap空间的对象进行多jvm共享……不再赘述

（这里是废话 ， 介绍jvm他想听的是 运行时数据区。）
 
----
## 二、类的加载 Class Loader Subsystem

> 问题3 ：简单介绍一下类的加载过程？ 
答：java中类的加载分为三个阶段： 加载、链接、初始化阶段。加载阶段负责将字节码问价丢进方法区，链接阶段负责校验class文件是否合法， 初始化阶段负责初始化静态方法、静态变量静态代码块。

### 1. 加载阶段 Loading

1）加载 ： 根据类全名通过二进制字节流将类加载到方法区（元数据/永久代，不同版本），再在内存中产生一个  java.lang.Class对象， 作为类的访问入口。此外 ClassLoader 负责 .class 字节码文件 加载到内存（方法区），至于能不能执行要看 Execution Engine 执行引擎 （解释器与编译器）。

>问题4 ：加载.class文件有哪些方式？
答：本地系统；通过网络获取；压缩文件 zip ， （jar ， war）；运行时计算生成，动态代理；其他文件生成，JSP ； 加密文件中获取（防止.class文件被反编译串改）

>问题5 ： 说说方法区、元空间、永久代？
答： 方法区 是 jvm虚拟机规范制定的内存区域的一种规范，永久代是hotspot虚拟机对方法区的一种落地实现。jdk 1.8 以后永久代改为了元空间，主要区别是数据位置由内存放到了硬盘，这部分避免了 OOM。

>命令参数：
-XX:MetaspaceSize=N //设置Metaspace的初始（和最⼩⼤⼩）
-XX:MaxMetaspaceSize=N //设置Metaspace的最⼤⼤⼩


### 2. 链接阶段 Linking

链接阶段又分为，验证、准备、解析。 

**注：加载阶段和链接阶段的部分内容是交叉进⾏的，加载阶段尚未结束，连接阶段可能就已经开始了。**

1）验证
验证class文件是否合法， 例如 ： 字节码文件（二进制流）头是否为 CAFEBABE （咖啡宝贝）这里可以使用软件  binary viewer 查看，例：

![咖啡宝贝](https://upload-images.jianshu.io/upload_images/15578361-731300881c643888.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2）准备

为类变量static分配内存初始值。（注意，这里不是实例变量）不包含 final static变量， 这种常量在 **JIT编译时** 内存已经分配了。

3）解析

引用相关的类，诸如对一些引用的系统类进行**符号引用**。如下 `#2`

>命令： javap  反编译 class 文件
```
alex@ubuntu:~/IdeaProjects/JavaSpider/out/production/JavaSpider$ javap -v Test.class 
Classfile /home/alex/IdeaProjects/JavaSpider/out/production/JavaSpider/Test.class
  Last modified Jun 12, 2020; size 751 bytes
  MD5 checksum 26bb72b8ad9a6d46cea306499361b54c
  Compiled from "Test.java"
public class Test
  minor version: 0
  major version: 58
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #2.#3          // java/lang/Object."<init>":()V
   #2 = Class              #4             // java/lang/Object

```

### 3. 初始化阶段 Initialization

1）执行**类的构造器方法 <clinit>()** ， 只执行一次。

2）<clinit>() 是执行 类变量显示赋值 和 执行静态代码块语句，（简言之就是执行 static 关键字修饰的变量和动作）， 如果没有 类变量和 static{} ，clinit就不会调用。

3） <clinit>() 多线程下自动同步加锁。 

>面试题6 ：分析方法的执行顺序 ，代码见下：
这题非常麻烦， 记住原则 “优先级最高，静态都在非静态前。然后，父类在子类前。static与非static按顺序”。 额外注意的是，子类构造方法隐式调用父类构造方法super， 这里初始化 就是执行 clinit 的过程 ； main方法所在类， 无论main有没有操作都自动初始化。 

```
父类 类变量
父类静态代码块
子类 类变量
子类静态代码块
子类main方法
父类 实例变量
父类非静态代码块
父类构造方法
子类 实例变量
子类非静态代码块
子类构造方法
```

```
package JavaFeatures;

public class InitOrderTest extends Father{
    private int i = initI();
    private static int j = initJ();

    private int initI() {
        System.out.println("子类 实例变量");
        return 0;
    }

    private static int initJ() {
        System.out.println("子类 类变量");
        return 0;
    }

    static {
        System.out.println("子类静态代码块");
    }

    {
        System.out.println("子类 非静态代码块");
    }

    public InitOrderTest(){
        //super();
        System.out.println("子类 构造方法");
    }

    public static void main(String[] args) {
        System.out.println("子类 main方法");
        InitOrderTest init = new InitOrderTest();
    }
}

class Father{
    private int i = initI();
    private static int j = initJ();

    private int initI() {
        System.out.println("父类 实例变量");
        return 0;
    }

    private static int initJ() {
        System.out.println("父类 类变量");
        return 0;
    }

    static {
        System.out.println("父类 静态代码块");
    }
    {
        System.out.println("父类 非静态代码块");
    }

    public Father(){
        System.out.println("父类 构造方法");
    }
}

```
----
## 三、类的加载器分类 ：

>面试题7：知道哪些类加载器？
答： 引导类加载器， 拓展类加载器， 应用程序类加载器。 用途如下：

### 1. 引导类加载器 Bootstrap ClassLoader

用于加载核心类库 ： 用c和c++编写， 没有父类加载器，用于包名java 、 javax 、 sun等开的头的的核心类库。


### 2. 扩展类加载器 Extension ClassLoader 

虚拟机自带的类加载器 ：java编写，继承ClassLoader ， 加载目录 jre/lib/ext 目录下的 class文件。

### 3). 应用程序类加载器 App ClassLoader 

也叫 系统类加载器： java编写，继承ClassLoader ，负责加载 classpath ，一般来说Java的程序的类都是用AppClassLoader 加载的。

注：
 如何获取类加载器？
`ClassName.class.getClassLoader();` 引导类加载器是不能被获取的。

**以上说的是jdk 1.8情况** ， 之后的版本（jdk9以后）考虑安全性，不能将jar 丢进ext 目录进行加载，所以扩展类加载器被废除。取代的是 PlatformClassLoader，如下 ： 

```
public class ClassLoaderTest {
    public static void main(String[] args) {
        System.out.println( ClassLoaderTest.class.getClassLoader());
        System.out.println( ClassLoaderTest.class.getClassLoader().getParent());
        System.out.println( ClassLoaderTest.class.getClassLoader().getParent().getParent());
    }
}
jdk.internal.loader.ClassLoaders$AppClassLoader@1f89ab83
jdk.internal.loader.ClassLoaders$PlatformClassLoader@7c30a502
null
```

何时自定义类加载器？
一般情况下不用自定义类的加载器， 如需要代码加密、隔离加载的时候需要自定义类的加载器。

----

## 四、 双亲委派机制

### 1. 介绍

当类加载器收到加载请求，先判断此类加载过吗 ，加载过直接返回类对象。  否则，先委托给父类加载器加载（向上委托，直到委托到引导类加载器），如果父类加载器可以完成加载任务，就由父类完成。如果，父类加载器不能完成， 再交由子类加载器进行加载。一个类只能由一个加载器加载。有时候存在“接口由引导类加载， 实现类由App加载器加载”。

### 2. 优势

保护程序安全 ，保证核心 Api 不会被篡改。（沙箱安全机制） 即使你自己写了 java.lang.String ，引导类加载器也会从jdk中加载String ， 忽略掉自创的类。

>问题8： 如何判断两个类完全一样？ 
答：完整的包名一致；两个类的类加载是相同的。

### 3. 类的主动引用和被动引用区别？ 
1）主动引用

>1、遇到new,getstatic,putstatic,invokestatic这4条字节码指令时，类如果没初始化就会被初始化，创建对象，读取或设置静态字段，调用静态方法。
2、反射
3、子类初始化前会先初始化父类
4、包含main方法的类，虚拟机启动时会先初始化该类

2）被动引用

除了以上主动引用情况，都是被动引用。 举例， 子类调用父类静态成员变量：
```
class SuperClass{ 
  static{ 
    System.out.println("super class init."); 
  } 
  public static int value=123; 
} 

class SubClass extends SuperClass{ 
  static{ 
    System.out.println("sub class init."); 
  } 
} 

public class test{ 
  public static void main(String[]args){ 
    System.out.println(SubClass.value); 
  } 

} 

```

3）区别 

类初始化的时候，是否调用类的初始化 <clinit>来 初始化静态变量、方法。

----
参考： 

[https://blog.csdn.net/weixin_42636552/java/article/details/82949999](https://blog.csdn.net/weixin_42636552/java/article/details/82949999)
[https://www.bilibili.com/video/BV1Eb411P7bP](https://www.bilibili.com/video/BV1Eb411P7bP)
