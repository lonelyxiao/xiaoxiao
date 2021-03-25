# 安装

下载、解压、编译

ps:https://redis.io/download  有安装介绍

```shell
wget http://download.redis.io/releases/redis-2.8.18.tar.gz    
tar xzf redis-2.8.18.tar.gz     
cd redis-2.8.１８    
make    
```

注：执行make时可能会出现的错误:

1. 未安装gcc，请先: yum install gcc tcl -y；

2. 安装报错 error: jemalloc/jemalloc.h: No such file or directory；解决方案：make 换==》make MALLOC = libc

```shell
## 安装
[root@localhost redis-6.2.1]# make install
## 查看安装结果
[root@localhost redis-6.2.1]# cd /usr/local/bin
[root@localhost bin]# ll
总用量 18844
-rwxr-xr-x. 1 root root 4833416 3月  22 10:09 redis-benchmark
lrwxrwxrwx. 1 root root      12 3月  22 10:09 redis-check-aof -> redis-server
lrwxrwxrwx. 1 root root      12 3月  22 10:09 redis-check-rdb -> redis-server
-rwxr-xr-x. 1 root root 5003464 3月  22 10:09 redis-cli
lrwxrwxrwx. 1 root root      12 3月  22 10:09 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 9450288 3月  22 10:09 redis-server

## 备份配置文件
[root@localhost redis-6.2.1]# cp redis.conf redis.conf.bak

# 测试启动
cd /usr/local/redis    
./redis-server redis.conf
```

- redis默认不是后台启动的，如果想要后台启动，一个办法就是修改配置文件

```conf
## 将其修改为yes
daemonize yes
```

- 关闭redis

```shell
[root@localhost redis-6.2.1]# redis-cli 
127.0.0.1:6379> shutdown 
```

# 性能测试

- redis-benchmark
- redis自带的测试

```shell
[root@localhost redis-6.2.1]# redis-benchmark -c 100 -n 100000
```

- 日志查看

```shell
====== SET ======                                                    
  100000 requests completed in 1.01 seconds
  100 parallel clients
  3 bytes payload ## 每次请求3个字节
  keep alive: 1  ## 只一台服务器接收请求
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": no
  multi-thread: no

Latency by percentile distribution:
0.000% <= 0.127 milliseconds (cumulative count 1) ##  
50.000% <= 0.463 milliseconds (cumulative count 54809)
```

# 基础知识

- redis默认16个数据库
- 切换数据库

```shell
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]>
```

- 查看数据库大小

```shell
127.0.0.1:6379[1]> DBSIZE
(integer) 1
```

- 清空当前数据库

```shell
127.0.0.1:6379[1]> FLUSHDB
OK
127.0.0.1:6379[1]> DBSIZE
(integer) 0

```

# 数据结构

## 字符串操作

### 注意

- 键命名规范：通常，公司用：：来代表一个级别

- redis key 值是二进制安全的，这意味着可以用任何二进制序列作为key值

  key取值原则

  - 键值不需要过长，会消耗内存
  - 键值不宜太短，可读性差

  一个字符类型的值最多能存512M字节的内容

### set

- 设置值

```shell
127.0.0.1:6379> set name laoxiao
OK
```

- 设置过期时间,查看过期时间

```shell
127.0.0.1:6379> EXPIRE name 10
(integer) 1
127.0.0.1:6379> ttl name
(integer) 2
```

- 在SET命令中，有很多选项可用来修改命令的行为。 以下是SET命令可用选项的基本语法。
  - EX seconds − 设置指定的到期时间(以秒为单位)。
  - PX milliseconds - 设置指定的到期时间(以毫秒为单位)。
  - NX - 仅在键不存在时设置键。
  - XX - 只有在键已存在时才设置。

```shell
redis 127.0.0.1:6379> SET KEY VALUE [EX seconds] [PX milliseconds] [NX|XX]
```

- 判断key是否存在

```shell
127.0.0.1:6379> EXISTS name
(integer) 0
```

- 移除key

```shell
127.0.0.1:6379> MOVE name 1
(integer) 1
```

- 自增,自减
  - 可以用来设置某个文章的阅读量

```
127.0.0.1:6379> INCR count
(integer) 1
127.0.0.1:6379> DECR count
(integer) 0
```

- 自增指定数字

```shell
127.0.0.1:6379> INCRBY views 10
(integer) 11
```

- 替换

```shell
127.0.0.1:6379> SETRANGE name 2 zy
(integer) 7
127.0.0.1:6379> get name
"lazyiao"
```

### mset

- 表示一次可以设置多个键值对

```shell
redis 127.0.0.1:6379> MSET key1 value1 key2 value2 .. keyN valueN
```

- msetnx 是原子性的，如果一个没有设置成功，则其他键值对都设置不成功
- 可以用mset来设置用户是否已读文章，如： title:userid:docId  1,设置多个

### Redis的CAS

- getset
- 如果存在值则进行替换

## List

### 基本操作

- list命令都是L或者R开头的
- 元素时字符串类型，列表头部和尾部增删快、元素可以重复，最多存2^32-1个元素
- push操作,往key里面设置值，放入列表的头部

```shell
127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> LPUSH list 2
(integer) 2
127.0.0.1:6379> lpush list 3
(integer) 3
```

- 获取值

```shell
127.0.0.1:6379> LRANGE list 0 1
1) "3"
2) "2"
```

- 将值push到列表尾部

```shell
127.0.0.1:6379> RPush list rpu
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "3"
2) "2"
3) "one"
4) "rpu"
```

- 从左边弹出值

```shell
127.0.0.1:6379> LPOP list
"3"
127.0.0.1:6379> LRANGE list 0 -1
1) "2"
2) "one"
3) "rpu"
```

### 移除

- 移除指定值
  - count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
  - count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。

  - count = 0 : 移除表中所有与 VALUE 相等的值。

```shell
127.0.0.1:6379> LREM key count element
```

- 移除一个值
  - 移除1个指定的值
  - 可以用来如:取消某个人的关注

```shell
127.0.0.1:6379> LREM list 1 one
(integer) 1

```

### 截断

- 设置 0 1 2 3四个元素
- 截取index =1  2的元素
- 可以看到四个元素只剩下1 2了

```shell
127.0.0.1:6379> LTRIM mylist 1 2
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "2"
2) "1"
```

### 弹入弹出

- 从右边弹出一个元素，从左边push到一个新的元素中

```shell
127.0.0.1:6379> RPOPLPUSH mylist mylist2
"1"
```

### 场景

- 队列
  - LPUSH  RPOP
- 栈
  - LPUSH LPOP

## SET集合

- 无序的、去重的、元素是字符串类型

```shell
# 添加一个或者多个成员
SADD key member1 [member2]
# 返回集合中的所有成员
smembers key
# 判断 member 元素是否是集合 key 的成员
## 存在返回1
sismember key member
# 返回集合中一个或多个随机数
srandmember key [count]
# 获取集合的成员数
scard key
# 移除并返回集合中的一个随机元素

```

- 获取集合个数

```shell
127.0.0.1:6379> SCARD myset
(integer) 4
```

- 移除指定元素

```shell
127.0.0.1:6379> SREM myset 3
(integer) 1
```

### 多个集合操作

- 将一个集合中的指定元素移动到另一个集合

```shell
127.0.0.1:6379> SMOVE myset myset2 2
(integer) 1
```

- 求差集

```shell
127.0.0.1:6379> SDIFF myset myset2
1) "1"
2) "1,"
```

- 求交集(用户之间的共同关注)
  - A用户的关注放一个集合，粉丝放一个集合
  - A用户和B用户的关注求交集就是共同关注

```shell
127.0.0.1:6379> sadd myset laoxiao
(integer) 1
127.0.0.1:6379> sadd myset2 laoxiao
(integer) 1
127.0.0.1:6379> SINTER myset myset2
1) "laoxiao"
```

- 求并集

```shell
127.0.0.1:6379> SUNION myset myset2
1) "1"
2) "laoxiao"
3) "2,"
4) "1,"
```

## HASH(哈希)

- 由field和关联的value组成的map键值对

- field和value是字符串类型

- 一个hash中最多包含2^32-1个键值对

```shell
set key field value
```

```shell
127.0.0.1:6379> hset myhash name laoxiao
(integer) 1
127.0.0.1:6379> hget myhash name
"laoxiao"
```

- 获取所有的键或者值

```shell
127.0.0.1:6379> HKEYS myhash
1) "name"
127.0.0.1:6379> HVALS myhash
1) "laoxiao"
```



- 不适用hash的情况
  - 使用二进制位操作命令
  - 使用过期键功能

## 有序集合

- 添加语法

```shell
127.0.0.1:6379> ZADD key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]
```

- 返回指定下标区间

```shell
127.0.0.1:6379> ZRANGE key min max [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES]
## 查询下标的元素
127.0.0.1:6379> zrange myz 1 2
1) "lisi"
2) "wangwu"

```



- 按照分数查询（返回指定分数区间）

```shell
127.0.0.1:6379> ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
```

- 获取分数2-3开始

```shell
127.0.0.1:6379> ZRANGEBYSCORE myz 2 3
1) "lisi"
2) "wangwu"
```

- 从负无穷到正无穷查询（升序查找）

```shell
127.0.0.1:6379> ZRANGEBYSCORE myz -inf +inf
1) "laoxiao"
2) "lisi"
3) "wangwu"
```

- 降序查询

```shell
127.0.0.1:6379> ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
```

# 特殊数据类型

## 经纬度

- 3.2版本推出
- 可以计算地理位置的距离
  - longitude 经度
  - latitude 纬度

```shell

GEOADD key [NX|XX] [CH] longitude latitude member [longitude latitude member
## 插入地理位置
## 两级无法添加
127.0.0.1:6379> GEOADD china:city 112.98626 28.25591 changsha
(integer) 1
127.0.0.1:6379> GEOADD china:city 113.64317 28.16378 liuyang
(integer) 1

```

- 获取经纬度

```shell
127.0.0.1:6379> GEOPOS china:city changsha
1) 1) "112.98626035451889038"
   2) "28.25590931465907119"
```

- 获取城市距离

```shell
127.0.0.1:6379> GEODIST china:city changsha liuyang
"65197.3795"

##计算的距离单位（km）
127.0.0.1:6379> GEOdist china:city changsha liuyang km
"65.1974"

```

- 求附近人，（以半径为中心）
  - 经度， 维度：longitude， latitude
  - radius：半径
  - withcoord：显示经度维度
  - withdist：直线距离
  - COUNT：查出来的数量

```shell
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC|DESC] [STORE key] [STOREDIST key]

127.0.0.1:6379> GEORADIUS china:city 112 28 200 km
1) "changsha"
2) "liuyang"
```

- 以元素为中心寻找周围城市

```shell
GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC|DESC] [STORE key] [STOREDIST key]

## 获取长沙100km访问的城市
127.0.0.1:6379> GEORADIUSBYMEMBER china:city changsha 100 km withdist
1) 1) "changsha"
   2) "0.0000"
2) 1) "liuyang"
   2) "65.1974"
```

- 删除地理位置

```shell
127.0.0.1:6379> ZREM china:city liuyang
```

## 基数

基数：一组集合中，不重复的数据量

- 网页的UV(一个人访问网站多次，但还是算作一个人访问)
  - 传统方式：使用set保存userId---但是用户的数量大，就有弊端
  - Hyperloglog：占用内存固定，2^64不同的元素，只需要12kb，但是有0.81%的错误率

```shell
PFADD key element [element ...]

## 存入用户
127.0.0.1:6379> PFADD uv user1 user2 user3 user4 user5
(integer) 1
## 统计
127.0.0.1:6379> PFCOUNT uv
(integer) 5

##合并两个集合
127.0.0.1:6379> PFADD uv2 user3 user 5 user6
(integer) 1
127.0.0.1:6379> PFMERGE uv3 uv uv2
OK

```



## 位运算

- 网站用户的上线次数统计（活跃用户），统计用户的活跃信息， 活跃 0 不活跃 1

  - 网站用户的上线次数统计（活跃用户）

  - 用户id作为key，天作为offset，上线置为1

  - 例如：id为500的用户，今年第1天上线、第30天上线

    setbit u500 1 1

    setbit u500 30 1

    bitcount u500

- 打卡， 哪天打卡=1

```shell
## 设置第一天 第三天打卡
127.0.0.1:6379> SETBIT u1 0 1
(integer) 0
127.0.0.1:6379> SETBIT u1 2 1
(integer) 0
##获取第二天和第三天有没有打卡（默认0）
127.0.0.1:6379> GETBIT u1 1
(integer) 0
127.0.0.1:6379> GETBIT u1 2
(integer) 1
## 获取用户打卡天数
127.0.0.1:6379> BITCOUNT u1 
(integer) 2

```

# 事务

- Redis单条命令保证原子性，但是事务不保证原子性
- Redis事务没有隔离级别概念
- Redis事务本质：一组命令，在队列中，按照顺序执行

- Redis事务
  - 开启事务：MULTI 
  - 命令入队
  - 执行事务

```shell
## 事务开启
127.0.0.1:6379> MULTI
OK
## 入队操作
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
## 执行命令
127.0.0.1:6379(TX)> EXEC
1) OK
2) OK

```

- 放弃事务
  - 事务里的命令不会执行

```shell
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> DISCARD
OK
```

- 错误
  - 命令错误，其他命令不会执行
  - 运行时异常，其他命令照样执行

# Redis乐观锁

```shell
127.0.0.1:6379> set money 100
OK
## 监控money
127.0.0.1:6379> WATCH money
OK
127.0.0.1:6379> MULTI
OK
## 执行新增的时候，在另一个线程执行加20
127.0.0.1:6379(TX)> INCRBY money 10
QUEUED
##执行命令的时候发现money值改了，不再进行修改
127.0.0.1:6379(TX)> EXEC
(nil)
127.0.0.1:6379> get money
"120"
## 解除监控 （ps：执行失败要先解锁，再执行watch）
127.0.0.1:6379> UNWATCH
OK
```

# 安全配置

```shell
## 默认获取密码是为空的
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
## 设置密码
127.0.0.1:6379> config set requirepass 123456
OK
## 再吃执行命令没有权限
127.0.0.1:6379> config set requirepass
(error) ERR Unknown subcommand
## 认证
127.0.0.1:6379> auth 123456
OK
###
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
```



# 配置文件

```shell
## 可以导入多个配置文件
# include /path/to/local.conf
# include /path/to/other.conf

## 绑定ip
bind 0.0.0.0 -::1
port 6379

# 是否以守护进程运行，默认是NO
daemonize yes
## 如果以守护进程运行，则需要指定一个进程文件
pidfile /var/run/redis_6379.pid

#日志级别
loglevel notice

## 日志文件名
logfile ""

#### 持久化配置

## 如果3600秒内有一个key修改，就进行持久化操作
# save 3600 1
# save 300 100
## 60秒内有一个key修改，就进行持久化
# save 60 10000
##持久化出错，是否继续工作
stop-writes-on-bgsave-error yes

## 是否压缩持久化（rdb）文件（会消耗cpu资源）
rdbcompression yes

## 是否校验rdb文件
rdbchecksum yes

# rdb保存文件
dbfilename dump.rdb

################################主从复制


####################### SECURITY(安全)

# 在配置文件中设置密码
# requirepass foobared

##最大的客户端连接数
# maxclients 10000

### 最大内存配置
# maxmemory <bytes>

## 内存满了的策略
# maxmemory-policy noeviction


##########APPEND ONLY MODE (另一种持久化模式)
## 默认不开启
appendonly no
## 持久化文件
appendfilename "appendonly.aof"

### 同步机制
## 每次修改都写入（速度慢）
# appendfsync always
## 每一秒同步
appendfsync everysec
## 不执行sync， 操作系统自己同步
# appendfsync no
```

# Redis 持久化

## RDB模式

- RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘。也是默认的持久化方式，这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb
- 既然RDB机制是通过把某个时刻的所有数据生成一个快照来保存，那么就应该有一种触发机制，是实现这个过程。对于RDB来说，提供了三种机制：save、bgsave、自动化
- save模式
  - 该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，直到RDB过程完成为止
  - 执行完成时候如果存在老的RDB文件，就把新的替代掉旧的。我们的客户端可能都是几万或者是几十万，这种方式显然不可取

```shell
127.0.0.1:6379> save
OK
```

- bgsave
  - 执行该命令时，Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求
  - 具体操作是Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短。基本上 Redis 内部所有的RDB操作都是采用 bgsave 命令

```shell
127.0.0.1:6379> BGSAVE
Background saving started
```

- 自动触发
  - 自动触发是由我们的配置文件来完成的
- 如何恢复
  - 将rdb文件放到对应文件下，redis启动会自动检查

```shell
127.0.0.1:6379> config get dir
1) "dir"
2) "/root"  ### 这个目录下存在rdb文件，就会恢复
```

- 优点
  - 适合大规模的数据恢复
  - 数据完整性要求不高，（在没有触发save规则的时候宕机，数据就没了）
- 缺点
  - 数据可能丢失（在没有触发save规则的时候宕机，数据就没了）
  - fork进程会占用空间

# 源码解析

- 每个链表都是用adlist.h来表示

```c
typedef struct listNode {
    //上一个节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点值
    void *value;
} listNode;
```

