---
title: Java 开发规范
date: 2020-08-17 23:58:46
tags: Java基础
categories: Java
---
# 一、字符
## 1.空格：ASCII （0x20）
String 及String格式字符都将转义；Tab不用于缩进。
## 2.转义序列：(\b, \t, \n, \f, \r, \", \' and \\)
使用该序列而不是相应的八进制（例如\ 012）或 Unicode（例如\ u000a）
## 3.其他：
非unicode 字符保持可读性，推荐：
String unitAbbrev = "μs";
Best: perfectly clear even without a comment.

# 二、资源文件
## 1.版权许可：不可省略
## 2.Package语句：
不换行，不受每行最多字符限制
## 3.导包：
禁止使用通配符静态导入
## 4.顺序：ASCII排序顺序
从一个组中静态导入；com.google
## 5.Class声明：
类名与顶层类相同；保证可阅读性，类的成员（变量&方法）排序不使用时间顺序，按照一定的内在逻辑。
## 6.构造方法重载：
多个重载构造方法之间，不要添加成员变量


<!-- more -->

# 三、格式
## 1.大括号（代码块）：不可省略
只有一条语句也不能省略；遵循 Kernighan and Ritchie 风格（《C程序设计语言(TheCProgrammingLanguage)》）
```
return new MyClass() {
  @Override 
  public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    }
  }
};
```
其实跟多数ide 相同， 不要自己在括号开闭时候添加多余换行，降低阅读性。
空代码块，两个括号紧邻。
## 2.缩进：两个空格
每次打开代码块，要缩进两个空格
## 3.语句：
一条语句一行；
根据不同项目，每行最多80/100个字符限制（package / import 语句， URL  ，shell命令不受限制）
## 4.换行位置：
（没有明确规范，个人习惯）
非赋值运算符处换行；在一些其他运算符处换行（，、&等等）
## 5.空白：
空行只用于分隔两个逻辑部分，比如所有成员变量和所有成员方法之间。
## 6.分组括号：
必要分组括号，没人能记清楚每个运算符的优先级。
## 7.变量声明：
一个变量声明，只声明一个变量，不要使用
int a，b;
##8.switch：
```
switch (input) {
  case 1:
  case 2:
    prepareOneOrTwo();
    // fall through
  case 3:
    handleOneTwoOrThree();
    break;
  default:
    handleLargeNumber(input);
}
```
default 语句即使没有代码，也不可省略。
## 9.修饰符顺序：
`public protected private abstract static final transient volatile synchronized native strictfp`
## 10.数字文字：
长整型，结尾大写L， 以免和1混淆
# 四、命名
## 1.减少下划线： name_, mName, s_name and kName,禁止。
## 2.包命名：
包名称全部小写，连续的单词简单连接在一起（没有下划线）。 例如，com.example.deepspace，而不是com.example.deepSpace或com.example.deep_space。
## 3.Class类命名：驼峰
UpperCamelCase，通常使用名词和短语。
测试类的名称以测试的类名称开头，以Test结尾。 例如，HashTest或HashIntegrationTest。
## 4.Method命名：驼峰
lowerCamelCase。
## 5.常量命名：大写
```
// Constants
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final Joiner COMMA_JOINER = Joiner.on(',');  // because Joiner is immutable
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }

// Not constants
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```
需要分辨什么是不可改变的常量。
## 6.驼峰命名：
基本都是驼峰命名规则，后序不再赘述。
