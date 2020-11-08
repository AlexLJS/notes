---
title: SQL 语法简介
date: 2020-08-06 21:05:29
tags: MySQL
categories: Database
---


## 一、 SQL语言
SQL语言是数据库中编程代码，译为结构化查询语言。（DML、DDL、DCL）

特点：
>1、不区分大小写
2、单行多行无所谓
3、有空格要求，分号结尾

建议规范：关键字大写，字母定义的数据表名用小写，注释同java：
```
SELECT * FROM infosharing.news_1;
/*是注释*/
```

## 二、基础通用语法

所谓通用语法无非 ，增删改查。

1、 **常用数据类型：**

int 整型 
double浮点  
varchar可变字符变量（根据数据改变存储空间） 
datetime时间类型
例如：varchar(20)  类型是长度为20 字符的数据类型
（除此之外还有大量的数据类型）

2、 **创建数据库**

创建 test 数据库，切换数据库

Create database test;
use  test;

注 ： MySQL 中 schema 与 DB 同意， Oracle 不同。

<!-- more -->

创建时指定默认字符编码：

`CREATE SCHEMA 'database_name' DEFAULT CHARACTER SET utf8;`

设置编码方式 

`character set UTF-8; `

3、 数据库操作

查看所有数据库：

`Show databases;`


删除数据库：

  `Drop 数据库名;`

使用数据库：

`Use 数据库名;`

创建数据表：

`Create table 数据表名(列1 数据类型 约束，列2 数据类型 约束，...);`

```
Create table myTable (
id INT NOT NULL AUTO_INCREMENT,
name VARCHAR(45)  NULL,
date DATE NULL,
PRIMARY KEY (`id`));
```
注 ： 最后一个列不写“，”
SQL语句并列值的最后都是不写 逗号 的， 今后写动态 sql 此处可能会报错。需要用 trim 或者 suffix 或者其他相关操作处理掉 最后一位的逗号。


注：什么是约束？ 约束是用来限制这个列的写入规则。

约束分类：主键约束（主键约束只有一种，唯一且非空。开发过程中，主键约束不能具有任何含义，例如编号可以，但电话号码不行，只能用来标识唯一性，primary key。主键约束实现自增 auto_incretment ,添加在后面），唯一约束，非空约束等。

4、数据表的操作

查看数据表的结构：

 `Desc 数据表名;`

删除表：

`Drop table 数据表名;`

修改表结构：统一关键字：**Alter table**

增加一列：`Alter table 表名 add 列名 数据类型（长度） 约束 ;`
修改列的数据类型：`Alter table 表名 modify 列名 数据类型（长度） 约束 ;`
修改列名：`Alter table 表名 change 旧列名 新列名 数据类型（长度） 约束 ;`
删除列：`Alter table 表名 drop 列名 ;`
改表名：`rename table 表名 to 新表名;`
也可以修改表的字符集（编码类型）character set

>其实，上述操作在日常开发中不常用。 关系型数据库， 在开发前已经将表的结构定义完全。 即使后期发生改动， 一般不由java程序员操作。对于个人项目，建议由可视化工具进行可视化操作。


#### 增 

向数据表中添加一条数据 ：

`Insert into 表名（列名1，列名2...） values (值1，值2...);`

注：
1、字符串用单引号  ，除了数字都要用单引号
2、主键设置为自增时候，不要添加主键值
3、如果给出全部列的值，列名可省略

添加多条数据 批量写入：

`Insert into 表名（列名1，列名2...） values (值1，值2...)，(值1，值2...);`

常用组合语句， 插入检索的数据 ： 

`insert into table1(col1,col2 ) select col1,col2 from table2 where ...;`

复制表 ：

`create table newTable as select * from oldTable; `

### 改
 
更新数据：

`Update 表名 set 列1= 值1 ， 列2 = 值2  where 条件;`

注：如果不加条件，则所有值都会被修改 ， 即批量修改。
Where 后面的条件一般选择，唯一性条件：例如主键值 id = 2，改动id 为2的值
也可以使用筛选条件，进行批量修改数据，and 关键字、<>不等于关键字等等。

### 删

删除数据：

`Delete from 表名 where 条件;`

删除表：(区别 drop table ，此操作清空所有行)

`Truncate table 表名;`

> 删除操作没啥意义， 业务中数据库存储所有数据包括“用户层面被删除的数据”，DB中通过修改字段实现删除功能， 不会真正删除数据。
## 三 、 查询操作(最重要的部分)

注：win命令行编码乱码问题的解决：
Windows的默认编码是GBK ，不是UTF-8，在dos命令行下，MySQL执行下面命令：`Set 列名 ‘gbk’;` 即可临时性，解决乱码问题。另一种解决方式是修改MySQL的配置文件，my.ini 。

`Select 列1，列2... from 表名；`
查询这个表中的指定列。

`Select *  from 表名；`
查询所有列的数据。

**进阶功能：**

distinct 查询去除重复的记录：
`Select distinct 字段 from 表名;`
所有结果字段值相同就会被去重：

limit 限制返回行数， 常用与分页操作：
`select * from myTable limit 开始行,查询总行数`
`select * from myTable limit n`返回前n行

查询重新命名列：
`Select 列名 as ‘临时列名’ from 表名;`

查询结果可以直接做数学运算，可以直接对列名进行加减乘除运算：
`Select 列1，列2*10  from 表名；`
列2的数据都会乘以10

## 四、 过滤 及 其他

使用where 进行条件查询 ： 
```
Between and 
In // 用于匹配一组值 ， 也可以接一个子查询
Is null 
Like
And 
Or 
Not
...
```

使用 Like 模糊查询：

模糊查询必须要配合通配符：
` like ‘%支出’` 以支出为结尾的
` ‘%支出%’`只要有支出即可
`Select * from 表名 where 列名 like  ‘_’; `_ 表示一个字符 ， %表示多个字符 

使用order 排序查询：
  `Order by 列名 [desc][asc]`
依照某列进行降序，升序排列 , 默认升序asc ， 只用记下 desc即可。


聚合函数：
对一列的数据进行计算，返回一个值。例如，求和、求最小值、求平均值…

常用聚合函数:
Count 求表中的数据个数：
`Select count(*) from 表名；`统计表中又多少行数据

Sum 求和运算：
`Select sum（列名） from 表名 ；`

Max、 min 、avg 函数使用同理。

分组查询：
`Group by 被分组的列名` 
分组查询会自动按照被分组列进行排序。如果想对聚合函数计算的值进行过滤，只能用关键字：`having 条件； ` （使用方式相当于where）， 而where 行过滤的优先级更高最先过滤。分组举例 ： 要求统计每个班级中分数大于80分的同学的个数 ， 并且返回数量大于2的情况（为了测试having）

![测试table： student](https://upload-images.jianshu.io/upload_images/15578361-11f82314c43fa114.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**除了聚合列外， select 每一列都必须是Group by 分组的依据**
```
select class,score,count(*) as num from testDB.student  where score > 80 group by class,score having num >= 2;
```



