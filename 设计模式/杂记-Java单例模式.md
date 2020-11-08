---
title: 杂记 Java单例模式
date: 2020-07-05 01:30:26
tags: Java基础
categories: Java
---
单例模式基本上是必考面试题，也是面试中引出多线程、JVM问题的万恶之源！

网上大概有五六七八种的实现方式，为了减轻记忆负担，只举三个最典型的。

----

## 一、什么是单例模式？

某个类在一个系统中只有一个实例对象。

如何保证这点呢？

首先想到的是，不能让其他用户调用构造方法；检测到实例已经存在，就返回已经存在的实例。
举例： JVM 的 Runtime类 ， 获取方式 `Runtime.getRuntime();`。

----
## 二、单例模式要点


1. 构造函数私有化

2. 接收实例静态化
（因为实例instance 只能有一份，所以实例应该属于class， 用于接收的对象必须静态化static）

3. 向外提供获取实例的接口
（饿汉，instance 公有化 class 直接调用。 懒汉 ， 暴露getInstance() 方法 ）

<!-- more -->
----
## 三、饿汉式 ： 最简单的单例

饿汉式： 饿汉很急，不管实例变量是否被使用都创建出来。因为进程开始就创建出来，所以 **不存在线程安全问题，但浪费资源**。

1. 枚举类
```
enum Singleton2 {
    INSTANCE;
}
```
枚举类枚举常量列表的个数为1 ，即为单例。其操作等价于：
```
class Singleton2 {
    public static final Singleton2 INSTANCE = new Singleton2();
    private Singleton2(){};
}
```

2. 静态代码块

```
class Singleton3{
    public final static Singleton3 INSTANCE;
    static {
        // 此处可以通过类加载器，来配置外部配置文件的参数输入。
        INSTANCE = new Singleton3();
    }
    private Singleton3(){};
}
```
静态代码块似乎跟 枚举类没什么区别！！ 
唯一区别 ： 静态代码块可以用于接收外部文件的参数。（通过类加载器）

总结 ： 由于恶汉是最先随程序创建的，所以利用static将单例写成常量即可。

----
## 四、懒汉式：双锁DCL

懒汉式 ： 懒汉很懒，需要时才创建对象。需要对象的时机与创建对象的耗时，**可能会引起线程安全问题**！

模拟过程：
Thread1 -> 创建对象 ing， 此时 instance 为未实例化；
这个过程中 Thread2 -> 获取对象 ，instance没创建完成， 那么 t2 也会开始创建instance。单例失败！

解决方式 ： 加锁！
 
```
public class Singleton {
    /**
     * 单例模式
     * 懒汉式 ： 双锁DCL
     */
    private volatile static Singleton instance;//静态化、私有化
    private Singleton(){}

    public static Singleton getInstance() {
        if (instance == null){//此步骤可有可无，为了优化性能，不要调用get就抢锁
            synchronized (Singleton.class){
                if (instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

----
## 五、拓展面试问题

1、volatile 不写行不行 ？

答 ： 不行。 volatile防止指令重排， 其他线程立刻获取instance的更新值。

>Java语言规范中指出：为了获得最佳速度，允许线程保存共享成员变量的私有拷贝，而且只当线程进入或者离开同步代码块时才与共享成员变量的原始值对比。

t1改过instance （共享成员变量），t2却copy了一份没改的在自己的寄存器中。 volatile 告诉JVM 这instance不稳定，Thread不要从寄存器读数据，每次从主内存中读。 


2、有没有别的方法写单例？

答：静态内部类。因为静态内部类不是 “真·静态”， 而是在使用时才加载和初始化。（其实就是一未初始化的静态成员变量）

```
public class Singleton {
    private Singleton(){}

    private static class Inner{
        private static final Singleton INSTANCE = new Singleton();
    }

    public Singleton getInstance(){
        return Inner.INSTANCE;
    }
}

```
其实我不太懂静态内部类内层引用外层，感觉像是控制了加载时间的静态代码块。 


