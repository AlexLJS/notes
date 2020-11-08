---
title: 算法题中的int边界处理
date: 2020-10-01 18:14:05
tags:
categories:
---

听了闫总的第一节课， 对算法题中 Int 边界问题，有了一个认识。



对于一个 int 越界问题， 

我们先关注一下 int 的取值范围，由于 0 的存在，导致这个 int 区间是不对称的，

`[-2^31,2^31 - 1]`

虽然都是 21 亿，但正向的绝对值要少1 ，产生了第一个问题： 

# 1、 Integer.MIN_VALUE 取反， 会导致直接越界


解决方式，对 Integer.MIN_VALUE 进行特判！


# 2、 如何处理一个 布尔表达式的越界问题

解决方式，利用恒等变换。

当左边一坨表达式 需要判定是否越界 Integer.MAX_VALUE， 如果左边已经越界就会报异常。 

a + b > Integer.MAX_VALUE 

则可以通过判定 a > Integer.MAX_VALUE - b，等效判断 a + b 是否越界。



# 实战

LC 7 整数反转

LC 8 字符串转整数


