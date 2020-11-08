---
title: jvm 字节码
date: 2020-08-23 08:39:30
tags:
categories:
---

前提： 使用命令 javap 对 class 文件进行反编译。


## 一、字节码





## 二、浅析字节码执行流程

### 1、自动装（拆）箱

自动装箱功能其实是 java 的语法糖（底层 jvm 偷偷摸摸执行了其他指令， 只是语法显示比较简单）， Integer.valueOf() Integer.intValue()， 写一段代码验证。
```
    public static void main(String[] args) {
        Integer a = 123;
        int b = a;
        System.out.println( a + b );
    }
```

反编译

```
public class Bytecode.AutoPack {
  public Bytecode.AutoPack();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: bipush        123
       2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1
       6: aload_1
       7: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
      10: istore_2
      11: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
      14: aload_1
      15: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
      18: iload_2
      19: iadd
      20: invokevirtual #5                  // Method java/io/PrintStream.println:(I)V
      23: return
}

```

### 2、分支


javap

```
Compiled from "AutoPack.java"
public class Bytecode.AutoPack {
  public Bytecode.AutoPack();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_1
       1: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       4: astore_1
       5: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       8: new           #4                  // class Bytecode/AutoPack
      11: dup
      12: invokespecial #5                  // Method "<init>":()V
      15: aload_1
      16: invokevirtual #6                  // Method java/lang/Integer.intValue:()I
      19: invokevirtual #7                  // Method judge:(I)Ljava/lang/String;
      22: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      25: return

  public java.lang.String judge(int);
    Code:
       0: iload_1
       1: iconst_1
       2: if_icmpne     8
       5: ldc           #9                  // String 输入为 1
       7: areturn
       8: ldc           #10                 // String 非1
      10: areturn
}


```

### 3、循环

### 4、调用


### 5、多态
