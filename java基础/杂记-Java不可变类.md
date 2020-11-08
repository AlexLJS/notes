---
title: 杂记 Java不可变类
date: 2020-07-04 15:33:25
tags: Java基础
categories: Java
---

## 一、什么是不可变类 ? 
简言之，immutable， 创建完实例对象后，实例对象不可被改变。最典型的例子是String类。
```
String str = new String("hello");
str = "world";
System.out.println(str);
Out：world
```

看似这个对象被修改了，但实际上是在内存中开辟了新空间。

> 注：在哪部分内存开辟了新空间？
答 ： Heap。学习String的时候，我们肯定被介绍过 **常量池**，知道String是个在底层存储是个final修饰的char[]。(jdk 11 byte[] )  String被放在常量池中，常量池又分了多种， 运行时常量池、静态常量池、字符串常量池…如果新的字符串与之相同，新实例对象指向常量池地址。常量池地址应该在永久代（后期改为元空间MetaSpace） ，1.8以后元空间被放在了本地内存，而字符串常量池被放进了Heap堆空间。  

<!-- more -->

```
    String str1 = new String("hello");
    String str2 = "hello";
    String str3 = "hello";
    System.out.println(str1 == str2);
    System.out.println(str2 == str3);
    System.out.println(System.identityHashCode(str1));
    System.out.println(System.identityHashCode(str2));
    System.out.println(System.identityHashCode(str3));

    Out：
    false
    true
    28014437
    19451386
    19451386
```
通过这段代码可以发现，使用new关键字创建的String并没有进入常量池，是储存在堆内存空间中（非常量池）。

----

## 二、创建一个不可变类

其实，我们可以自定义一个像String一样的不可变类。
创建原则：

>1、类的成员变量， final private修饰
2、构造器带参数，初始化成员变量
3、只提供getter方法，不提供setter方法
4、关于hashCode() 和equals() ： 有必要的话进行重写。

存在一个bug ， 如果一个**不可变类中的某个成员变量是可变类的实例**，直接引用的话，就变成了可变类。如下：

```
public class Test {
    public static void main(String[] args) {
        Job job_1 = new Job("Student");
        Person person_1 = new Person("Alex",job_1);
        System.out.println(person_1);

        job_1.setJobName("Beggar");
        System.out.println(person_1);
    }
}
class Job{
    //可变
    private String jobName;
    public Job(){}
    public Job(String jobName){
        this.jobName = jobName;
    }

    public String getJobName() {
        return jobName;
    }

    public void setJobName(String jobName) {
        this.jobName = jobName;
    }

    @Override
    public String toString() {
        return jobName ;
    }
}
class Person{
    //不可变
    private final String name;
    private final Job job;
    public Person(String name,Job job){
        this.name = name;
        this.job = job;
    }

    public String getName() {
        return name;
    }

    public Job getJob() {
        return job;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", job=" + job +
                '}';
    }
}

Out：
Person{name='Alex', job=Student}
Person{name='Alex', job=Beggar}
```

所以**不可变类中的可变成员变量都需要在类构造函数中创建出来**。对构造器的修改如下：

```
public Person(String name,Job job){
        this.name = name;
        this.job = new Job(job.getJobName());
    }
```
##三、缓存实例的不可变类
由于不可变实例不可被更改，所以相同的不可变实例要频繁开辟内存空间，这对系统资源是极大的消耗，所以我们要考虑缓存不可变实例。
**注：**当然盲目缓存，或者被缓存的对象使用频率很低会使得系统性能下降。
对Person修改如下：

```
public class Test {
    public static void main(String[] args) {
        Job job_1 = new Job("Student");
        //new 类似 new创建String
        //valueOf()相当于String s = "Student";类似于进入常量池
        Person person_1 = new Person("Alex",job_1);
        Person person_2 = new Person("Alex",job_1);
        Person person_3 = Person.valueOf("Alex",job_1);
        Person person_4 = Person.valueOf("Alex",job_1);
        System.out.println(person_1 == person_2);
        System.out.println(person_1 == person_3);
        System.out.println(person_3 == person_4);
    }
}
class Job{
    private String jobName;
    public Job(){}
    public Job(String jobName){
        this.jobName = jobName;
    }

    public String getJobName() {
        return jobName;
    }

    public void setJobName(String jobName) {
        this.jobName = jobName;
    }

    @Override
    public String toString() {
        return jobName ;
    }

    @Override
    public boolean equals(Object o) {
        return o.toString().equals(jobName);
    }

}
class Person{
    //建立缓存,要在类之前加载static
    private static int MAX_SIZE = 10;
    private static Person[] cacheSpace = new Person[MAX_SIZE];
    private static int position = 0;

    private final String name;
    private final Job job;

    //构造器也可以隐藏
    public Person(String name, Job job){
        this.name = name;
        this.job = new Job(job.getJobName());
    }
    public static Person valueOf(String name , Job job){
        for(int i = 0 ; i < cacheSpace.length ; i++){
            if (cacheSpace[i] != null &&
                    (cacheSpace[i].getName().equals(name)&&cacheSpace[i].getJob().equals(job))){
                return cacheSpace[i];
            }
        }
        if (position == MAX_SIZE){
            cacheSpace[0] = new Person(name,job);
            position = 1;
        }else{
            cacheSpace[position++] = new Person(name ,job);
        }
        return cacheSpace[position - 1];
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name) &&
                Objects.equals(job, person.job);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, job);
    }

    public String getName() {
        return name;
    }

    public Job getJob() {
        return job;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", job=" + job +
                '}';
    }
}
Out：
false
false
true
```

（参考了疯狂java讲义代码）