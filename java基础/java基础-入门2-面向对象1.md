---
title: java基础 入门2 面向对象1
date: 2021-02-03 22:43:23
tags: Java基础
categories: Java
---

# 第二章 面向对象

### 2.1 基础概念
#### 2.1.1 面向对象

面向对象：考虑我该找“谁”（对象）去做  ，例 java
面向过程：考虑我该怎么解决问题， 例 c
面向对象用来描述现实生活，java中万物皆对象！我们引入类Class的概念。

2021.1.23 补充 ： 不同阶段对概念的认知是不一样的。
面向对象是一种编程思想，不只是对于现实中的抽象。 golang 不是完整的面向对象语言，但是可以通过结构体实现面向对象思想。 我理解，面向对象是一种高度的抽象，对功能的合理复用和分类。封装、继承、多态可以使得代码的结构更清晰，复用程度和扩展性更高。

1. 类Class：
Java 把具有相同属性和相似行为的一类事物称为类。相同属性用属性来表示，相似行为用方法来表示。而找出这两者的过程称为抽象。抽象又分为过程抽象和数据抽象。通过封装行为将“属性” 和“行为”封装成类。例如：
```
class Person{
  int age;
  String name;

  void walk(){
    System.out.println("walking");
  }
}
```
注 ： 类的属性、行为不分先后没有关系，全部同级并列。

2. 对象：
创建对象： 
```
//class name = new class(); // 创建对象就是创建类的实例
Person p1 = new Person();
```

3. 成员：
成员分为成员变量和成员方法 （对应上文属性、行为）。目前类中只能放成员，成员都是并列的不分先后。成员变量就是类的属性，成员方法就是类的功能。访问成员只能通过对象去访问(暂时，略过static方法的静态调用)，例如：`p1.walk();`

4. 局部变量与成员变量区别：
Class中的变量：成员变量
方法里的变量：局部变量 (最主要的判定依据)
主要区别 ：作用范围不同， 成员变量类中起作用
1）初始化：
成员变量，不赋值有默认值
局部变量，不赋值不能使用
2）内存位置不同：  
成员变量跟随变量进入堆内存
局部变量跟随方法进入方法栈
3）声明周期不同：
堆中成员变量需要jvm清理，生命周期会长一点

2021.1.2.3 补 ： 
变量就分类型与作用范围， 不要起名成员变量、成员方法！ 学习时候看得视频教程有些误人子弟。

#### 2.1.2 类的初始化

如上所述，成员变量创建后，不赋值也有基本类型的默认值。如果我们希望这个变量有我们赋予的默认值，需要添加构造函数。

构造函数：构造函数是该函数名与类名相同，但没有返回值的函数。

特点 ： 构造函数在创建对象时会自动调用一次。 创建一个对象调用一次。不需要显示调用。现阶段，主要起赋初始值的作用。例如：
```
class Person{
  int age;
  String name;
  void walk(){}

  public Person(){
    this.name = “alex”;
    ...; // 构造函数代码
  }
}
```
构造函数也可以有参数，这样在new对象的时候可以直接对成员变量赋值。如果，没写构造函数，jvm会自动给类中添加无参构造函数。

定义初始化：class 的所有对象的某个属性都是固定值。
创建对象时，定义初始化与构造函数都会执行，定义初始化会比构造函数先执行。例如:
``` 
class Person{
  int height = 175; // 直接初始化
}
```

#### 2.1.3 方法的重载

1. 重载：同一个类中的一组函数，他们函数名字相同但参数列表不同。

2. 解决什么问题？
例如：写一个相加方法 sum ，传递的参数不只是两个 int ，想让double也可以调用，但还要使用同一方法名，这时就需要方法的重载。

3. **重载要求：参数列表不同（参数个数不同，或参数类型不同，或参数类型和个数都不同）**与函数的返回值类型没有关系。

4. 匹配原则：在函数调用的过程中，jvm会根据不同参数自动匹配相应的函数，符合就近原则（即如果没有最精确的匹配，jvm会选择能匹配上的，不需要人为干预）

5. 构造函数的重载：
```
class Person{
  int age;
  String name;
  void walk(){}

  public Person(){
    this.name = "alex";
    ...; // 构造函数代码
  }

  public Person(String name){
    this.name = name;
    ...; // 构造函数代码
  }
}
```

#### 2.1.4 初识内存分配

栈内存：main方法压栈
堆内存：变量开辟堆内存空间
方法区：在对象创建中，没有使用

过程：main 压栈运行，创建类变量后，先在堆空间对变量附默认值，而引用型变量只是储存堆空间的**地址**。其他成员方法依次压栈运行。程序执行完毕，方法、main，按照先进后出的弹出栈内存。而堆内存的释放，需等到 jvm 自动清理（gc）。

何为引用？
例如 `Person p 1= new Person(); Person p2 = p1; `
 p1称为引用了一个对象的实例，因为p1在栈内存空间中创建了一个区域用来存放堆内存空间中的地址，这个过程称作创建的一个对象的引用。p2为p1的引用， p2 还是指向了p1的地址。

#### 2.1.5 this ：当前对象的引用

this： 在类中的方法中，如果方法参数与成员变量同名，那么如何区分？用 this 区分，即方法中想访问成员变量通过 this 。
```
public Person(String name){
  this.name = name;
  ...; // 构造函数代码
}
```

this是什么？this 表示的是当前实例（的地址）例如`Person p1= new Person();` p1存放的是地址，p1中的this存储的也是当前对象的地址，可以说p1就是p1中的this。

注意：
`this();`表示通过this构造函数，这只能放在构造方法的第一句。否则对象没存没有创建，后续无法操作（super()一样）。

#### 2.1.6 其他

静态成员 static ：
```
class Person{
  int age;
  String name;
  static String SPECIES = "human";

  void walk(){}

  public Person(){
    this.name = "alex";
    ...; // 构造函数代码
  }
}
```
注 ： 
1. 静态成员属于类，不属于某个特定对象。
2. 使用类的时候完成静态成员的初始化。
3. 推荐类名调用访问，`Person.SPECIES`下文static关键字会着重介绍。

-----

### 第二节 封装、继承、多态

#### 2.2.1 封装

java 中方法和类都是封装体， 封装隐藏了实现细节（黑箱），对外只提供了公共访问方式（public）便于调用者使用。封装提高了类的复用性与安全性。

参考对八种基本数据类型的封装类， 例如 Integer暴露的方法 ， 如 Integer.max(a,b) 用来比较大小等。

#### 2.2.2 继承

1. 继承：描述事物之间的关系，在现有类的基础上创建新类，子类自动拥有父类可继承的所有方法和属性，也成为全部继承 。 所以 java 中的继承是全部继承（除构造函数外）。

2. 语法格式：`extends 关键字：class 子类 extends 父类 （）`

3. 继承的作用：子类对象可以调用父类成员，可以重写父类的方法，当然也可以调用自己的特有成员。所以在类之间存在继承关系时，必须同时观察两个类。

4. 优势：集成提高代码复用性，提高效率，使类与类之间产生关系，为多态提供前提。

#### 2.2.3 继承中的构造函数

类中发生继承，会影响构造函数的加载顺序，简言之遵循：

>”先静态后普通，先父类后子类“。

复习 ： 普通‘ 构造器/构造方法/构造函数’：作用初始化对象，或者在初始化的时候对对象赋值，不写的话 jvm 会自动添加空参构造方法。构造方法在 main 函数 new 对象的时候直接执行。

举例：
```
public Person( String name,int age){
  this.name = name;
  this.age = age;
}
```
main中调用：` Person p = new Person(“alex”,20);`

类的内存加载过程：main方法压栈，构造方法进栈，对象进堆，对象数据替换堆内存，对象 new 的地址传递给 this 。构造方法运行完毕后，p 指向堆内存空间（this 与 p 相同），构造方法出栈。
（目前没有涉及到 jvm 类加载器 部分知识）

其他：
1. 首先，继承中构造函数是不会继承，子类必须调用父类的构造函数。如果子类没有写构造函数，那么子类默认调用父类的无参数构造函数。如果要显示调用需要：super（参数列表） 置于第一行。
2. 继承中的顺序先后：**父类静态，子类静态，父类的定义初始化(`int age = 23;`)，父类的构造函数，子类的初始化，子类的构造函数 。**
3. 其实每一个类都有一个继承，继承于Object类。
4. 例子：
```
public class Test{
	public static void main(String[] args) {
		/*继承的演示：学生继承人类*/
		Person xiaoming = new Person("xiaoming", 23);
		xiaoming.eat();
		Student alex = new Student("ALex" , 24);
		alex.read();
	}
	static class Person{
		String name ;
		int age;
		Person(String name , int age){
			this.name = name;
			this.age = age;
		}
		void eat(){
			System.out.println(this.name + "正在吃饭");
		}
	}
	static class Student extends Person{
		void read(){
			System.out.println(this.name + "正在读书");
		}
		Student(String name , int age){
			super(name , age);
			this.name = name;
			this.age = age;
		}
	}
}
```

#### 2.2.4 关键字介绍 （面试题）
 
1. super：

使用场合：用于在子类构造方法中调用父类构造方法。（父类调不了子类的构造器）

使用方法：子类构造方法，有一个默认构造方法。大多数情况，你的子类构造方法什么都不写，其实默认添加`super();` ， 位置在子类构造方法的第一行。这称为”隐式调用“，会执行父类的空参构造方法。即使什么都不写，new子类对象时也会执行父类空参构造方法。如果，父类只有**有参构造方法**，为了那么super（） 也应含参。否则super()会 默认调用空参构造方法。

注意：构造方法不能继承

2. final ：
应用：有些东西就不想被修改，加个final关键字。
范围：类，类的成员，局部变量。但是不能修饰 ”抽象“ 类、”抽象“方法、接口。（抽象的方法就是用来重写的！抽象的类就是用来被继承的！面试常考）
例如: String 就是一个java 定义好的 final 类，不能继承。

1）final 修饰class，不能被继承。对于，不像被改动的类，直接final掉，这样不能继承类中的方法也不会被重写。

2）final 修饰方法，不能被重写，子类可以继承。就是不让改，但可以用。如果结合private 对子类不可见，用都不能用。

3）final 修饰变量，只能一次赋值，终身不变（也就是常量，常被用来大写表示常量。final + static 全局常量）。final 修饰成员变量必须有初值 ， 如果没有初值，就需要在static代码块中赋值。	
```
final int VALUE;//这是不可以的，成员变量没有赋值
final int VALUE = 1;
```
`final int VALUE = 10；`

4）final 修饰引用数据类型，只能赋一次地址，内存地址终身不变。其成员的值是可以改变的。
```
public class Student {
    String name;

    public Student(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                '}';
    }
}

public static void main(String[] args) {
        final Student s = new Student("zhangsan");
        System.out.println(s);
        s.name = "lisi";
        System.out.println(s);
    }
//out :
//Student{name='zhangsan'}
//Student{name='lisi'}
```
3. static：

static 修饰的成员变量变成了 共享数据。即成员变量属于类，不再属于对象的特有内容，新new的对象在堆内存中共同使用同一个 static 数据。经常访问的数据可以考虑使用，但不宜过度使用，并发时容易造成阻塞问题。
```
public class Person{
  String name;
  static Sting country;
}
//p1一旦赋值，p2调用country与p1相同。
```
注：static成员可以被类名直接调用：Person.country；同理，静态方法也可以类名直接调用，不需要new对象。

注：内存模型补充：
如果涉及到静态成员，静态成员的数据进入数据共享区（方法区）。程序执行前，先进内存先加载静态成员。接下来就是程序运行，jvm 从方法区复制main方法压栈运行balabala……

总之，静态的内存空间的生命周期要长于非静态对象。
**执行static静态时，对象还不存在。**

2021.2.3 补充 ：
jvm 类加载器 classloader 在加载 class 时就将类信息 和 static信息加载到了方法区。私以为，可以将其称为 java 中的共享内存。

注：
静态方法不能使用非静态变量，原因就是生命周期导致。静态优先于非静态存在于内存中，静态方法调用时非静态变量还不存在，没有办法调用。同理，非静态方法（普通方法）可以调用静态（static）。静态方法也可以调用静态变量。（static之间可以互相调用）同理， static 中也不能写 this 和 super。（生命周期， 自身和父类都没有加载）

使用场合：
事物之间的共性数据 ，定义成为静态成员变量。

#### 2.2.5 方法的重写

重写与重载的区别：

重写：针对**父类和子类**， 父类中的方法在子类中被重新声明一遍。
重载：针对**同一类**中的方法同一，接受不同的参数列表，例如构造函数，但是参数列表不同。

注意：
1. 子类权限须大于等于父类，否则会影响多态调用 。 四大权限：public 》protected 》（default）》private，default 权限默认不写，今后会详细解释。
2. 如果在子类重写方法中，想要调用父类方法，`super.func();` 。如果不重写的话，直接调用即可。例如 ：
父类parent：`void callName(){return 0;}`
子类son：`void callName(){super.callName();System.println("abc");}`
如果未被重写，直接 `son.callName();` 调用父类方法 ，如果已经重写，这样会调用子类的方法。

#### 2.2.6 多态

1. 多态：指同样的事物具有多种形态，直接表现为父类对象类型接收子类对象的初始化，也叫多态调用/动态绑定， 多态是各种框架存在的基础。

2. 调用公式：`接口（或者父类） 变量 = new 子类对象（）`

```
public class Student extends Person｛...//study()｝；
Person P = new Student();
```
3. 多态成立的前提：继承和方法的重写。

成员变量：在编译时参考父类的成员变量，编译运行全看父类。
成员方法：在编译时参考父类的被重写的方法。如果，父类没有这个方法编译失败。但运行时，运行子类的方法。
>编译看父类，运行看子类！

2021.2.3 补充 
之前没配例子，这里有些奇怪， 故粘了一段代码 ：https://www.jianshu.com/p/3d69236d198c
```
public class Test {
    public static void main(String[] args) {
        Parent p = new Child();
        System.out.println(p.name + "\t" + p.address);
        p.sayName();
    }
}

class Parent {
    public String name = "father";
    public static String address = "JiangXi";
    
    public void sayName() {
        System.out.println(name + "\t" + address);
    }
}

class Child extends Parent {
    public String name = "son";
    public static String address = "ZheJiang";
    
    public void sayName() {
        System.out.println(name + "\t" + address);
    }
}

out :
father  JiangXi
son ZheJiang
```
解释一下这个老哥的问题 ，其实是 java 中经典的 “多态向上转型” ：
1） 前提 ：java 是一种强制数据类型的语言，参与运算的数据类型必须统一。引用数据类型也是数据类型 ， 如果发生运算中类型不统一，由子类到父类的向上转型。但是，该例子， Child 转成了 Parent 吗？
```
        Parent p = new Child();
        System.out.println(p instanceof Parent);
        System.out.println(p instanceof Child);

        System.out.println(p.getClass().equals(Parent.class));
        System.out.println(p.getClass().equals(Child.class))

out :

true
true
false
true
```
结果非常有意思啊！ jdk 这里显示了不一致性，这个对象根据 java api 来看还是一个子类对象！矛盾点： 这个对象明明应该是一个父类对象， 它调用成员变量（或者静态变量和静态方法），都是调用父类的内容！
```
        Parent p = new Child();
        // 我认为， 应等价于下面两条命令
        Child c = new Child();
        Parent p = new Parent();
        p = c; // 方法覆盖
```
先 new 了一个完整的子类对象，再将子类对象向上转型赋值给父类引用。 在这个过程中， 只替换重写方法。 所以，**对象 p 具父类的引用， 却是子类对象**， 该对象方法的实现应该遵循子类实现，且只能调用父类方法！ （此处是各种框架的基础！！）

2）**多态中的向下转型： 有坑！**

最好就不要这么写了！这里面有坑，参考下面文章：
[https://blog.csdn.net/qq_31655965/article/details/54746235](https://blog.csdn.net/qq_31655965/article/details/54746235)

强制转型：`子类 变量名 = （子类）父类；`
优势：可以调用子类特有功能。

#### 2.2.7 四种访问权限

访问权限关键字可以修饰成员变量、成员方法，但是限定都是针对类class，毕竟java基于class嘛。 （这里面的包，就是文件夹）

1. private：私有权限，一般private修饰成员变量，该成员只能在类的内部访问。
2. protected：受保护权限，修饰成员，表示同一包下或者在不同包的子类中可以访问。（default+子类，平时使用较少）
3. default：默认权限，修饰成员和类，表示可以在同一包下访问，可以省略。
4. public：公共权限，修饰成员和类，不同包下也可以访问。

开发中的一般情况：成员变量 private，成员函数 public 。

#### 2.2.8 包的概念

1. 包：是类名的一部分，用于命名管理。对于系统来说包是文件夹，但对于 java 来说包是类名的一部分。需要在java文件开头进行声明 `package name;` 如果没声明包，jvm 认为文件在默认包下。

2. 导入包：`import XX.XXX;`

3. 包的环境变量：cmd命令
`>>set classpath=包的路径`
这样你在任意目录编译java文件都能找到相应包文件。

4. 声明规则：通常网址反写`com.alexljs.demo01`，IDE软件自动帮你建立文件夹，自动快捷导入。

