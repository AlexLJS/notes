---
title: Redis 数据类型常用命令
date: 2020-07-06 00:13:54
tags: Redis
categories: Database
---
命令查询中文文档 ： [http://www.redis.cn/commands.html](http://www.redis.cn/commands.html)

换库 ： select [dbIndex]


## 一、Key


1. 查看key：`keys [pattern]`
不建议使用`keys *` 查看所有Keys。建议正则匹配， 如 `keys t*` 

2. 添加元素： `set [key] [value]`
发生冲突则覆盖

3. 取得元素： `get [key]`

4. 判断key是否存在： `EXISTS [key]`
存在返回 integer 1， 否则 0

5. 移动key到某个数据库： `move [key] [dbIndex]`

6. 设置key过期时间： `expire [key] [seconds]`
秒为单位设置key的过期时间， 已过期则移除DB

7. 查看key还有多少秒过期： `ttl [key]`
-1 表示永不过期， -2 表示已经过期

8. 查看key是什么类型： `type [key]`

9. 删除key ： `del [key]`
<!-- more -->
## 二、String

1. 添加、获得、删除、长度： `set/get/del/append [key] [str]/strlen [key]`
操作对应的value string 对象

2. 数字操作自加、自减、加某值、减某值： `incr [key]/decr [key]/incrby [key] [num]/decrby [key] [num]`
（v为num生效，否则报错）

3. 范围操作： `getrange [key] [start] [end]/setrange [key] [start] [value]`
类似substring的范围操作

4. 添加过期时间： `Setex [key] [second] [value]`

5. 如果不存在设置值： `setnx [key] [value]`
省去一步判空

6. 合并操作，批量添加、修改、非空添加： `mset [k1] [v1] [k2] [v2]…/mget [k1] [k2].../msetnx [k1] [v1] [k2] [v2]…`

7. 先get再set： `getset [key] [value]`

## 三、List
1. 左插值、右插值、选取范围：`lpush [key] [values]/rpush [key] [values]/lrange [key] [start] [end]`
0 ~ -1 则为全选

2. 左弹出、右弹出： `lpop/rpop [key]`

2. 取索引元素： `lindex [key]`

3. list长度： `llen [key]`

4. 删除N个值value： `lrem [key] [N] [value]`

5. 截取范围值，赋值给key： `ltrim [key] [start] [end]`

6. 右出左压： `rpoplpush [key1] [key2]`
反之亦然

7. 设置某下标值： `lset [key] [index] [value]`

8. 插入值 ： `linsert [key] after/before [value] [insertValue]`


## 四、Set
1. 插值、得到值、判断是否在集合中：`sadd [key] [values]/smember [key] /sismember [key] [value]`

2. 获取集合元素个数： `scard [key]`

3. 删除元素： `srem [key] [value]`

4. 随机出N个整数： `srandmember [key] [N]`

5. 随机出栈： `spop [key]`

6. 将key1的某值加入到key2： `smove [key1] [key2] [key1_value] `

7. 差集： `sdiff [key1] [key2] [key3]...`
key1 - （key2 + key3 + ...）

8. 交集： `sinter [key1] [key2] [key3]...`

9. 并集 ： `sunion [key1] [key2] [key3]...`


## 五、Hash （重要）
key-value关系存在，value 是一个键值对

1. 插值、取值、批量插入、批量取出、得到key下所有kv关系、删除key：`hset [key] [value-k] [value-v]/hget [key] [value-k] /hmset [key] .../hmget  [key] .../hgetall [key]/del [key]`

2. 长度： `hlen [key]`

3. subkey 存在性： `hexists [key] [subkey]`

4. 得到value 的keys 和values： `hkeys [key] / hvals [key]`

5. 自增某某值： `hincrby [key] [subkey] [int]/ hincrbyfloat [key] [subkey] [float]`

6. 将key1的某值加入到key2： `smove [key1] [key2] [key1_value] `


## 六、Sorted Set 

```
zset ：
---key1  ： 
---------------score1 : v1
---------------score2 : v2
...
```

1. 插值、取值、取带score值： `zadd [key] [score1] [value1].../ zrange [key] [start] [end]/zrange [key] [start] [end] withscores`
key1 - （key2 + key3 + ...）

2. 分数范文取值： 
`zrangebyscore [key] [score1] [score2]` 左闭右闭
`zrangebyscore [key] [score1] ( [score2]`左闭右开
`zrangebyscore [key] ([score1] ( [score2]`左开右开
`zrangebyscore [key] [score1] [score2] limit [num1] [num2]` 从score1- score2 结果中的索引num1位置，截取num2个结果
（类似分页操作）
 

3. 删除 ： `zrem [key] [value]`

4. 统计个数、统计范围个数： `zcard [key]/ zcount [key] [score1] [score2]`


5. 获得下标值： `zrank [key] [value]`

6. 获得对应分数： `zscore [key] `

7. 得到逆序排名： `zrevrank [key] [startindex] [endindex]`
zrevrange 、zrevrangebyscore 同理不再赘述

