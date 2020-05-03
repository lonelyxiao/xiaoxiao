# shell脚本的建立

一般我们是使用centos的vi来编辑脚本，如果使用vim，则更方便，我们可以给vim起别名，在使用vi的时候，也使用vim，编辑方法如下

```shell
#给vim起别名
[root@localhost ~]# echo "alias vi='vim'" >> /etc/profile
[root@localhost ~]# source /etc/profile

```

## 脚本开头

指定哪个解释器来执行脚本

```shell
#! /bin/bash
```

或者

```shell
#! /bin/sh
```

## shell脚本的执行

它会先查环境变量ENV，如果脚本遇到子脚本，会先执行子脚本，再执行父脚本

**脚本执行方式**

1、bash xx.shell 或者 sh xx.shell （推荐这种方式，不需要执行的权限）

2 地址 /xx.shell 需要执行权限

3 source xx.shell 或者 .  xx.shell ： 在当前shell中执行脚本，而不是产生一个子shell

# 变量

## 环境变量

## 全局环境变量

配置文件

```shell
/etc/profile
/etc/bashrc
#如果想登陆后初始化或者显示加载内容，则将脚本文件放到这个目录下
/etc/profile.d/
```

## 普通变量

普通变量定义 a=xx ， 单引号不会解析，双引号和不要引号会解析变量

## 将命令结果赋值

```shell
## 第一种方式
[root@localhost home]# varLs=$(ls)
[root@localhost home]# echo $varLs
main.248d632daf858772a20c.js.gz myredis mysql

## 第二种方式 反引号，不推荐
[root@localhost home]# varls2=`ls`
[root@localhost home]# echo $varls2
main.248d632daf858772a20c.js.gz myredis mysql

```

## 特殊状态变量

当对服务器的数据进行备份时，我们会在执行完关键命令，例如tar或cp后，通过获取返回值来判断命令是否成功，备份数据是否完整

```shell
##打印上个指令的执行状态，0：成功；
[root@localhost ~]# echo $?

```

## shell内置变量命令

常用的内部命令有：echo、eval、exec、export、read、shift

- eval

eval args

当Shell程序执行到eval语句时，Shell读入参数args，并将它们组合成一个新的命令，然后执行。

## 算数运算符

- 双小括号“(())”数值运算的基础语法

```shell
## 计算1+1
[root@localhost ~]# echo $((1+1))
2
##计算a+8的结果赋值给a
[root@localhost ~]# a=8
[root@localhost ~]# echo $((a=a+8))
16
##((a++))输出a的值
[root@localhost ~]# echo $((a++))
16

```

## let赋值表达式

```shell
[root@localhost ~]# i=1
[root@localhost ~]# i=i+8
[root@localhost ~]# echo $i
i+8
##类似 ((i=i+8))
[root@localhost ~]# i=1
[root@localhost ~]# let i=i+8
[root@localhost ~]# echo $i
9
```



# 脚本编写

## 特殊变量

- 参数变量

| 位置变量 | 说明                                                        |
| -------- | ----------------------------------------------------------- |
| $0       | 获取shell脚本的文件名，如果执行脚本带了路径，则包含脚本路径 |
| $n       | 1-9,脚本传入的参数                                          |
| $#       | 当前脚本所传参数个数                                        |
| $*       | 脚本所传的所有参数，加了“”则视所有参数为一个字符串          |
| $@       | 同$*， 加""视所有参数为不同的参数，如"$1" "$2"              |

- 利用命令快速输出

```shell
[root@localhost home]# vim p.sh
[root@localhost home]# echo \${1..15} >p.sh 
[root@localhost home]# cat p.sh 
$1 $2 $3 $4 $5 $6 $7 $8 $9 $10 $11 $12 $13 $14 $15

```

## rpcbind.sh脚本学习

```shell
case "$1" in ##接收第一个参数
  start) ## 如果第一个参数是start字符串
    start ## 执行start函数
    RETVAL=$? #接收函数返回值
    ;;
  stop)
    stop
    RETVAL=$?
    ;;
  status)
    status $prog
    RETVAL=$?
    ;;
  restart | reload| force-reload)
    stop
    start
    RETVAL=$?
    ;;
  condrestart | try-restart)
    if [ -f /var/lock/subsys/$prog ]; then
        stop
        start -w
        RETVAL=$?
    fi
    ;;
  *)
    echo $"Usage: $0 {start|stop|status|restart|reload|force-reload|condrestart|try-restart}"
    RETVAL=2
    ;;
esac
 
exit $RETVAL
```

```shell
start() { ##定义start 函数
    
    return $RETVAL ## 返回值
}
```

# 条件表达

## 条件测试

```shell
## test语句
## 如果 文件存在则输出true， 否则书册false
[root@localhost ~]# test -f ./anaconda-ks.cfg && echo true || echo false
true

## 检测字符串长度是否为0
[root@localhost ~]# test -z 'aaa' && echo true || echo false
false
```

