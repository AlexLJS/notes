---
title: jvm GC-Roots 与 可达性分析
date: 2020-08-24 11:52:04
tags: Jvm
categories: Jvm
---

什么是垃圾？从 GC-Root 开始没有被引用的对象就是 垃圾 。

一、四种引用

强引用 ： 绝大部分对象都是强引用 ， 例如 ： 赋值操作。

弱引用 ： 方便垃圾回收，一旦发生垃圾回收 GC，弱引用对象会被回收， 例如 ： ThreadLocal 中的 ThreadLocalMap 中 Entry 的 Key 就是对 ThreadLocal 的弱引用。

软引用 ： GC 发生，如果内存还不够，就回收软引用。例如 ： 常用作缓存，毕竟缓存对象可有可无。

虚引用 ： 基本没什么使用

>使用方式： 
弱引用为例，`A a = new A(); WeakReference<A> wa = new WeakReference<A>(new A());`


二、GC-Roots

问题： 什么可以作为 GC-root ？ 

1、所有活的线程对象
2、栈帧中的局部变量表
3、方法的静态成员变量

4、Native 方法持有的引用


三、可达性分析


000 000 000
