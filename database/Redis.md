# 安装

下载、解压、编译

```shell
wget http://download.redis.io/releases/redis-2.8.18.tar.gz    
tar xzf redis-2.8.18.tar.gz     
cd redis-2.8.１８    
make    
```

(注：执行make时可能会出现的错误:1.未安装gcc，请先: yum install gcc tcl -y；2.安装报错 error: jemalloc/jemalloc.h: No such file or directory；解决方案：make 换==》make MALLOC = libc)

```shell
# 测试启动
cd /usr/local/redis    
./redis-server redis.conf
```

# 基于字符串

在SET命令中，有很多选项可用来修改命令的行为。 以下是SET命令可用选项的基本语法。

```shell
redis 127.0.0.1:6379> SET KEY VALUE [EX seconds] [PX milliseconds] [NX|XX]
```

- EX seconds − 设置指定的到期时间(以秒为单位)。

- PX milliseconds - 设置指定的到期时间(以毫秒为单位)。
- NX - 仅在键不存在时设置键。
- XX - 只有在键已存在时才设置。

## Mset

键命名规范：通常，公司用：：来代表一个级别

```shell
redis 127.0.0.1:6379> MSET key1 value1 key2 value2 .. keyN valueN
redis 127.0.0.1:6379> MSET key1 "Hello" key2 "World" 
OK 
redis 127.0.0.1:6379> GET key1 
"Hello" 
redis 127.0.0.1:6379> GET key2 
1) "World"

```

## SETRANGE

```shell
redis 127.0.0.1:6379> SET key1 "Hello World" 
OK 
redis 127.0.0.1:6379> SETRANGE key1 6 "Redis" 
(integer) 11 
redis 127.0.0.1:6379> GET key1 
"Hello Redis"
```

## 其他

redis key 值是二进制安全的，这意味着可以用任何二进制序列作为key值

key取值原则

- 键值不需要过长，会消耗内存
- 键值不宜太短，可读性差

一个字符类型的值最多能存512M字节的内容

## 清空所有键

Flushdb

## 位操作

设置某一位上的值

offset: 偏移量，从0开始

```shell
setbit key offset value
```

获取某一位上的值

```shell
getbit key offset
```

返回指定值在区间第一次出现的位置

```shell
bitpos key bit [start] [end]
```

## 位运算

对一个或者多个保存二进制位的字符串key进行位元操作，并将结果保存到destkey中

operation 可以时and（与）、or（或）、not（异或）、xor（逻辑非） 这四种操作

bitop operation destkey key [key...]

bitop not destkey key 对给定的key进行逻辑非

bitop处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看做0

## 场景

网站用户的上线次数统计（活跃用户）

用户id作为key，天作为offset，上线置为1

例如：id为500的用户，今年第1天上线、第30天上线

setbit u500 1 1

setbit u500 30 1

bitcount u500

# 基于LINKLIST

元素时字符串类型，列表头部和尾部增删快、元素可以重复，最多存2^32-1个元素

索引：从左至右，从0开始

​			从右至左，从-1开始

```shell
#从右边插入一个或者多个值
rpush key value1...valuen

redis 127.0.0.1:6379> RPUSH mylist "hello"
(integer) 1
redis 127.0.0.1:6379> RPUSH mylist "foo"
(integer) 2
redis 127.0.0.1:6379> RPUSH mylist "bar"
(integer) 3
##移除列表的最后一个元素，并将该元素添加到另一个列表并返回
redis 127.0.0.1:6379> RPOPLPUSH mylist myotherlist
"bar"
redis 127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello"
2) "foo"

```

**Redis Lrem 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。**

COUNT 的值可以是以下几种：

- count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。

- count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。

- count = 0 : 移除表中所有与 VALUE 相等的值。

```shell
LREM KEY_NAME COUNT VALUE
```

**命令用于在列表的元素前或者后插入元素**

LINSERT KEY_NAME BEFORE EXISTING_VALUE NEW_VALUE 

**命令移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止**

BLPOP LIST1 LIST2 .. LISTN TIMEOUT

# hash散列

由field和关联的value组成的map键值对

field和value是字符串类型

一个hash中最多包含2^32-1个键值对

set key field value

不适用hash的情况

- 使用二进制位操作命令
- 使用过期键功能

# 基于set集合

无序的、去重的、元素时字符串类型

```shell
# 添加一个或者多个成员
SADD key member1 [member2]
# 返回集合中的所有成员
smembers key
# 判断 member 元素是否是集合 key 的成员
sismember key member
# 返回集合中一个或多个随机数
srandmember key [count]
# 获取集合的成员数
scard key
# 移除并返回集合中的一个随机元素
spop key
# 返回给定所有集合的差集
sdiff key1 key2

```

## 场景

求交集：贴吧的共同关注