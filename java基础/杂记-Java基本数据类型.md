---
title: 杂记 Java基本数据类型
date: 2020-07-16 00:20:51
tags: Java基础
categories: Java
---

我的第一次面试的第一道题就是“java有哪些基本数据类型”。现在回想，哪怕基本数据类型都有许多值得深挖的地方。特此分享，与大家共勉。

## 一、基础

![基本数据类型.jpg](https://upload-images.jianshu.io/upload_images/15578361-d2553d3cadbfe3e7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1、**初始值：**
如图所示，java 中有八种基本数据类型，我注明了他们未被初始化的默认值。对于类的成员变量，未初始化进行调用，会赋默认值，在解决某些leetcode问题时需要注意这点。

<!-- more -->
2、**范围：**
因为有0的存在，取值范围都是左闭右开，例如：byte类型整数占1字节占8bit，范围[-128~128)，导致这样`byte b = 128;`的语句是错误的。

3、**整数存储：**
java默认的整数类型是int， 也就是说随便写一个数字，按int 存储。但是，如果该数字在byte、short范围内，被赋值给byte、short变量时，是不按int 存贮。如果超出了int 范围，是需要强转或者+L。
看似无关紧要，但下面这段代码会报错：
```
        short s = 4;
        s = s - 2;
```
2按int处理 ， s-2 会被自动升级为int！这是**表达式的自动提升**。

4、**什么是char：**
char 是字符变量，但可以进行加减乘除，甚至比较大小！因为系统按char 的字符编码进行处理，即char完全可以当成int处理。同理，也可以将0~65535整数赋值给int变量。
```
        char a = '李';
        int b = a;
        System.out.println(a+b);
        System.out.println(String.valueOf(a) + b);

Out:
52892
李26446
```

5、**浮点型不精确：**
大家都知道浮点数float、double不精确，double精确度更高。
```
        float f = 5.2345556f;
        System.out.println(f);
Out:
5.2345557
```
都知道要用精确的浮点数要用BigDecimal类。但，为什么float、double不精确？因为底层数据使用了二进制科学计数法。float：1位符号，8位指数，23位尾数。double：1位符号，11位尾数，52位尾数。我也不懂，java的默认浮点数是double，转成float需要+F。`float f = 2.5`错误语句，2.5默认double。

## 二、进阶
我们接着浮点数说……
### 1、java数字能除以0吗？
整数不行，浮点数可以。
像数学运算一样，正浮点数除以0得到无穷大，负数除以0得到负无穷大，如果0/0得到NaN（非数）
```
        float f = 5.2345556f;

        System.out.println(f/0);
        System.out.println(-f/0);
        System.out.println(0.0/0);

        System.out.println(f/0 == Float.POSITIVE_INFINITY);
        System.out.println(Float.NaN == Float.NaN);
Out：
Infinity
-Infinity
NaN
true
false
```
所有的无穷大都相等，所有的NaN都不相等。
### 2、强制类型转换
语法规则简单，但有几点需要注意：
byte、short、char 参与运算，jvm会直接将变量值强转为int，最后得到的也是int。
```
        short s1 = 1;s1=s1+1;//编译出错
        short s2 = 1;s2+=1;
```
有趣的是，“+=”是特殊运算符，就不会报错。
“小转大”一般不会出现问题，“大转小”多半会出现意外。发生，丢失数据，正负号改变的问题。是因为，计算机存储整数是用补码存储。补码正数不变，负数留下符号位，取反加一。第一位是符号位，大转小容易使得第一位为1 。
```
        int a = 646;
        byte b = (byte)a;
        System.out.println(b);
Out:
-122
```
强转不复杂，自动转要比强转有更多说法，像是：
字符串的true false 不能转boolean， 但是boolean可以自动转为String。
```
        boolean a = true;
        System.out.println(a + "abc");
Out:
trueabc
```
补充：
java中数字是可以用断开的，看起来清楚
`int c = 89_7_57;System.out.println(c);`