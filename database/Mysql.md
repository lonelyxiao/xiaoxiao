# 名词

- 数据data
- 数据库database
- 数据库管理系统dbms
  - 目前分为关系型和非关系型
- 实例instance
  - 启动一个数据库叫一个实例



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

select执行顺序

from   ->  where -> select -> group by -> having

### 多表关联

- 笛卡尔积

先将a和b做笛卡尔积，再做where筛选

```mysql
select * from a, b
#或者
select * from a inner join a.id=b.id
```

### 将查询结果导入到文件

My.cnf配置文件中添加secure_file_priv=/tmp/后重启再执行 

```mysql
 SELECT sid,sname,sex INTO OUTFILE '/tmp/students.txt'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM students;
```

## 索引

### 创建索引

```mysql
create index idx_st_sname on students(sname); ##创建普通索引
create index idx_st_union on students(sname,sex); ##创建复合索引
create unique index idx_st_sid on students(sid); ##创建唯一索引
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

  

