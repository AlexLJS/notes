---
title: 集合框架 hash与HashMap
date: 2020-08-17 22:38:06
tags: Java基础
categories: Java
---

# 一、哈希函数
#### 1、定义
>把任意长度输入，通过散列算法（哈希函数）映射成固定长度的输出。


#### 2、经典哈希函数特点 ：
>1、输入域无穷大
>2、输出域有穷尽
>3、唯一性，输入一定得到固定输出
>4、离散性输出域呈现均匀分布（等价于 ：输出与输入规律无关）
>5、不可逆

[https://blog.csdn.net/sinat_31011315/article/details/78699655](https://blog.csdn.net/sinat_31011315/article/details/78699655)

在java中，通过Object.hashcode() 可以得到对象的哈希值 ，源码 ： 

```
    @HotSpotIntrinsicCandidate
    public native int hashCode();
```

注： 使用native关键字说明这个方法是原生函数，也就是这个方法是用C/C++语言实现的，并且被编译成了DLL，由java去调用。这些函数的实现体在DLL中，JDK的源代码中并不包含，你应该是看不到的。对于不同的平台它们也是不同的。这也是java的底层机制，实际上java就是在不同的平台上调用不同的native方法实现对操作系统的访问的。[https://blog.csdn.net/bifuguo/article/details/81513526](https://blog.csdn.net/bifuguo/article/details/81513526)

通过如下测试，object的hashcode就是内存地址（的十进制表示）：

```
public class Test {
    public static void main(String[] args) {
        ListNode o = new ListNode(1);
        System.out.println(o.hashCode());
        System.out.println(o);
    }
}
```

输出 ：

```
1239731077
Utils.ListNode@49e4cb85
```
(49e4cb85 十六进制转十进制即为1239731077)

所以，很多文章分析hashcode的实现只能从String 重写的hashcode 来入手。

<!-- more -->

#### 3、String 的hashcode的实现 

手头这台机器是 JDK 11 ， 对HashCode 进行了优化：

```
public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            hash = h = isLatin1() ? StringLatin1.hashCode(value)
                                  : StringUTF16.hashCode(value);
        }
        return h;
    }
```
注 ： 其实这是是 String 的value改动，从jdk8的 char[] 变为 byte[] ，char是16位两个字节对应utf-16编码，而byte是8位一个字节在jdk11中 使用 latin1 编码 来节省空间。（实际上多数字符8位就能存储）

此文重点不在String，所以仅以 latin1的HashCode 为例 ： 
```
public static int hashCode(byte[] value) {
        int h = 0;
        for (byte v : value) {
            h = 31 * h + (v & 0xff);
        }
        return h;
    }
```
解释： 
`h = 31 * h + (v & 0xff);` 这里发生了byte ->  int 的向上转型， 计算机保存数字是以补码保存的， 如果计算机检测到v的高位是1 ， 即判断为负数。那么向上转型时，为了保证十进制数值的一致性， 八位之上的高位全补为1。但我们计算hashcode 根本不关心数值的一致性，所以 `& 0xff` ， 1111 1111 ， 只保留低八位的二进制值。至于 ， & 0xff 的其他作用参考 ：[https://blog.csdn.net/i6223671/article/details/88924481](https://blog.csdn.net/i6223671/article/details/88924481)

哈希计算公式可以计为 s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1] 
权重为何是31 ？ 网上给出解释 ：
> 一方面 ， 计算可拆解成位运算  `31*i=32*i-i=(i<<5)-i`  从而得到更好。 另一方面 ，31是奇质数不能进行因式分解减少了哈希碰撞的几率，且大小比较合适，而 16 - 1 = 15 则不满足要求。 

这样，String的hashcode() 也得到了 一个32位的二进制数， 与Object的hashcode() (8位16进制内存地址)同为32位的二进制数，方便统一在程序中使用。

#### 4、哈希碰撞 ： 会存在不同输入，对应同一输出

看如下例子：
```
public class Test {
    public static void main(String[] args) {
        String s1 = "gdejicbegh";
        String s2 = "hgebcijedg";

        System.out.println(s1.hashCode());
        System.out.println(s2.hashCode());
        
        System.out.println(s1.equals(s2));
        System.out.println(s1 == s2);

    }
}
```
```
-801038016
-801038016
false
false
```
从返回值可以看出，不同的String得到同样的哈希值，发生了哈希碰撞。由于输出域有穷尽，这是必然现象。此外，String 重写了equals 方法 ， 逐一比较每一个char（或 byte）所以即使hashcode 相等，equals 返回false。

#### 5、哈希函数的实现原理：

涉及到数学知识， 密码学知识 ， 有兴趣了解一下这个视频。
[https://www.bilibili.com/video/BV1TE411q7mW](https://www.bilibili.com/video/BV1TE411q7mW)


#### 6、由已知哈希快速形成若干个相互独立哈希函数：

h1 、 h2 为哈希函数的结果，视为种子，n为整数。那么无穷尽的相互独立哈希函数可以表示为 ： `newHash(n) = h1 + n * h2`

----


# 二、HashMap
#### 1、HashMap
>**问题 ： key - value可以为空值吗？**
答 ：可以 。HashMap源码注释翻译 ：  “HashTable 基于Map接口的实现。这个实现提供了Map接口的所有操作，并允许null值（value）和null键（key）。HashMap大致相当于HashTable，但是它（HashTable）是异步的且不允许kv为空）。 此类不保证Map的顺序；特别是，它不能保证顺序一直不变。”

#### 2、底层实现原理：


HashMap是以空间换时间的数据结构 ，底层使用 ：**数组 + 链表 + 红黑树 （jdk8以后）**
数组存放的是 Entry<k,v> 的实现类，称为entry数组（entry理解为是一种对应关系 ）: 
```
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        ...//省略若干方法
}
```
简述一下put 和 get的流程：

1、我们put 一对儿 key 和 value 的流程： 利用 key 算出key的hashcode()(原生的) ， 再算出hash （重写过的hash）， 创建Node对象， 再根据hash算出Entry数组的下标。最后把 这个node放进去。

2、根据 key get value 的流程：同样算两遍 hash ， 得出数组下标， 找到元素。


（关于 get 和 put 细节， 下文结合源码 ，详细展开）


#### 3、hash算法细节

数组的查询效率非常高 O(1) ， hashmap 要达到这个速度解决“如何利用对象的hashcode 算出Entry数组下标”。
HashMap 中先将 key的 hashcode （32位）的高16位 无符号右移动 16位 ， 并异或低16位。这样的目的是将整个hashcode 的信息都保存下来， 将高16位的信息整合到低16位，减少hash碰撞。而高16位因为 “异或0” ， 并没有发生变化。（详见 参考链接）源码如下：
```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
在默认情况下， Entry 数组长度 16， hash只取用后四位，通过位与&操作完成对数组下标映射。注意这里位与的是15 ， 二进制 ： 1111 ， 不但将数组的所有位置都利用上，而且在扩容时候留下**伏笔**。
```
tab[(n - 1) & hash]  // n为 table.length
```
可见，HashMap只是为了性能考虑，微调了hash算法。

参考 ：[https://blog.csdn.net/a314774167/article/details/100110216](https://blog.csdn.net/a314774167/article/details/100110216)


#### 4、 扩容机制 ：

因为哈希一定存在哈希碰撞， Entry数组的同一位置有多个元素的话，会采用链表链接在后面。这样get(key) 需要遍历链表， 如果链表过长会增加时间复杂度。当链表单链长过8时，链表的结构会被改为红黑树。为了减少这种情况的发生，最好的方式是在合适的时机对数组扩容。引出hashmap的容量和负载因子，默认是16和0.75 ， 即Entry数组 超过12被占用会发生二倍扩容。
 
查看构造方法 ，了解参数的默认值： 
```
public HashMap(int initialCapacity, float loadFactor) {
     private static final long serialVersionUID = 362498820763181265L;  
      static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16  
      static final int MAXIMUM_CAPACITY = 1 << 30;//最大容量  
      static final float DEFAULT_LOAD_FACTOR = 0.75f;//填充比  
      //当add一个元素到某个位桶，其链表长度达到8时将链表转换为红黑树  
      static final int TREEIFY_THRESHOLD = 8;  
      static final int UNTREEIFY_THRESHOLD = 6;  
      static final int MIN_TREEIFY_CAPACITY = 64;  
     transient Node<k,v>[] table;//存储元素的数组  
     transient Set<map.entry<k,v>> entrySet;  
    transient int size;//存放元素的个数  
     transient int modCount;//被修改的次数fast-fail机制  
     int threshold;//临界值 当实际大小(容量*填充比)超过临界值时，会进行扩容   
     final float loadFactor;//填充比（......后面略）
}
```
初始容量 ： 默认16 
容量上限 ： 2^31 ， 因为hashcode最多算32位，超出后，映射不到数组下标
负载因子 ： 默认0.75 
扩容时机： 数组的容量超过 数组大小的 0.75 （长度16 ， 则为 12）threshold 临界值。
扩容大小：一倍

**注意！！！**
**面试时候，“扩容”这里会被问一些很奇怪的问题 ：**
例1 ： hashmap的扩容机制？什么时候需要扩容？为什么要扩容两倍，而不是1.5倍 1倍？ 
答 ： 当然不是什么“位移操作快”！2333  …… 1、resize 时候  ，重新计算 node 在数组的下标值，位与（&）操作时会出现0，数组Entry的某些位置不会被分配值。如，默认情况下从16 扩容1.5倍到24 ， 
```
 10111 //24 - 1的二进制
// 10111 的0位置 位于任何key的hashcode，该位置都是0
 01000// 8 ， Entry 数组的索引8，就永远不会被使用，造成空间浪费
```
2、“扩容一倍”导致简便算法，不用重新计算每个node的hash值！上文说 & length-1 留伏笔，就用在这里！默认状态下，&15 ， 即 1111 。resize发生时， &31，11111，变化就是&多了一位1 。已知 ，原node的key不变，hashcode不变，hashcode的**倒数第五位**不是1 就是0 。 如果是0， 这次resize不用换位，否则此node索引前进16个位置！而判断倒数第五位刚好&原容量16 （& 10000）， 对应源码 ： `if ((e.hash & oldCap) == 0) {……`
 
例2  ：Hashmap扩容可能会带来什么问题？
答 ： 线程不安全resize的时候可能会导致死循环。这个问题几句话说不清，后面细说。
resize源码 ： 
```
 /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```
注意 ： 这里的resize 已经改成了 尾插法，在死循环问题上会用到。

#### 5、 put

```
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
 
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //table未初始化或者长度为0，进行扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //（n-1）& hash确定元素存放在哪个桶中，桶为空，新生节点放到桶中（此时这个节点是放在数组中）
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            //桶中已存在元素
            Node<K,V> e; K k;
            //比较桶中第一个元素（数组中的结点）的hash值相等，key相等
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                //将第一元素赋值给e,用e来记录
                e = p;
            else if (p instanceof TreeNode)//放到树中
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {//为链表节点
                //在链表最末插入结点
                for (int binCount = 0; ; ++binCount) {
                    //到达链表尾部
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //节点数量达到阀值，转化为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        //跳出循环
                        break;
                    }
                    //判断链表中节点的key与插入的元素的key值是否相等
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    //相等，跳出循环
                        break;
                    //用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                    p = e;
                }
            }
            //表示在桶中找到key值、hash值与插入元素相等的结点
            if (e != null) { // existing mapping for key
            //记录e的value
                V oldValue = e.value;
            //onlyIfAbsent 为false或者旧值为null
                if (!onlyIfAbsent || oldValue == null)
            //用新值替换旧值
                    e.value = value;
            //访问后回调
                afterNodeAccess(e);
            //返回旧值
                return oldValue;
            }
        }
        //修改次数
        ++modCount;
        //实际大小大于阈值则扩容
        if (++size > threshold)
            resize();
        //插入后回调
        afterNodeInsertion(evict);
        return null;

```
参考 ： https://blog.csdn.net/qq_39736103/java/article/details/105292954

#### 6、死循环

HashMap本身是线程不安全的， 在多线程中使用会出现各种问题。
jdk 1.8及之前，使用的是头插法，resize时发生死循环。具体看下文：
[https://www.jianshu.com/p/1e9cf0ac07f4](https://www.jianshu.com/p/1e9cf0ac07f4)

简言之，该node的list 顺序是 ： a->b->c  
 线程1刚开始resize()， 线程2 resize()完毕，但没有替换引用时，线程2停了 。这时可能还是同一个位置，因为是头插法，顺序变为：c->b->a ，线程1接过来，从原本的 b插a，变为a插b了。死循环。

（水平有限，这里勉强看懂，解释不清）

**1.8之后，就不会出现死循环了吗？**
还是会的， 只不过出现的原因不同。具体见下文 ： 
[https://blog.csdn.net/Liu_JM/article/details/105582711](https://blog.csdn.net/Liu_JM/article/details/105582711)
 
可见是在 list和tree转换的过程中，有时会出现不能跳出循环的情况。（原因不明）

#### 7、对比其他数据结构


hashtable  ， 线程安全 ，关键方法加了 synchronized