---
title: java基础 锁2.0
date: 2020-08-18 19:52:23
tags: Java基础
categories: Java
---

最近，看了一些有关 Synchronized 和 ReentrantLock 的博客和视频， 对Java 的锁机制有了全新的认识。写篇文章记录一下。 仿照 ReentrantLock 手写一个可重入 Lock 和 并且简单分析一下 ReentrantLock 和 AQS 的源码。 

## 一、必备常识

手写一个 可重入锁 ReentrantLock， 需要一些前置知识。

### 1、synchronized 和 ReentrantLock 的恩怨情仇

众所周知， 远古时代的 synchronized 性能较差。 在 jdk 1.5 时候 Doug Lea 看不惯，手撸了并发包。（姑且这么理解）

**你有没有想过 synchronized 为什么性能差？**

这要从 线程模型 说起…… Java 中线程模型采用的是与操作系统“一对一”的线程模型， 即，一个 Thread 线程就对应操作系统的一条线程。我们把操作系统线程叫做 *内核线程* ， Java 线程叫做 *用户线程* 。 为何 “一对一” 也要做这种区分OS 与 用户 的区分？这是出于系统安全性的考虑，毕竟对CPU的操作非常危险。 反应到 Java 上 就是 native 方法 。（native 方法是由 C++ 、C 写的代码，直接操作系统底层。简言之，就是陷入内核操作。由于这方法和 jdk 提供的方法很不同， 所以 jvm 内存划分会有 本地方法栈 ， 就是一个专门用来存 native 方法的 table）

通过反编译 javap 可知 synchronized 的底层是加了同步监视器 moniter， 而且我们是看不到 synchronized 的源码的，说明其是一个 native 底层代码。这就是我们为什么说优化之前的 synchronized 是重锁。因为，一个 Thread 在 jvm 层面做着做着事突然遇到 synchronized ， 必须得切换到内核状态了。即，CPU切换状态，陷入操作系统内核耗时非长久。所以在有多条线程的并发情况下，性能非常差。

所以，我们的 ReentrantLock 横空出世，在 JVM 的层面上 CAS 自旋节省了**部分**陷入os时间，但消耗了CPU资源。

>为什么说这里是“部分”？
如 ， t1 持有 lock， t2、t3来了， t2 尝试获得资源、发生竞争。首次还是要发生争抢。

题外话：在 JMM （java 内存模型）层面上 ，moniterenter 和 moniterexit 是对于 lock 和 unlock 的封装。所以 synchronized 加锁底层也就是通过修改了对象头的 MarkWord ， 线程 lock 和 unlock 了主内存的资源。如果从linux 层面考虑 synchronized 利用 内核的mutex线程互斥锁。

不过经过多代优化过后，synchronized 现在性能已经可以了， 毕竟是亲儿子终将实现反杀！从ConcurrentHashMap 就能略见一斑。


### 2、ReentrantLock 什么原理

或者说， 自己实现一个 Lock 需要些什么？

锁的原理，用大白话说 ： 需要有一个让所有线程都能看见、立即生效的变量（可见性），用来表示这把锁是否被某线程占有了！其他线程来了， 一旦发现被锁被占了，后面排队！（需要一个队列）

啊，你可能有疑问： 
为什么用队列，怎么不CAS？费CPU资源
为什么用队列，怎么不阻塞线程？那不陷入内核…

#### 注意 ： 并发不一定发生资源竞争


最后，分析一下，实现一把锁需要什么：
volatile 修饰的状态变量；一个等位置的队列（双向链表），必须的是支持并发同步队列，这队列里存什么呢，得是线程对象！

没了！就这么简单。

（后期 ， 此处的队列就是 AQS 抽象队列同步器）

开整！


## 二、手撸一把锁

```
package JavaFeature;


import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.CountDownLatch;
import java.util.stream.IntStream;

public class MyLock {

    private static class Node {
        // 线程
        Thread thread;
        // 前一个节点
        Node prev;
        // 后一个节点
        Node next;

        public Node() {
        }

        public Node(Thread thread, Node prev) {
            this.thread = thread;
            this.prev = prev;
        }
    }

    static final Node EMPTY = new Node();

    // 标识是否被锁定
    private volatile int state;
    // 链表头
    private volatile Node head;
    // 链表尾
    private volatile Node tail;

    // state的偏移量
    private static long stateOffset;
    // tail的偏移量
    private static long tailOffset;
    // Unsafe的实例
    private static Unsafe unsafe;

    static {
        try {
            // 获取Unsafe的实例
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
            // 获取state的偏移量
            stateOffset = unsafe.objectFieldOffset(MyLock.class.getDeclaredField("state"));
            // 获取tail的偏移量
            tailOffset = unsafe.objectFieldOffset(MyLock.class.getDeclaredField("tail"));
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    // 原子更新state字段
    private boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }

    // 原子更新tail字段
    private boolean compareAndSetTail(Node expect, Node update) {
        return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
    }

    // 构造方法
    public MyLock() {
        head = tail = EMPTY;
    }

    // 加锁
    public void lock() {
        // 尝试更新state字段，更新成功说明占有了锁
        if (compareAndSetState(0, 1)) {
            return;
        }
        // 未更新成功入队
        Node node = enqueue();
        Node prev = node.prev;
        // 再次尝试获取锁，需要检测上一个节点是不是head，按入队顺序加锁
        while (node.prev != head || !compareAndSetState(0, 1)) {
            // 未获取到锁，阻塞
            unsafe.park(false, 0L);
        }
        // 下面不需要原子更新，因为同时只有一个线程访问到这里
        // 获取到锁了且上一个节点是head
        // head后移一位
        head = node;
        // 清空当前节点的内容，协助GC
        node.thread = null;
        // 将上一个节点从链表中剔除，协助GC
        node.prev = null;
        prev.next = null;
    }

    // 入队
    private Node enqueue() {
        while (true) {
            // 获取尾节点
            Node t = tail;
            // 构造新节点
            Node node = new Node(Thread.currentThread(), t);
            // 不断尝试原子更新尾节点
            if (compareAndSetTail(t, node)) {
                // 更新尾节点成功了，让原尾节点的next指针指向当前节点
                t.next = node;
                return node;
            }
        }
    }

    // 解锁
    public void unlock() {
        // 把state更新成0，这里不需要原子更新，因为同时只有一个线程访问到这里
        state = 0;
        // 下一个待唤醒的节点
        Node next = head.next;
        // 下一个节点不为空，就唤醒它
        if (next != null) {
            unsafe.unpark(next.thread);
        }
    }

    private static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        MyLock lock = new MyLock();

        CountDownLatch countDownLatch = new CountDownLatch(1000);

        IntStream.range(0, 1000).forEach(i -> new Thread(() -> {
            lock.lock();

            try {
                IntStream.range(0, 10000).forEach(j -> {
                    count++;
                });
            } finally {
                lock.unlock();
            }
           System.out.println(Thread.currentThread().getName());
            countDownLatch.countDown();
        }, "tt-" + i).start());

        countDownLatch.await(); // 控制 1000 线程跑完了， 再跑主线程

        System.out.println(count);
    }
}

```

