# 名词

- 数据data
- 数据库database
- 数据库管理系统dbms
  - 目前分为关系型和非关系型
- 实例instance
  - 启动一个数据库叫一个实例

# Mysql分层

## 结构

![](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210708163932.png)

![image-20210712212602360](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210712212609.png)

## 查询操作

1. 先去缓存区里查找这条sql是否有缓存
2. 如果缓存区没有，则去词法分析器分析这条sql（分析词法是否正确、语法分析）
3. 优化器：分析条件有没有建立索引，分析哪种查询效率高
4. 执行器：真正的执行操作，去调用引擎的接口
5. 数据库引擎去磁盘进行查找数据
6. 返回数据，将数据已key-value形式放入缓存中（key: sql，  value: 数据）

## 分词概念

- Server层:连接器、查询缓存、分析器、优化器、执行器
  - 内置函数:日期、时间、数学、加密函数等
  - 跨存储引擎功能:存储过程、触发器、视图等
- 存储引擎

## 缓存机制

- 缓存适合读多写少的情况，如果需要，需要去my.cnf文件配置query_cache_type参数（0,1,2）
  - 0：
  - 1: 无论哪张表，都会使用缓存
  - 2： 按需缓存。如果：:select SQL_CACHE* from test; 表示当前查询，使用缓存
- mysql> show status like '%Qcache%'; 
  - 获取缓存的数据信息
- 5.7以后版本没有查询缓存这个功能

# 事务

## 事务隔离级别

# mysql 安装

<https://dev.mysql.com/downloads/mysql/>

选择generic

下载：**Linux - Generic (glibc 2.12) (x86, 64-bit), Compressed TAR Archive**

## 常规安装

将mysql安装到它推荐的安装路径中 /usr/local/

```shell
#将其解压
[root@localhost local]# tar -xvf mysql-8.0.13-linux-glibc2.12-x86_64.tar.xz 
#更换文件夹名字
[root@localhost local]# mv mysql-8.0.13-linux-glibc2.12-x86_64 mysql
#建立数据文件夹
[root@localhost mysql]# mkdir data
#创建mysql的用户组和用户
[root@localhost mysql]# groupadd mysql
[root@localhost mysql]# useradd mysql -g mysql
[root@localhost mysql]# chown  mysql:mysql data
#初始化数据库
[root@localhost mysql]# ./bin/mysqld --initialize --user=mysql --datadir /usr/local/mysql/data
##可以看到它有个初始化密码 password is generated for root@localhost: FepopP/LP9aW
#如果忘记初始化密码，可以删除data文件夹，重新初始化mysql
#检查etc下面有没有这个文件，有的话就删除
[root@localhost data]# rm -rf /etc/my.cnf
#启动mysql
[root@localhost mysql]# ./support-files/mysql.server start
[root@localhost mysql]# netstat -an | grep 3306
#配置mysql环境变量
[root@localhost ~]# vim .bash_profile 
[root@localhost ~]# source .bash_profile 
#进入mysql，设置密码
mysql> alter user user() identified by '123456';
```

## 非常规安装

1、数据盘在其他目录下

```shell
#移动数据文件
[root@localhost ~]# mv /usr/local/mysql/data /home
#编辑配置文件
[root@localhost mysql]# vi /etc/my.cnf
#内容
[mysqld]
datadir=/home/data

```

2、mysql安装目录在其他路径下面

- 参数解释
  - --no-defaults：不读取任何选项文件，该选项在mysql_install_db启动时，若因为读取了未知的参数而启动失败时使用

```shell
## 移动安装文件
[root@localhost home]# mv /usr/local/mysql ./install/

##初始化数据库
./mysql/bin/mysqld --no-defaults --initialize --user=mysql --basedir=/home/install/mysql --datadir=/home/install/data --innodb_data_home_dir=/data/newMysql/ibdata --innodb_data_file_path=ibdata3:100M:autoextend



#编辑配置文件，配置mysql安装目录
[mysqld]
basedir=/home/install/mysql
datadir=/home/data

```

## 修改初始化密码

```shell
mysql> alter user user() identified by '123456';
```

## 展示数据库

```mysql
mysql> show databases;
```

## 使用数据库

```mysql
mysql> use test;

```





# mysql授权

## mysql权限级别

1、全局

```mysql
#存储两部分信息
#1、创建的用户信息
#2、全局的权限
mysql> select * from mysql.user;
#查询用户，和运行操作的来源
mysql> select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
#查看root用户在localhost来源拥有哪些权限
mysql> show grants for root@'localhost';

```

2、数据库级别

3、数据库对象级别（表、视图）

## 创建用户

```mysql
#创建一个localhost来源的用户
mysql> create user test@localhost identified by '123456';
#查询用户只有连接的权限
mysql> show grants for test@localhost;
+------------------------------------------+
| Grants for test@localhost                |
+------------------------------------------+
| GRANT USAGE ON *.* TO `test`@`localhost` |
+------------------------------------------+
1 row in set (0.00 sec)

```

## 授权、回收

```mysql
#将所有权限的所有数据库、所有表的权限授予test@localhost
mysql> grant all privileges on *.* to test@localhost;
#回收权限
mysql> revoke all privileges on *.* from test@localhost;
#只给students表查询id字段的权限
mysql> grant select(id) on course.students to test@localhost;
```

## 创建其他ip能访问的user

问：创建这两个用户，那么test登录时，使用的权限是哪一个？

答：两个用户的并集权限

```mysql
mysql> create user test@'192.168.%' identified by '123456';
Query OK, 0 rows affected (0.08 sec)

mysql> create user test@'192.168.1%' identified by '123456';
Query OK, 0 rows affected (0.02 sec)

```

## 权限生效

如果执行insert/update/delete操作上述的系统权限表之后，则必须再执行刷
新权限命令才能同步到系统内存中，刷新权限命令包括：flush
privileges/mysqladmin flush-privileges/mysqladmin reload

```mysql
mysql> set password=password('mysql');
Query OK, 0 rows affected, 1 warning (0.01 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```



## mysql 用户资源

```mysql
##通过执行create user/alter user设置/修改用户的资源限制
mysql> CREATE USER 'francis'@'localhost' IDENTIFIED BY 'frank'
-> WITH MAX_QUERIES_PER_HOUR 20
-> MAX_UPDATES_PER_HOUR 10
-> MAX_CONNECTIONS_PER_HOUR 5
-> MAX_USER_CONNECTIONS 2;
mysql> ALTER USER 'francis'@'localhost' WITH MAX_QUERIES_PER_HOUR 100;
##取消某项资源限制,则把原先的值修改成0（0代表没有限制）
mysql> ALTER USER 'francis'@'localhost' WITH MAX_CONNECTIONS_PER_HOUR 0;
```

## mysql连接

```mysql
#mysql 查询连接密码加密方式，默认的加密方式是caching_sha2_password，navicat无法连接
mysql> select user, host, plugin from mysql.user;
+------------------+------------+-----------------------+
| user             | host       | plugin                |
+------------------+------------+-----------------------+
| test             | 192.168.%  | caching_sha2_password |

## 创建native
mysql> drop user test@'192.168.%';
mysql> create user test@'192.168.%' identified with mysql_native_password by '123456';

```

## MySQL用户lock

通过执行create user/alter user命令中带account lock/unlock子句设
置用户的lock状态
• Create user语句默认的用户是unlock状态
• mysql> create user abc2@localhost identified by 'mysql' account lock;
• Query OK, 0 rows affected (0.01 sec)
• Alter user语句默认不会修改用户的lock/unlock状态
• mysql> alter user 'mysql.sys'@localhost account lock;
• Query OK, 0 rows affected (0.00 sec)
• mysql> alter user 'mysql.sys'@localhost account unlock;
• Query OK, 0 rows affected (0.00 sec)
• 当客户端使用lock状态的用户登录MySQL时，会收到如此报错
Access denied for user 'user_name'@'host_name'.
Account is locked

# 常用sql语句

## 创建表

```mysql
CREATE TABLE `students_copy` (
• `sid` int(11) DEFAULT NULL,
• `sname` varchar(20) DEFAULT NULL,
• `sex` int(11) DEFAULT NULL,
    primary key (sid),# 创建主键
• UNIQUE KEY `idx_st_sid` (`sid`), #创建唯一索引
• KEY `idx_st_union` (`sname`,`sex`) # 创建索引
• )
```



## 复制表结构

```mysql
mysql> create table stu_copy like stu;
mysql> desc stu_copy;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(10)     | NO   | PRI | NULL    |       |
| name  | varchar(50) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+

```

## 也可以查询创建表的sql语句

```mysql
mysql> show create table stu;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                             |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| stu   | CREATE TABLE `stu` (
  `id` int(10) NOT NULL,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

```

## 将表结构复制，并将数据也复制

```mysql
mysql> create table stu_cope1 as select * from stu;
mysql> select * from stu_cope1;
+----+--------+
| id | name   |
+----+--------+
|  1 | 张三   |
+----+--------+
```

## insert

当insert语句中使用on duplicate key update子句时，如果碰到当前
插入的数据违反主键或唯一键的唯一性约束，则Insert会转变成
update语句修改对应的已经存在表中的这条数据。比如如果a字段
有唯一性约束且已经含有1这条记录，则以下两条语句的执行结
果相同
• INSERT INTO table (a,b,c) VALUES (1,2,3)
ON DUPLICATE KEY UPDATE c=c+1;
• UPDATE table SET c=c+1 WHERE a=1;

## update

### 多表修改

修改students.sname的数据 

```mysql
update students,students2
set students.sname=students2.sname,students.gender=students2.gender
where students.sid=students2.sid;
```

```mysql
##执行失败不会停止但数据不会修改
Update ignore students set sid=1 where sid=2; 

```

## delete



## select

> select执行顺序

from   ->  where -> select -> group by -> having

- on 早于 where 执行

> 多表关联

- 笛卡尔积

先将a和b做笛卡尔积，再做where筛选

```mysql
select * from a, b
#或者
select * from a inner join a.id=b.id
```

> 将查询结果导入到文件

My.cnf配置文件中添加secure_file_priv=/tmp/后重启再执行 

```mysql
 SELECT sid,sname,sex INTO OUTFILE '/tmp/students.txt'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM students;
```

## 表重命名

```mysql
RENAME TABLE old_table TO tmp_table
```

## truncate table语句 

truncate table语句用来删除/截断表里的所有数据 , (不带条件的全部删除)

和delete删除所有表数据在逻辑上含义相同，但性能更快 

```mysql
truncate table students_test;
```

## 修改表

```mysql
 ##新增字段
 Alter table … add [column_name]
 ## 新增索引
 Alter table … add constraint [name] unique [index/key] [name]
 #新增外键
 Alter table … add constraint [name] foreign key (column_name) references table_name(column_name)
 #删除表字段
Alter table … drop column [column_name]
#删除索引
Alter table … drop [index/key] [index_name]
```

## 常用函数

-   Isnull(expr)操作符  

  如果expr是null，则返回1，否则返回0  

```mysql
mysql> select ISNULL(NULL);
+--------------+
| ISNULL(NULL) |
+--------------+
|            1 |
+--------------+
mysql> select ISNULL(1);
+-----------+
| ISNULL(1) |
+-----------+
|         0 |
+-----------+
```

###   流程控制函数  

-  当value等于compare_value时，则返回result，否则返回else里的result，如果没有else子句则返回null  

  ```mysql
  CASE value WHEN [compare_value] THEN result [WHEN][compare_value] THEN result ...] [ELSE result] END  
  ```

-  当第一个condition满足时，则返回result，否则返回else里的result，如果没有else子句时则返回null  

  ```mysql
  CASE WHEN [condition] THEN result [WHEN [condition] THEN result ...] [ELSE result] END
  ```

-   当expr1为1/true时，则返回expr2，否则返回expr3  

  ```mysql
  IF(expr1,expr2,expr3)
  ```

-   当expr1为非null时，则返回expr1，否则返回expr2  

  ```mysql
  IFNULL(expr1,expr2)
  ```

-   当expr1等于expr2时，则返回null，否则返回expr1  

  ```mysql
  NULLIF(expr1,expr2)
  ```

### 字符串函数

-   返回字符串的字符长度  

  ```mysql
  CHAR_LENGTH(str)
  ```

-   返回括号里所有参数字符串连接在一起，当其中有参数NULL时则返回NULL  

  ```mysql
  CONCAT(str1,str2,...)
  ```

-   返回以第一个参数为分隔符的连接后的一个字符串，当有参数为NULL时则null被忽略  

  ```mysql
  CONCAT_WS(separator,str1,str2,...)
  ```

-   将str中从pos位置开始后的len个字符替换成newstr字符串  

  ```mysql
  INSERT(str,pos,len,newstr)
  ```

-   返回str字符串中第一个出现substr字符串的位置  

  ```mysql
  INSTR(str,substr)
  ```

# 事务

## 事务概念  

> 原子性（ atomicity)  

原子性的体现在于回滚，innodb操作会产生undo log 文件，记录sql的数据信息

1. 一效性 （ consistency)  
   1. 是数据库追求的最终目标，也就是说数据操作后的状态要是合法的
2. 隔离性（ isolation)  
   1. 写-写：锁方式
   2. 写-读：MVCC

> 持久性（ durability)  

- 指的是事务一旦提交，对数据库的改变就是永久性的

- mysql在查询过程中，会去先查询buffer(一部分数据的缓冲)， 修改数据时，有些数据也会先写入缓冲，再由缓冲写入磁盘io

  - 问题：如果缓冲写入io时，mysql挂了，则数据就可能丢了
  - 这时就引入了redo log概念

  > > redo log

  - 修改数据时，有些数据也会先写入缓冲, 并且写入redo log中，再由缓冲写入磁盘io
  - redo log 的数据是append的方式，磁盘也是顺序写入的，所以快

  

## 隔离级别  

> READ UNCOMMITIED （未提交读）  

- 可能出现脏读的情况

```tex
在 Read UNCOMMITIED 级别，事务中的修改，即使没有提交，对其他事务也都是可见的。
事务可以读取未提交的数据，这也被称为脏读（Dirty Read ）。这个级别会导致很多问题，
从性能上来说， READ UNCOMMITIED 不会比其他的级别好太多，但却缺乏
其他级别的很多好处，除非真的有非常必要的理由，在实际应用中一般很少使用。
```

> READ COMMITIED （提交读）

- 避免了脏读，但是可能出现不可重复读(在同一个事务中执行相同的sql语句，查询到的结果是不同的)
- 读的数据是最新版本

```tex
大多数数据库系统的默认隔离级别都是 READ COMMITTED （但Mysql不是）。
一个事务开始时 ，只能“看见”已经提交的事务所做的修改。
该隔离级别避免了脏读，但是可能出现不可重复读。事务A事先读取了数据，
事务B紧接着更新了数据，并提交了事务，而事务A再次读取该数据时，数据已经发生了改变
```

> Repeatable read(可重复读取)

- 这样避免了不可重复读和脏读
- 但是有时可能会出现幻读（第一次读该行数据不存在，第二次读数据已存在）
- 可重复读主要是针对已经存在的数据(读的数据是事务开启前的版本)

```tex
在一个事务内，多次读同一个数据，在这个事务还没结束时，
其他事务不能访问该数据(包括了读写)，
这样就可以在同一个事务内两次读到的数据是一样的
```

**不可重复读的重点是修改; 幻读的重点在于新增或者删除**

> Serializable(可串行化 )

```tex
提供严格的事务隔离，它要求事务序列化执行，事务只能一个接着一个地执行，但不能并发执行
```

# 日志模块

## bin log(逻辑日志)

> 概念

- 二进制日志
- 在服务层实现的功能（引擎共用）
- Binlog为逻辑日志,记录的是一条语句的原始逻辑
- Binlog不限大小,追加写入,不会覆盖以前的日志

> 查看是否开启

```sql
### log_bin=on 标识开启了
mysql> show variables like '%log_bin%';
+---------------------------------+--------------------------------+
| Variable_name                   | Value                          |
+---------------------------------+--------------------------------+
| log_bin                         | ON                             |
| log_bin_basename                | /var/lib/mysql/mysql_bin       |
| log_bin_index                   | /var/lib/mysql/mysql_bin.index |
| log_bin_trust_function_creators | OFF                            |
| log_bin_use_v1_row_events       | OFF                            |
| sql_log_bin                     | ON                             |
+---------------------------------+--------------------------------+
```

> 配置开启

在my.conf配置

```conf
[mysqld]
server-id=123454
log_bin=mysql_bin
binlog_format=ROW
expire_logs_days=30
```

## redo log

- 记录InnoDB存储引擎的事务日志
- mysql 先写日志再写磁盘（当mysql不忙的时候，将日志数据写入磁盘文件中）

![image-20210712223827118](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210712223827.png)

> Redo Log写入原理

- 以循环方式写入日志文件,不断的写与擦除,文件1写满时，切换到文件2，文件2写满时，再次切换到文件1。
- writepos当前记录的位置,循环边写边后移
- checkpoint当前要擦除的位置,循环边写边后移
- 当擦除的时候，先会操作阻塞，将日志写入磁盘表里面的结构里

![image-20210712224508580](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210712224508.png)

> Flush log 原理

- os buffer： 内核态的内存
- innodb_flush_log_at_trx_commit：默认是1， （配置值： 0,1,2）

![image-20210712224838371](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210712224838.png)

> 日志刷盘规则

默认情况下事务每次提交的时候都会刷事务日志到磁盘中，这是因为变量 innodb_flush_log_at_trx_commit 的值为1。但是innodb不仅仅只会在有commit动作后才会刷日志到磁盘，这只是innodb存储引擎刷日志的规则之一。

1.发出commit动作时。已经说明过，commit发出后是否刷日志由变量 innodb_flush_log_at_trx_commit 控制。

2.每秒刷一次。这个刷日志的频率由变量 innodb_flush_log_at_timeout 值决定，默认是1秒。要注意，这个刷日志频率和commit动作无关。

3.当log buffer中已经使用的内存超过一半时。

4.当有checkpoint时，checkpoint在一定程度上代表了刷到磁盘时日志所处的LSN位置。

# 索引

## 分类

InnoDB

- 主键索引
  - 设定为主键后数据库会自动建立索引,innodb为聚簇索引
- 单值索引
  - 即一个素引只包含单个列,一个表可以有多个单列索引
- 唯一索引
  - 索引列的值必须唯一，但允许有空值（只能存在一个null）
- 复合索引
  - 即一个素引包含多个列
  - 常用于多个值查询

MYISAM

- 全文索引（FULL TEXT）
  - 全文索引类型为FULLTEXT，在定义索引的列上支持值的全文查找，允许在这些索引列中插入重复值和空值。全文索引可以在CHAR、VARCHAR,TEXT上创建

## 创建

- 查看一个表的索引

```mysql
show index from t_user;
```

- 建表后创建

```mysql
create index idx_st_sname on students(sname); ##创建普通索引
create index idx_st_union on students(sname,sex); ##创建复合索引
create unique index idx_st_sid on students(sid); ##创建唯一索引
```

- 建表前创建

```mysql
create table t_user (id int primary key, name varchar(50), key(name));

create table t_user (id int primary key, name varchar(50), unique(name));
```

## 索引原理

- innodb采用的是B+树
- 如果查普通索引，先会去普通索引上查（普通索引的data存储的是主键id）, 查到主键id后，再去主键索引（聚合索引）上查询数据

### B-树

- 非叶子节点也存储数据

![](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210702112453.jpg)

### B+树理解

- 是*二叉查找树的改良版本*

![](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210702111418.jpg)

- 每页相当于上图的一个磁盘块
- InnoDB存储引擎中页的大小为16KB（操作系统的预读一页是4K，所以存储引擎的页大小应该是4k的整数倍），一般表的主键类型为INT（占用4个字节）或BIGINT（占用8个字节），指针类型也一般为4或8个字节，也就是说一个页（B+Tree中的一个节点）中大概存储16KB/(8B+8B)=1K个键值（因为是估值，为方便计算，这里的K取值为〖10〗^3）。也就是说一个深度为3的B+Tree索引可以维护10^3 * 10^3 * 10^3 = 10亿 条记录。

### 为什么要选择B+

- 磁盘预读：每次读取数据，磁盘都是将一块数据进行返回（也称为页），所以将数据放到节点会使得一页存储的数据减少（我们每次比对，只需要比对key）
- 如果将数据存储在非叶子节点，那么会造成树的层级增加，增加了磁盘IO

### B+与B-树的区别

- 非叶子节点只存储键值信息。
- 所有叶子节点之间都有一个链指针。
- 数据记录都存放在叶子节点中。

### 聚簇索引与非聚簇索引

- 聚集索引与非聚集索引的区别是：叶节点是否存放一整行记录
- InnoDB 主键使用的是聚簇索引和辅助索引，MyISAM 不管是主键索引，还是二级索引使用的都是非聚簇索引。
  下图形象说明了聚簇索引表(InnoDB)和非聚簇索引(MyISAM)的区别：

![](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210702113122.png)

innodb中：

- InnoDB使用的是聚簇索引，将主键组织到一棵B+树中，而行数据就储存在叶子节点上，若使用"where id= 14"这样的条件查找主键，则按照B+树的检索算法即可查找到对应的叶节点，之后获得行数据。
- 若对Name列进行条件搜索，则需要两个步骤:第一步在辅助索引B+树中检索Name，到达其叶子节点获取对应的主键。第二步使用主糖在主索引B+树种再执行一次B+树检索操作，最终到达叶子节点即可获取整行数据。(重点在于通过其他键需要建立辅助索引)
- 聚簇索引默认是主键，如果表中没有定义主键，InnoDB会选择一个唯一且非空的索引代替。如果没有这样的索引，InnoDB会隐式定义一个主键(类似oracle中的Rowld)来作为聚簇索引。如果已经设置了主键为聚簇索引又希望再单独设置聚簇索引，必须先刑除主键，然后添加我们想要的聚簇索引，最后恢复设置主键即可。

MYISAM中:

- 索引和数据是分开的

## 为什么主键通常建议使用自增id

聚簇索引的据的物理存放你序与索引顺序是一致的，即:只要索引是相邻的，那么对应的擞据一定也是相邻地存放在磁盘上的。如果主键不是自增id，那么可以想象，它会干些什么，**不断地调整数据的物理地址、分页**，当然也有其他一些措施来减少这些操作，但却无法彻底避免。但，如果是自增的，那就简单了，它只需要一页一页地写，索引结构相对紧凑，磁盘碎片均效率也高。

## 聚簇索引一定会是主键？

在 InnoDB 中，聚集索引不一定是主键，但是主键一定是聚集索引：原因是如果没有定义主键，聚集索引可能是第一个不允许为 null 的唯一索引，如果也没有这样的唯一索引，InnoDB 会选择内置 6 字节长的 ROWID 作为隐含的聚集索引。

## 什么情况下无法利用索引呢?

- 在查询语句中使用LIKE关键字进行查询时，如果匹配字符串的第一个字符为"%”，索引不会被使用。如果"%"不是在第一个位围，索引就会被使用。
  - 如果使用两个%% 可以使用覆盖索引，如： select phone from user where phone like '%56456%'

> 多列索引是在表的多个字段上创建一个索引，只有查询条件中使用了这些字段中的第一个字段，索引才会被使用。
- 如 索引是：name age sex  , 查询的时候 使用了name 就可以使用索引（最左原则）
- 如： age sex 无法使用索引，  age sex name  可以使用

> > 原理

左边的key是有序的排列在树结构上的

右边的只有在a相等的情况下，b才有序的

![image-20210708102059747](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210708102059.png)

- 查询语句只有OR关键字时，如果OR前后的两个条件的列都是索引，那么查询中将使用索引。**如果OR前后有一个条件的列不是索引，那么查询中将不使用索引**。
- 多列索引，左边使用了范围查询 如：   where  name=''  and age>10 and sex=0  (age索引在sex索引左边)
- 计算，如: +、-、*、/、!=、<>、is null、 is not null、or
- 函数，如: sum(）、 round(）等等
- 手动/自动类型转换，如: id = "1"，本来是数字，给写成字符串了

## hash索引

- mysql 的memory引擎使用hash索引
- InnoDB 引擎有一个特殊的功能叫做“自适应哈希索引。当InnoDB注意到某些索引值被使用得非常频繁时 ，它会在内存中基于 B - Tree 索引之上再创建一个哈希索引，这样就让 B-Tree 索引 也具有哈希索引的一些优点， 比如快速的哈希查找。这是一个完全自动的、内部的行为，用户无战控制或者配置，不过如果有必要，完全可以关闭该功能。

> 为什么不选择hash作为innodb索引

1. hash会有碰撞问题，比如说：很多值计算hash之后，都占用了一个index，那么会导致这个index的链表过长
2. 利用hash存储的话需要将所有的数据文件添加到内存，比较耗费内存空间
3. hash更加适合等值计算

## 索引下推

执行下列sql

```sql
SELECT * from user where  name like '陈%' and age=20
```

> 没有索引下推：

会忽略age这个字段，直接通过name进行查询，然后拿着取到的id值一次次的回表查询

> 有索引下推：

InnoDB并没有忽略age这个字段，而是在索引内部就判断了age是否等于20，对于不等于20的记录直接跳过，因此在(name,age)这棵索引树中只匹配到了一个记录，此时拿着这个id去主键索引树中回表查询全部数据，这个过程只需要回表一次。

## MRR

- 全称「Multi-Range Read Optimization」
- MRR 通过把「随机磁盘读」，转化为「顺序磁盘读」，从而提高了索引查询的性能

> 使用MRR

```sql
mysql > set optimizer_switch='mrr=on';
Query OK, 0 rows affected (0.06 sec)

mysql > explain select * from stu where age between 10 and 20;
+----+-------------+-------+-------+------+---------+------+------+----------------+
| id | select_type | table | type  | key  | key_len | ref  | rows | Extra          |
+----+-------------+-------+-------+------+---------+------+------+----------------+
|  1 | SIMPLE      | tbl   | range | age  |    5    | NULL |  960 | ...; Using MRR |
+----+-------------+-------+-------+------+---------+------+------+----------------+
```

- 我们开启了 MRR，重新执行 sql 语句，发现 Extra 里多了一个「Using MRR」

**对于 Myisam，在去磁盘获取完整数据之前，会先按照 rowid 排好序，再去顺序的读取磁盘。**

**对于 Innodb，则会按照聚簇索引键值排好序，再顺序的读取聚簇索引。**

> 顺序读带来了几个好处

1. 磁盘和磁头不再需要来回做机械运动；
2. 可以充分利用磁盘预读

btree索引的存储是有序的，所以访问索引是顺序io，而通过索引访问数据时确实是随机的

# MVCC

多版本并发控制，保证读和写之间没有锁竞争

> ACID

- 原子性（atomicity)： undo log实现
- 一致性（consistency)：
- 隔离性（isolation）：加锁和MVCC实现
- 持久性（durability）： redo log

> 读已提交

读已提交/重复读，都是基于mvcc实现的

> MVCC概念

- undo log
  - 如，下图，每行数据都有事务id，回滚指针
  - undo log存储我们的历史数据
  - roll point 指向上个版本的undo log,undo log 里面的roll point 又指向他的前一个版本数据

![image-20210708091728051](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210708091735.png)

- 版本链
- read view
  - 我们知道在版本链中，去查询哪个版本的数据
  - 他在代码中就是一个对象

m_ids: 表示在生成`ReadView`时当前系统中**活跃的读写事务**的`事务id`列表(事务未提交id)

min_trx_id： 表示在生成`ReadView`时当前系统中活跃的读写事务中最小的`事务id，也就是`m ids中的最小值

max_trx_id: 表示生成`ReadView`时系统中应该分配给下一个事务的`id`值

creator_trx_id:表示生成该`Readview`的事务的`事务id

> ReadView如何判断版本链中的哪个版本可用?

trx_id == creator_trx_id: 可以访问这个版本（当前数据的创建者）

trx_id  < min_trx_id:  可以访问这个版本（表示trx_id这个数据已经提交）

trx_id  > max_trx_id: 不可以访问这个版本（表示trx_id数据未提交）

min_trx id <= trx_id <= max _trx_id:如果trx_id在m ids中是不可以访问这个版本的，反之可以

- 读已提交：每一个select都会生成一个readview
- 可重复读：生成的readview是以事务为单位

> 解决可重复读的幻读问题

- 快照读的解决方式
  - 读取专门的快照 (对于RC，快照（ReadView）会在每个语句中创建。对于RR，快照是在事务启动时创建的)
  - 每个事务都只生成一个readview，外界再怎么修改数据，readview没有变化
- 间隙锁（mysql默认）
  - 锁的就是两个值之间的空隙。在一行行扫描的过程中，不仅将给行加上了行锁，还给行两边的空隙，也加上了间隙锁

# InnoDb和MyIsam区别

- InnoDB支持事务，MyISAM不支持事务。这是MSQL将默认存储引擎从MylSAM变成InnoDB的重要原因之一。
- InnoDB支持外键，而MylSAM不支持。对一个包含外键的InnoDB表转为MYISAM会失败。
- InnoDB是聚集（聚簇)索引，plylSAM是非聚集(非聚簇）索引.
- MylSAM支持FULLTEXT类型的全文索引,
- lnnoDB最小的锁粒度是行锁，MylSAM最小的锁粒度是表锁。

# 锁的类型

> 表锁和行锁

mysql  如果以索引为条件进行操作的话，则是行锁

如果**索引失效**,则会由行锁上升到表锁

> 间隙锁

如果我在索引列插入： 1,3,5,6,7这些数据

然后我们查找范围： where  id>=1  and id<=7

这个时候，数据库会将 1-7的数据锁上，这时我们插入id=2的数据是插入不了的，因为已经产生了间隙锁

> 获取数据库锁的次数信息

```sql
show status like "innodb_row_locks%";
```

# 性能分析Explain

- 我们常常用到explain这个命令来查看一个这些SQL语句的执行计划

```sql
-- 实际SQL，查找用户名为Jefabc的员工
select * from emp where name = 'Jefabc';
-- 查看SQL是否使用索引，前面加上explain即可
explain select * from emp where name = 'Jefabc';
```



> 描述

id:选择标识符
select_type:表示查询的类型。
table:输出结果集的表
partitions:匹配的分区
type:表示表的连接类型
possible_keys:表示查询时，可能使用的索引
key:表示实际使用的索引
key_len:索引字段的长度
ref:列与索引的比较
rows:扫描出的行数(估算的行数)
filtered:按表条件过滤的行百分比
Extra:执行情况的描述和说明

