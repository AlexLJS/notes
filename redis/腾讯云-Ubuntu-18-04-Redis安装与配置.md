---
title: 腾讯云 Ubuntu 18.04 Redis安装与配置
date: 2020-07-07 17:39:22
tags: Redis
categories: Database
---

参考教程 ：[https://cloud.tencent.com/developer/article/1163860](https://cloud.tencent.com/developer/article/1163860)

官方网站：[http://www.redis.cn/](http://www.redis.cn/)

## 一 、 过程中遇到的坑

虽然初学Redis，总的来说 Redis 安装配置基本没坑，可以不用docker。

1. redis配置文件（`/etc/redis/redis.conf `）有一千多行 ，我吐了！定位需要的位置：

1） `grep -rn "dbfilename"  /etc/redis/redis.conf ` 定位 dbfilename 在哪行。
2） vim  ： `:n` 跳到第n行

或者参考conf ： [http://download.redis.io/redis-stable/redis.conf](http://download.redis.io/redis-stable/redis.conf)

2. 可以顺利完成安装，启动服务，记下常用文件位置，方便后续调整使用。 

配置文件位置： /etc/redis/redis.conf

dump的rdb文件位置： /var/lib/redis

redis性能测试 ： /usr/local/bin/redis-benchmark


3. 第二台服务器 make test ，第53项报错，错误信息如下：

<!-- more -->
```
[err]: diskless no replicas drop during rdb pipe in tests/integration/replication.tcl
rdb child didn't terminate
```
查看 tests/integration/replication.tcl ，报错位置：
```
# test diskless rdb pipe with multiple replicas, which may drop half way
start_server {tags {"repl"}} {
    set master [srv 0 client]
    $master config set repl-diskless-sync yes
    $master config set repl-diskless-sync-delay 1
    set master_host [srv 0 host]
    set master_port [srv 0 port]
    set master_pid [srv 0 pid]
...
 # wait for rdb child to exit
                    wait_for_condition 500 100 {
                        [s -2 rdb_bgsave_in_progress] == 0
                    } else {
                        fail "rdb child didn't terminate"
                    }
...
```
本人才疏学浅，查了好久也没解决这个问题。 **发现跳过这个问题，后续安装可以继续执行，不受影响。**

## 二、配置常识


1. systemd 是什么 ？  

>（百度百科 ） systemd即为system daemon,是linux下的一种init软件,由Lennart Poettering带头开发,并在LGPL 2.1及其后续版本许可证下开源发布,开发目标是提供更优秀的框架以表示系统服务间的依赖关系，并依此实现系统初始化时服务的并行启动，同时达到降低Shell的系统开销的效果，最终代替常用的System V与BSD风格init程序。

配置systemd ， 就不用配置守护线程来保证Redis在后台实时拉起。

>当redis.conf中选项daemonize设置成yes时，代表开启守护进程模式。在该模式下，redis会在后台运行，并将进程pid号写入至redis.conf选项pidfile设置的文件中，此时redis将一直运行，除非手动kill该进程。


2. 为何给redis 权限组？如何多个redis服务？ 

出于安全考虑。  
systemd 配置文件位置 /etc/systemd/system/redis.service 。
添加 service服务，并为多个redis服务配置不同的config 文件。


## 三、redis 常识

1. redis的线程模型原简介  ： (问题 ： 介绍一下 Redis 单线程模型 ？)

答 ： redis 的 文件时间处理器 （file event handler）包含四个部分 ： 
1）多个套接字
2）IO多路复用程序
3）文件事件分派器
4）事件处理器 （单线程）

文件事件处理器是单线程的 ， 所以Redis称为单线程模型。（或者说是文件事件分派器的消费是单线程的）

注 ： redis处理请求流程 ：
![文件事件处理器.png](https://upload-images.jianshu.io/upload_images/15578361-f21d8bb3d7bea688.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>消息处理流程 ： 文件事件处理器使用I/O多路复用(multiplexing)程序来同时监听多个套接字，并根据套接字目前执行的任务来为套接字关联不同的事件处理器。当被监听的套接字准备好执行连接应答(accept)、读取(read)、写入(write)、关闭(close)等操作时，与操作相对应的文件事件就会产生，这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。[参考](https://www.jianshu.com/p/6264fa82ac33)

单线程模型大差不差都是维护一个事件queue。参考Nodejs 单线程异步。


2. 常见面试题 ： 为何单线程模型的 Redis 速度快？　

答 ： 
IO多路复用技术，
在内存中完成操作，省去大部分与硬盘的IO消耗。
单线程，省去多线程的竞争资源消耗。 


3. epoll IO多路复用

不太理解 [https://blog.csdn.net/wxy941011/article/details/80274233](https://blog.csdn.net/wxy941011/article/details/80274233)

