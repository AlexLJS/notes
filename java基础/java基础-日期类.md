---
title: java基础 日期类
date: 2020-08-17 22:33:23
tags: Java基础
categories: Java
---
## 综述

 Date类是从JDK1.0开始存在的类，总之这个类有许多的问题，开发的过程中尽量不要使用。例如，它无法实现国际化，对于月份和日期有不同的偏移量。后期，改进了Date类为Calendar类，JDK1.8以后借鉴了Joda-Time新增了time包。

作为web开发中不可避免的时间组件，本文依次对这些做简要的介绍。

****
## Java中过时的的Date类

包：java.util.Date

##### 计时原则：
从一个时间原点来计算到一个时刻的毫秒值（时间原点：1970年1月1日0点），原因：时间格式不能够计算，但是每个时间对应唯一的毫秒值，这个数可以用来计算时间。计算机系统底层都是以毫秒值计算。System.currentTimeMillis() 返回long型参数：
```
//获取当前时间的毫秒值
System.out.println(System.currentTimeMillis());
out:
1553578428677
```

<!-- more -->

##### 构造方法

六个构造方法过时4个，2个没过时的构造方法：

1、`Date date1 = new Date();`
获取当前操作系统的时间与日期。

2、`Date date2 =new Date(0);`
传递毫秒值，对应相对于时间原点的一个日期。

注：输出时间格式，重写过toString()，输出：
`Tue Mar 26 13:39:50 CST 2019`
`Thu Jan 01 08:00:00 CST 1970`

##### 常用方法
getTime() :返回long型毫秒值，日期转为毫秒值返回	
setTime() :将毫秒值转为日期对象，传入毫秒值

##### dateFormat ：
如何实现String 和 Date 对象的互相转化

实现日期对象转字符串：
使用SimpleDateFormat`new SimpleDateFormat(“yyyy MM dd HH mm ss”) `年月日 时分秒

字符串转成日期对象：
Date parse(String s) 	
注意：
可能会有异常  ， throws ，格式需要相同。
```
        Date date = new Date();
        System.out.println(date);
        System.out.println(date.getTime());

        date.setTime(System.currentTimeMillis() + 1000);
        System.out.println(date);
        //格式化输出
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        System.out.println(sdf.format(date));
        //格式化创建date对象
        date = sdf.parse("1995-01-07");
        System.out.println(date);
out:
Tue Mar 26 13:55:30 CST 2019
1553579730298
Tue Mar 26 13:55:31 CST 2019
2019-03-26
Sat Jan 07 00:00:00 CST 1995
```
**总之，不要使用了！**
****
## 改进的Date类：Calendar
包：java.util.Calendar
抽象类不能建对象，子类实现
Calendar c = Calendar.getInstance();
System.out.println(c);

##### 常用方法
```
        Calendar c = Calendar.getInstance();
        System.out.println(c);
        System.out.println(c.get(Calendar.YEAR));
        System.out.println(c.get(Calendar.MONTH));
        System.out.println(c.get(Calendar.DATE));
        c.set(Calendar.YEAR,1995);
        c.set(Calendar.MONTH,1);
        c.set(Calendar.DATE,7);
        System.out.println(c.getTime());//得到Date对象
        c.add(Calendar.DATE,40);
        c.roll(Calendar.MONTH,3);
        System.out.println(c.getTime());
        c.set(2018,10,25);
        System.out.println(c.getTime());

out:
java.util.GregorianCalendar[time=1553582945821,areFieldsSet=true,areAllFieldsSet=true,
lenient=true,zone=sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,
dstSavings=0,useDaylight=false,transitions=19,lastRule=null],firstDayOfWeek=1,
minimalDaysInFirstWeek=1,ERA=1,YEAR=2019,MONTH=2,WEEK_OF_YEAR=13,
WEEK_OF_MONTH=5,DAY_OF_MONTH=26,DAY_OF_YEAR=85,DAY_OF_WEEK=3,
DAY_OF_WEEK_IN_MONTH=4,AM_PM=1,HOUR=2,HOUR_OF_DAY=14,MINUTE=49,
SECOND=5,MILLISECOND=821,ZONE_OFFSET=28800000,DST_OFFSET=0]
2019
2
26
Tue Feb 07 14:49:05 CST 1995
Mon Jun 19 14:49:05 CST 1995
Sun Nov 25 14:49:05 CST 2018
```

##### 注意事项
1、int field 参数是 Calendar类的静态成员变量，可以被调用
例如，Calendar.YEAR代表年时间字段。
2、add 和 roll ：
add 方法增加偏移量超过范围后，会向上一个时间字进位。roll不会进位。
3、容错性：
设置月份为13会不会报错？不会。
如果Calendar对象执行 ， `setLenient(false)`，关闭容错性，则会运行异常。
4、set()方法不会马上修改日期：
set的日期会在执行，get() getTime() add() roll()...方法后才会修改Calendar日期。

至于JDK1.8 后的time 包， 和 常用的joda-time 以后再做笔记吧。






