---
title: java基础 锁1.0
date: 2020-08-18 19:50:37
tags: Java基础
categories: Java
---


复习一下线程安全问题。本篇简介基础常识，源码见下一篇。

----
## 一、基础概念

#### 1.1 乐观锁与悲观锁
乐观锁：对使用数据过程不积极采取加锁措施，仅在更新的时候判断数据是否被其他线程修改。实现方式，java中使用无锁编程即使用乐观锁，参考java.util.cocurrent.AtomicInteger原子类。

悲观锁：认为数据使用时一定会有其他线程来修改，需要先加锁，且在使用过程中确保数据不会被修改。实现方式，如sychronized 和 Lock都是悲观锁。


#### 1.2 公平锁与非公平锁
公平锁：按照队列顺序排队获取锁。

非公平锁：后来的先去竞争锁，恰巧资源释放就直接获取锁， 如果被占用则去队列尾排队。

#### 1.3 可重入锁
可重入锁（递归锁）：线程外层方法获取锁后，内层方法自动获取锁，synchronized 和 ReentranLock 都是可重入锁。

非可重入锁：容易形成死锁。

<!-- more -->
还有独享锁 共享锁等多种锁。

----
## 二、双锁单例模式

```

public class Singleton {
    /**
     * 双锁单例模式
     */
    private volatile static Singleton instance = null;

    private Singleton() {

    }

    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}

```

----

## 三、基础知识

#### 1、CAS自旋算法：Compare And Swap
比较与交换是一个原子操作 ，简言之，即从内存中读取数据值v  赋值给a 进行操作，如果想用b 替换a的值，那么a必须要与 内存中值v相同。如果中途其他线程修改了内存v，那么这个原子操作不断重试，即自旋。

源码 ： 
```
    public final boolean compareAndSet(int expectedValue, int newValue) {
        return U.compareAndSetInt(this, VALUE, expectedValue, newValue);
    }

```
使用 ：

```
        AtomicInteger ai = new AtomicInteger(1);

        ai.compareAndSet(1,5); // return boolean 修改是否成功

        System.out.println( ai.get());
```

#### 2、并发编程三大概念：

2.1 原子性：一个或多个操作捆绑执行，同时成功or同时失败。

2.2 可见性：线程对一个变量进行修改，其他线程是否能立刻发现。

2.3 有序性：语句顺序执行，显然jvm会对语句进行重排序。

#### 3、缓存一致性：

cpu运算速度比内存、硬盘等快很多，程序在内存中运行无法保证与cpu进行实时交互， 所以cpu设置了缓存。内存中程序需要复制一份到缓存中，cpu执行完指令，更新缓存数据，再将缓存数据写入内存。

在并发编程时，由于内存区域只有一片，但不同线程拥有自己的缓存，容易产生缓存不一致问题！解决办法，缓存一致性协议，或者总线锁。

#### 4、Java对象头 ： 
标记字段 + 类型指针
其中标记字段存放了GC分代、hashcode、锁的标志位等信息。

锁的标志位四种状态 ： 无锁、偏向锁、轻量级锁、重量级锁。

其中偏向锁对象被一个线程访问，该线程可以自动获取锁，减少性能开销。轻量锁，在偏向锁对象被其他线程访问时候，升级为轻量锁，其他线程进入自旋，防止阻塞。（因为阻塞和挂起对于cpu的消耗很高，如果同步逻辑很短， 建议其他进如CAS自旋，所以引入了偏向锁和轻量锁）


#### 5、内部锁Monitor ： 

Monitor提供一种同步机制，线程所有。线程对对象上锁，对象关联的monitor进入线程的monitor recoder。Monitor底层依赖一种互斥锁， sychronized 同步代码使用Monitor实现同步。所以同步过程中，其他线程处于阻塞状态。

#### 6、死锁：

线程争夺同一资源产生僵局，在无外力介入情况下，线程无法向前推进。
例如， T1 先锁a 同步代码块内部再锁b ， 而T2 先锁b 同步代码块内部再锁再锁a， T1得到了a 在请求b时阻塞了， 而T2 拿着b在请求a时也阻塞了，形成了死锁。
```
//t1
...
synchronized(a){
    synchronized(b){
        do sth
    }
}
...
//t2
...
synchronized(b){
    synchronized(a){
        do sth
    }
}
...
```

synchronized 关键字同步方法时，容易形成死锁。（api源码中的同步方法，会产生隐蔽的死锁）

解决方式 ： 
或者人为调整加锁顺序。
synchronized显示锁没有超时放弃，建议使用Lock并设置超时放弃tryLock(long timeout ,timeUnit unit)。 


#### 7、volatile 关键字：

1） volatile 可以保证可见性 ： 被volatile修饰的变量，可以保证在更新完后直接更新主存数据。
2）volatile 可以影响jvm的顺序 ： 尽管 jvm无序 ， 但jvm会保证 在 volatile 之前的指令，先于 volatile之后的指令执行。 （禁止指令重排）
3）volatile 不能保证原子性。
4）实现原理：在内存加锁。


----
## 四、参考
[https://www.cnblogs.com/jyroy/p/11365935.html](https://www.cnblogs.com/jyroy/p/11365935.html)
[https://www.cnblogs.com/dolphin0520/p/3920373.html](https://www.cnblogs.com/dolphin0520/p/3920373.html)
