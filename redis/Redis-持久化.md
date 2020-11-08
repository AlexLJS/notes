---
title: Redis 持久化
date: 2020-07-09 00:14:45
tags: Redis
categories: Database
---
面试中如果被问到Redis持久化方式，只能给出以下回答：

>“Redis持久化方式有两种，RDB 和 AOF。RDB是系统fork 出一条Redis子进程用来做父进程的快照，复制完毕后形成dump.rdb文件覆盖掉之前的rdb文件。rdb的触发可以通过conf文件进行配置，redis 遇到故障 restart时会加载rdb文件。AOF 是也是fork 一条进程实时追踪Redis命令记录，将更新的命令记录到aof缓存中，并默认每隔一秒追加到Aof文件末尾。恢复记录时按顺序执行一遍aof文件即可。”

对，但深度不够。


----

## 一、RDB 持久化原理

1、原理：

父进程收到bgsave命令后，会调用linux的fork()出一条子进程，所有的备份工作由子进程完成。但是此时父进程与子进程共享同一份内存空间，父进程继续向外提供服务，子进程负责备份。利用COW写入时复制机制，保证子进程备份的数据是bgsave命令下达时刻的快照。（详见下文“Redis面试题”）


2、存在问题：

1) fork() 也会短暂阻塞父进程
2) RDB会造成最后一次数据丢失


## 二、COW 写入时复制机制 copy-on-write

1、基本含义 ： 维基百科

> 写入时复制（英语：Copy-on-write，简称COW）是一种计算机程序设计领域的优化策略。其核心思想是，如果有多个调用者（callers）同时要求相同资源（如内存或磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的（transparently）。此作法主要的优点是如果调用者没有修改该资源，就不会有副本（private copy）被创建，因此多个调用者只是读取操作时可以共享同一份资源。

<!-- more -->
2、Redis 面试问题：进行快照时修改数据，快照是修改后的数据吗？

答：不是。 
当子进程进行备份，父线程尝试修改数据（此时linux内核将数据设置为Read-only），触发COW机制。子进程指向内存空间不变，父进程copy出一份修改页，在copy页中进行修改。子进程不会同步父进程的改动。

3、Java 中COW的实现：

CopyOnWriteArrayList和CopyOnWriteArraySet， 用于读多写少的并发场景，比如白名单，黑名单。


## 三、AOF

吃硬盘空间，占用硬盘IO 。由于Redis 多为缓存用途， 对于并发下的数据的一致性C没有那么高要求，换言之损失点数据没啥关系。还有主从复制备份数据。AOF尽量少用吧！ 

----
参考 ：

[http://ifeve.com/java-copy-on-write/](http://ifeve.com/java-copy-on-write/)

[https://blog.csdn.net/ctwctw/article/details/105147277](https://blog.csdn.net/ctwctw/article/details/105147277)

