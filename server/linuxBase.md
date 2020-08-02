

# shell上传下载

```shell
## 安装命令
[root@localhost home]# yum install -y lrzsz
#上传
[root@localhost home]# rz
# 下载
[root@localhost home]# sz 3.pdf
```

# 概念

- linux不区分扩展名，它靠权限来区分文件


# linux 目录结构

## 一级目录

| 目录   | 目录作用                                                     |
| ------ | ------------------------------------------------------------ |
| /bin/  | 存放系统命令的目录，普通用户和超级用户都可以访问，是/usr/bin/目录的软连接 |
| /sbin/ | 存放系统命令的目录，只有**超级用户**才可以执行。是/usr/sbin/目录的软连接 |
| /boot/ | 系统的启动目录，保存于系统启动相关的文件                     |
| /dev/  | 设备文件保存位置                                             |
| /etc/  | 配置文件保存位置。系统内所有采用默认安装方式（rpm）的服务配置文件保存在此目录，如用户信息、启动脚本等 |
| /home/ | 普通用会在这个目录下建立一个家目录，在这个家目录下拥有超级权限，登录后，就是在家木箱 |
| /media | 挂载目录                                                     |

## 二级目录

| 目录             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| /user/local      | 一般建议做安装目录                                           |
| /user/local/src  | 一般建议放源程序                                             |
| /usr/src/kernels | 内核源码目录                                                 |
| /var/log         | 系统日志                                                     |
| /var/www/html    | rpm包安装的apache的网页主目录                                |
| /var/lib         | 程序运行的数据保存目录，如mysql数据默认保存在/var/lib/mysql/目录中 |
| /var/run/        | 服务运行的pid保存位置                                        |
| /var/spool/cron  | 定时任务目录                                                 |

# 服务器建议

- 服务器只能重启，不允许关机
  - 重启前，终止正在执行的服务（否则可能造成硬盘损坏）
  - 重启前，建议对外的网络断掉
  - 重启前，建议指向sync命令
  - 重启命令建议用：shutdown -r now

# 常见命令

## ls

```shell
[root@localhost home]# ls --help
Usage: ls [OPTION]... [FILE]...
-a 显示所有文件
-h 按照习惯显示文件大小，默认全是byte

```

## mkdir

```shell
## 递归建立目录
[root@localhost home]# mkdir -p a/b/c/d
```

## stat查看文件详情

```shell
[root@localhost home]# stat derby.log 
  File: ‘derby.log’
  Size: 686       	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 50803096    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:home_root_t:s0
## 最后访问时间
Access: 2020-03-27 11:49:13.739673217 -0400
## 最后数据修改时间
Modify: 2020-04-03 21:21:19.130301270 -0400
## 最后修改状态时间，如权限等
Change: 2020-04-03 21:21:19.130301270 -0400
 Birth: -
```

## cat

```shell
## 显示行号
cat -n
```

## less 分行显示

向上箭头 ：向上走

## hard显示文件头

和tail对应

## ln链接

### 硬链接

建立一个硬链接，查看，发现两个文件的**i节点**是一样的

```shell
[root@localhost ~]# echo aaaa >> a
[root@localhost ~]# ln a /home/a_a
[root@localhost ~]# ls -ild a /home/a_a
33574992 -rw-r--r--. 3 root root 5 May 30 11:33 a
33574992 -rw-r--r--. 3 root root 5 May 30 11:33 /home/a_a
```

硬链接特点

- 源文件和链接文件inode和block相同
- 任意一个修改，另一个也会修改
- 删除一个，另一个还能使用
- 硬链接不能夸分区

原理：因为他们的inode 相同

**不建议使用，因为他的标记不清楚，它不能链接目录**

### 软链接

建立软连接，发现他们的inode不一样。 **骚年，记得用绝对路径**

```shell
[root@localhost ~]# echo ddd d
ddd d
[root@localhost ~]# echo ddd >> d
[root@localhost ~]# ln -s /root/d /home/d_s
[root@localhost ~]# ls -il /root/d /home/d_s
53234855 lrwxrwxrwx. 1 root root 7 May 30 11:54 /home/d_s -> /root/d
33575679 -rw-r--r--. 1 root root 4 May 30 11:54 /root/d
```

特性

- 任意一个修改，另一个也会修改
- 删除源文件，软链接失效
- 不论源文件多大，软链接不变
- 软连接可以链目录

原理：软链接的block里面，存放的是源文件的inode

## 查看帮助文档

- man
- info

## 看看系统中的别名

```shell
[root@localhost home]# alias
```

## 文件搜索

### locate

- 优点，查询的是数据库，速度快，数据库位置：/var/lib/mlocate/mlocate.db
- 缺点： 只能安装文件名搜，不能安装时间，权限等

```shell
##安装
[root@localhost yum.repos.d]# yum  -y install mlocate
##更新数据库
[root@localhost yum.repos.d]# updatedb
##搜索
[root@localhost yum.repos.d]# locate a_a
```

### find

- 按文件名搜

```shell
## 安装文件名搜。 这个搜索范围越大，性能越消耗大
[root@localhost mlocate]# find / -name a_a
/home/a_a
## 忽略大小写搜索
[root@localhost mlocate]# find / -iname a_a

```

- 按大小搜索

```shell
## 搜索比29M大的
[root@localhost home]# find . -size +29M
## 搜索比29M小的
[root@localhost home]# find . -size -29M
```

- 按照权限搜索

```shell
[root@localhost home]# find . -perm 755
```

- 逻辑运算查询
  - -a :&&
  - -o: ||

```shell
#查询size=10m&&type=文件
[root@localhost home]# find . -size 10M -a -type f

```

- 将查询到的结果放入命令中
  - -exec
  - -ok

## grep 

### 搜索文件里面的内容

grep是使用正则表达式匹配，find是使用通配符匹配

所以grep是包含匹配的

搜索a_a文件夹中的内容

-n : 显示行号

```shell
[root@localhost home]# grep -n 'a' a_a
1:aaaa
```

反向查找，不存在a

```shell
[root@localhost home]# grep -v 'a' a_a
```

## 命令别名

alias 别名=“原命令”

alias： 查询别名

想要永久生效，编辑 ~/.bashrc

## 压缩

- zip

```shell
## 压缩
[root@localhost ~]# zip a.zip a
# 将a.zip文件解压到tmp文件夹中
[root@localhost ~]# unzip -d tmp a.zip 
Archive:  a.zip
 extracting: tmp/a 
```

- gzip（不会打包）

gzip不会打包，也就是压缩文件夹，他会将里面的文件一个个压缩

```shell
#将b文件压缩成b.gz文件（会删除原文件）
[root@localhost ~]# gzip b
# 解压
[root@localhost ~]# gzip -d b.gz
```

- bzip2

不能压缩文件夹，算法比gzip更快，压缩比更好

-d:解压

-k:压缩时，保留源文件

-v:显示压缩过程

- tar （不会压缩）

打包命令，不压缩

```shell
# -c: 打包 -v 显示打包过程 -f 指定压缩包的文件名 -x 解压缩
[root@localhost ~]# tar -cvf acdf.tar a b c f
```

打包和压缩

```shell
# 将当前文件打包压缩是啥        
[root@localhost ~]# tar -zcvf a.tar.gz ./*
```

解压

```shell
# 解压到tmp文件夹     
[root@localhost ~]# tar -zxvf a.tar.gz /tmp/
```

只查看不解压

```shell
[root@localhost home]# tar -ztvf b.tar.gz
-rw-r--r-- root/root         8 2020-05-30 11:50 b

```

解压到指定目录

```shell
[root@localhost home]# tar -zxvf b.tar.gz -C /tmp/
b

```

## 刷新文件缓存区

一开始，文件可能会写入内存中，没写入硬盘

重启时使用

```shell
[root@localhost home]# sync
```

## 系统痕迹命令

### w命令

当前正在登录的命令

当前系统1 2 5 分钟前的负载，一般超过核心数高负载

load average: 0.02, 0.02, 0.05

WHAT：当前正在干嘛

```shell
[root@localhost mail]# w
 12:39:09 up 18 min,  1 user,  load average: 0.02, 0.02, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.1.105    12:26    5.00s  0.04s  0.00s w

```

### last命令

系统登录过用户

```shell
[root@localhost mail]# last
root     pts/0        192.168.1.105    Sat Jun  6 12:26   still logged in   
reboot   system boot  3.10.0-957.el7.x Sat Jun  6 12:20 - 12:43  (00:22)
```

### 所有用户

所有用户以及登录时间

```shell
[root@localhost mail]# lastlog
Username         Port     From             Latest
root             pts/0    192.168.1.105    Sat Jun  6 12:26:53 -0400 2020
```

### 登陆被拒绝用户

```shell
[root@localhost mail]# lastb

btmp begins Sat Jun  6 12:20:56 2020

```



# 权限

linux靠权限来区分文件类型

“-” ： 普通文件

“d” ： 目录文件

"l" ： 软链接

"s" ： 套接字文件，**不能删**

```shell
-rw-r--r--.  1 root root  33958849 Mar 31 11:29 seata-server-1.0.0.tar.gz
drwxr-xr-x.  3 root root        20 Mar 26 09:03 work
lrwxrwxrwx.  1 root root         1 May 30 11:46 b_b -> b
```

权限位置

文件类型， 所属用户 u（rwx），所属组 g（rwx）， 其他人 o（rwx）

## 基本权限

### 修改权限

chmod 

- 使用字母赋值权限

```shell
## 给所属人，所属组添加可执行权限
[root@localhost home]# chmod u+x,g+x a_a
[root@localhost home]# ls -l a_a
-rwxr-xr-x. 3 root root 5 May 30 11:33 a_a
## 减少权限
[root@localhost home]# chmod u-x a_a

```

- 使用数字赋值权限

4+2+1（读，写， 执行）

基本数字类型

644： 文件的基本权限

755: 文件的执行权限和目录的基本权限

```shell
## 给文件添加 可读权限
[root@localhost home]# chmod 444 a_a
[root@localhost home]# ls -l a_a 
-r--r--r--. 3 root root 5 May 30 11:33 a_a

```

### 修改所属用户：所属组

修改所属用户

```shell
[root@localhost home]# chown user1 a_a
[root@localhost home]# ls -l a_a
-r--r--r--. 3 user1 root 5 May 30 11:33 a_a
```

修改所属组

```shell
[root@localhost home]# chgrp user1 a_a
[root@localhost home]# ls -l a_a
-r--r--r--. 3 user1 user1 5 May 30 11:33 a_a

```

修改所属用户和所属组

```shell
[root@localhost home]# chown root:root a_a
[root@localhost home]# ls -l a_a
-r--r--r--. 3 root root 5 May 30 11:33 a_a

```

**普通用户能修改所有者是自己的的文件权限**

## 权限的作用

- r
  - 可读
- w:
  - 可以修改文件
  - 如果想**删除文件，需要对文件的上级目录拥有写权限**
- x
  - 可执行

**目录的权限只能是：0、 5、 7**

## 系统默认权限umask、

新建文件最大权限默认为 666 ，没有执行权限

目录默认权限最大 777

```shell

[root@localhost home]# umask
0022

##
[root@localhost home]# umask -S
u=rwx,g=rx,o=rx

```



## 用户

### 新建用户

```shell
##添加用户
[root@localhost home]# useradd user1
## 给用户添加密码
[root@localhost home]# passwd user1

```

## 查看当前登录用户

```shell
[root@localhost ~]# w
 12:33:51 up 13 min,  1 user,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.1.105    12:26    7.00s  0.04s  0.01s w

```

## ACL权限管理

### 概述

为了解决文档对身份不足的问题

**一个文件只能有一个所属用户和所属组**

### 开启ACL

centos7 xfs文件系统貌似已经强制开启ACL权限了

### 设置ACL权限

```shell
#新建文件夹
[root@localhost data]# mkdir test
#只给用户user1给读和执行的权限
[root@localhost data]# setfacl -m u:user1:5 test/
##查看发现user1已经有了对应权限
[root@localhost data]# getfacl test
# file: test
# owner: root
# group: root
user::rwx
user:user1:r-x
group::r-x
mask::r-x
other::r-x

## 递归加上ACL权限
[root@localhost data]# setfacl -m u:user1:5 -R test/

## 加个d, 给目录加个ACL权限的模板，以后在这个目录下新建的文件都有ACL权限
[root@localhost data]# setfacl -m d:u:user1:5 -R test/
```

```shell
## 发现文件多了个+
# 表示这个文件有ACL的权限
[root@localhost data]# ll
总用量 0
drwxr-xr-x+ 2 root root 6 6月  17 02:48 test

```

### 删除ACL权限

```shell
[root@localhost data]# setfacl -d u:user1:5 test/
```

## sudo授权

给普通用户授予部分管理员权限

/sbin   /usr/sbin  两个目录下的命令只有超级用户才能使用

### 授权

输入visudo，进入sudo文件

```shell
## 用户名   被管理者的ip    授权命令（绝对命令）
root    ALL=(ALL)       ALL
## 组名
%wheel  ALL=(ALL)       ALL
```

给user1授予重启的权限（这个all表示被管理者ip）

```shell
user1 ALL=/usr/sbin/shutdown -r now
```

进入user1查看

```shell
[user1@localhost root]$ sudo -l
```

测试user1的权限(**必须要加sudo**)

```shell
[user1@localhost root]$ sudo /usr/sbin/shutdown -h now
```



# vim

- vim输入模式

a : 字符后插入

i : 字符前插入

o: 换行插入

- 编辑模式（输入：进入）

| 操作      | 含义                                            |
| --------- | ----------------------------------------------- |
| w         | 保存(**将文件内容另存到新文件中： w 新文件名**) |
| /查找内容 | 向下查找                                        |
| ?查找内容 | 向上查找                                        |

2. 替换

| 操作               | 含义                                      |
| ------------------ | ----------------------------------------- |
| ：1，10s/old/new g | 从1到10行，所有的old换成new （s标识替换） |
| ：1，10s/^/# g     | 将每行加上#注释（^表示正则的开头）        |
|                    |                                           |



- 命令模式

1. 移动

| 操作 | 含义        |
| ---- | ----------- |
| gg   | 移到文件头  |
| G    | 移动到行尾  |
| :n   | 跳转到第N行 |
| ^    | 移动到行首  |
| $    | 移动到行尾  |

2. 删除

| 操作   | 含义                 |
| ------ | -------------------- |
| dG     | 从光标位置删除到行尾 |
| u      | 撤销                 |
| ctrl+r | 反撤销               |
| r+字符 | 替换光标的字符       |
| x      | 删除当个字符         |

3. 复制

| 操作 | 含义           |
| ---- | -------------- |
| yy   | 复制           |
| nyy  | 复制n行        |
| p    | 光标下一行粘贴 |

4. 设置替换字符

| 操作                   | 含义                                          |
| ---------------------- | --------------------------------------------- |
| :ab 字符  替换的字符   | 当vim中输入字符后，回车就变成了替换字符       |
| ：ab email 2123@qq.com | 如，这个输入email后，回车就变成了后面得字符了 |

# 软件安装

## rpm

### 安装命令

```shell
rpm -ivh 软件名
```

### 默认安装路径

| /etc           | 一些配置档放置的目录，例如 /etc/crontab |
| -------------- | --------------------------------------- |
| /usr/bin       | 一些可运行文件                          |
| /usr/lib       | 一些程序使用的动态函式库                |
| /usr/share/doc | 一些基本的软件使用手册与说明档          |
| /usr/share/man | 一些 man page 文件                      |

### 常用命令

```shell
# 查询是否安装过
rpm -q 包名
# 查询安装的所有包
rpm -qa
# 升级
rpm-Uvh 包全名
# 卸载(rpm卸载需要一个个卸载）,一般不要用yum卸载
rpm -e 包名
# 查询
rpm -q 包名
# 查询所有的包
rpm -qa
## 查询包的安装路径
[root@localhost init.d]# rpm -ql 包名

## 查询当前文件属于哪个软件包
rpm -qf 文件名
```

### rpm包命名格式：name-version-release.arch.rpm

 name：表示包的名称，包括主包名和分包名

 version：表示包的版本信息

release：用于标识rpm包本身的发行号，可还包含适应的操作系统

 arch:表示主机平台,noarch表示此包能安装到任何平台上，和架构无关

### 注意

rpm包建议默认安装路径

因为

1. RPM是有卸载命令的，可以直接卸载，源码安装只能直接删除

## rpm在线安装（yum）

### yum源文件解析

默认情况，base.repo文件生效

```shell
[root@localhost ~]# ls /etc/yum.repos.d/
CentOS-Base.repo 
```

文件内容

baseurl:基本地址，mirrorlist:镜像地址（两个只能有一个生效）

```shell
[base] ## 容器名称，一定要放在[]中
## 容器说明
name=CentOS-$releasever - Base - mirrors.aliyun.com
#
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
## 唯一标识数字证书生效，0表示不生效
gpgcheck=1
#数字证书公钥保存位置（验证的）
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
#是否生效，不写或者1表示生效
enabled=0

```

## 搭建本地yum源

- 挂载光盘

```shell
##先建立挂载点
[root@localhost ~]# cd /mnt/cdrom/
#挂载光盘
[root@localhost cdrom]# mount /dev/cdrom /mnt/cdrom/
mount: /dev/sr0 is write-protected, mounting read-only

```

- 配置yum源

```shell
[root@localhost mnt]# cd /etc/yum.repos.d/
## 将默认的都更改文件名
[root@localhost yum.repos.d]# mv CentOS-Base.repo CentOS-Base.repo.bak
[root@localhost yum.repos.d]# mv CentOS-CR.repo CentOS-CR.repo.bak
[root@localhost yum.repos.d]# mv CentOS-Debuginfo.repo CentOS-Debuginfo.repo.bak
[root@localhost yum.repos.d]# mv CentOS-fasttrack.repo CentOS-fasttrack.repo.bak
[root@localhost yum.repos.d]# mv CentOS-Sources.repo CentOS-Sources.repo.bak
[root@localhost yum.repos.d]# mv CentOS-Vault.repo CentOS-Vault.repo.bak
##备份要修改的
[root@localhost yum.repos.d]# cp CentOS-Media.repo CentOS-Media.repo.bak
```

- 修改文件

```shell
[c7-media]
name=CentOS-$releasever - Media
baseurl=file:///mnt/cdrom/
#        file:///media/cdrom/
#        file:///media/cdrecorder/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

- 生效

```shell
#清空yum已存在的原信息（缓存）
[root@localhost yum.repos.d]# yum clean all
# 建立yum资源缓存
[root@localhost yum.repos.d]# yum makecache
```

## yum 组管理

```shell
#能看出当前系统装了什么样的组（如最小化安装、桌面等）
yum grouplist
```

## 源码包安装

一般源码包必须制定安装路径

/usr/local/*

目的是为了安装方便

### 步骤

- 解压缩源码包
- 进入解压缩目录
- ./configure     编译前准备
  - 这个命令是当前软件包提供的
  - ./configure --prefix=安装路径
  - 执行后，将结果写入makefile文件中，后续编译和安装会依赖这个文件的内容
- make    编译
  - 调用gcc编译器
- make clean   清空编译内容（非必须步骤）
  - 如果上面两个步骤有一个报错，一定要执行这个命令来情况makefile和.o的头文件
- make install    编译安装
  - 真正的安装过程，会清楚记录安装位置，可以记录下来以备以后删除应用

>>>>>>> origin/master
