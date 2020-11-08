---
title: 杂记 Java代理模式
date: 2020-07-24 18:05:41
tags: Java基础
categories: Java
---

简单的几句话讲能不能讲明白代理模式……

----
## 一、简介

1. 什么是代理模式
答： 代理模式是一种封装，将被代理对象封装到代理对象中，由代理对象决定何时执行被代理对象操作。java中的代理是对反射的应用

2. 举一个案例 
主人 （Master ）的代理人是 仆人（Servant）。主人想买东西（buy）这个操作要交给仆人代理完成。

3. 分析案例
为什么叫“代理”？因为主人自己也能完成buy操作，但他觉得麻烦不自己做，他一天要买很多东西，不像在路上（“before”重复动作）浪费时间。

4. 模型抽象（静态代理）
抽象这个实例，将buy方法抽象成接口。

<!-- more -->

```
public class ProxyDemo1 {
    /**
     * 热身，静态代理
     */

    public static void main(String[] args) {
        Servant s = new Servant();
        s.buy("打印机");
    }
}

interface Human{
    void buy(String item);
}
class Master implements Human{

    @Override
    public void buy(String item) {
        System.out.println("buy..." + item);
    }
}

class Servant implements Human{
    private Master myMaster = new Master();
    @Override
    public void buy(String item) {
        System.out.println("before , 准备钱, 去商店……");
        myMaster.buy(item);
        System.out.println("after , 开车, 回家……");
    }
}
```
5. 静态代理分析
分析上述代码，得到两个优点： ① buy动作是由代理人（Servant）发出，对外没有暴露被代理人（Master），但实际buy动作定义在Master中。②对Human接口的buy方法进行了监控（添加了before和After）

![代理模式.png](https://upload-images.jianshu.io/upload_images/15578361-3302c653f88694cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6. 静态代理的问题
由上述案例，已知完成代理操作，得实现相同接口。那么问题来了，如果没有实现相同接口怎么代理？问题二，如果再来一个接口，假如Master是个罪犯（Criminal），能杀人（Kill），但管家不能帮我杀人，得找杀手（Killer）代理。能否写一个通用代理类不是编译期间创建，而是运行期间，要杀人就找杀手代理，要买东西就找管家？

```
interface Criminal{
    void kill(String name);
}
```

----

## 二、动态代理

1. 动态代理
我们通过反射，可以在运行时动态创建代理，就不再需要提前写好 Killer 和 Servant。将产生代理的结构用工厂类进行封装，代码如下：
```
class ProxyFactory{
    public static Object getProxyInstance(Object o){//传入被代理对象
        /**
         * classLoader
         * 创建的代理类需要实现被代理类的哪些接口
         * invocationHandler 通过代理对象调用原方法时 ， 会自动调用invoke方法
         */
        return  Proxy.newProxyInstance(o.getClass().getClassLoader(),o.getClass().getInterfaces(),new MyHandler(o));
    }
}
```
2. 实现InvocationHandler接口

```
class MyHandler implements InvocationHandler{

    private Object o;//本例：master

    public MyHandler(Object o) {
        this.o = o;
    }
    //method , args 方法参数
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object returnValue = null;
        //代理 实现逻辑
        if ("kill".equals(method.getName())){
            System.out.println("before, 买枪...");
            returnValue = method.invoke(o,args);
            System.out.println("after, 逃跑...");
        }else if ("buy".equals(method.getName())){
            System.out.println("before , 准备钱, 去商店……");
            returnValue = method.invoke(o,args);
            System.out.println("after , 开车, 回家……");
        }
        return returnValue;
    }
}
```

3. 运行与结果

```
public class ProxyDemo2 {
    /**
     * 动态代理
     */

    public static void main(String[] args) {

        Human humanProxy = (Human) ProxyFactory.getProxyInstance(new Master());
        Criminal criminalProxy = (Criminal) ProxyFactory.getProxyInstance(new Master());

        humanProxy.buy("面包");
        criminalProxy.kill("jack");
    }
}
```
```
before , 准备钱, 去商店……
buy...面包
after , 开车, 回家……
before, 买枪...
kill...jack
after, 逃跑...
```
----
## 三、总结

至此， 我们可以发现动态代理与静态代理差别仅在于 “代理” 是否用反射创建，本质没有什么差别。代理的本质是对接口方法的监控和对被代理对象的封装。而动态代理起到了解耦和降低代码量的作用。 代理模式主要应用在AOP和Mybatis中。  

如果没有相同的接口， 想使用代理模式需要用CGLIB 用底层技术创建被代理对象的子类，在子类中做父类方法的拦截。 这部分应该放在SpringAOP中整理了。