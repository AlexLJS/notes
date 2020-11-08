---
title: 杂记 Java正则表达式
date: 2020-07-27 12:33:18
tags: Java基础
categories: Java
---

剑指Offer中有一道题 ：表示数值的字符串 

>题目描述
请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。

第一感觉就是正则，因为写if判断太繁琐了。

`^[+-]?([0-9]+(\.[0-9]*)?|\.[0-9]+)([eE][+-]?[0-9]+)?$`

这条reg 涵盖了日常开发的常用写法， 就以这条为例，复习一下Java中的正则表达式。
（注意java中 \ 需要 \\\\ 双倍）

 ----
## 一、 使用方式

正则表达式也是个String ，下文用 regex 表示正则的Pattern String。

 #### 1. String 字符串相关

matches(regex) 匹配，返回boolean

split(regex) 切割，返回String[]

replaceAll(regex,String) 替换，返回String

<!-- more -->
 #### 2. Pattern 类

`import java.util.regex.Pattern;`

Pattern.matches(regex) 同 String.matches

Pattern.compile(regex) 编译正则表达式，返回 Pattern 对象， 实际使用中需要搭配 Matcher

` Matcher m = p.matcher(str);`

 #### 3. Matcher

常用 find() 和 group()

```

Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("22bb23"); 
m.find();//返回true 
Matcher m2=p.matcher("aa2223"); 
m2.find();//返回true 
Matcher m3=p.matcher("aa2223bb"); 
m3.find();//返回true 
Matcher m4=p.matcher("aabb"); 
m4.find();//返回false
//---------------------
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("我的QQ是:456456 我的电话是:0532214 我的邮箱是:aaa123@aaa.com"); 
while(m.find()) { 
     System.out.println(m.group()); 
} 
//-----------------------
456456 
0532214 
123 
```


----

## 二、 正则规则
正则规则建议自己百度。
[https://www.runoob.com/java/java-regular-expressions.html](https://www.runoob.com/java/java-regular-expressions.html)

这里举例开头提到的字符串：
```
^[+-]?([0-9]+(\\.[0-9]*)?|\\.[0-9]+)([eE][+-]?[0-9]+)?$
```

1. ^ 正则匹配的开始

2. $ 正则匹配的结束

3. [] 匹配一个字符

4. . 可以匹配任何字符

5.  \*  表示 0个 或 多个

6. ? 表示 0个 或 1个

7. () 捕获组：表示子正则表达式，看作一个整体

8. {} 用来限制个数

9. ＋表示 １个 或 多个

也可以写逻辑判断， 就不再赘述。正则表达式很强大，例如，上题判断科学第一个因数的合法性：
```
([0-9]+(\\.[0-9]*)?|\\.[0-9]+)
```

表示： 有整数部分的话，小数部分可以有可以没有，但只能存在一组 ； 没整数部分的话，小数部分至少有一个数字。
写if 判断会非常复杂， regex写起来也很困难需要在线测试工具。

----
## 三、常用正则

```
Email地址：^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$

域名：[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+\.?

InternetURL：[a-zA-z]+://[^\s]* 或 ^http://([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?$

手机号码：^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$

```

详见工具推荐网址，具体业务酌情修改。

----
## 四、工具推荐


[https://c.runoob.com/front-end/854](https://c.runoob.com/front-end/854)




