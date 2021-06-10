# 持续交付

- 版本控制系统
- 持续集成工具
- 部署工具

# GitLab

GitLab 分布式版本控制

## GitLab安装

```shell
#关闭防火墙
[root@localhost ~]# systemctl stop firewalld
#禁止防火墙开机启动
[root@localhost ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

```

```shell
##关闭selinux,将里面SELINUX=disabled的值改为disabled，重启linux
[root@localhost ~]# vi /etc/sysconfig/selinux
#重启后查看是否关闭
[root@localhost ~]# getenforce 
Disabled
```

```shell
# 安装依赖
[root@localhost ~]# yum install curl policycoreutils openssh-server openssh-clients postfixs	
```

或者去这个网站下载https://packages.gitlab.com/gitlab/gitlab-ce
```shell
##下载gitlab
[root@localhost ~]# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

```

```shell
#开启postfix
[root@localhost ~]# systemctl start postfix
[root@localhost ~]# systemctl enable postfix
```

```shell
#启动gitce
[root@localhost ~]# yum -y install gitlab-ce 
```

```shell
# 创建gitlab证书
[root@localhost ~]# mkdir -p /etc/gitlab/ssl
# 创建本地私钥
[root@localhost ~]# openssl genrsa -out "/etc/gitlab/ssl/gitlab.example.com.key" 2048
## 创建证书
[root@localhost gitlab]# openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"

```

```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:bj
Locality Name (eg, city) [Default City]:bj
Organization Name (eg, company) [Default Company Ltd]:gitlab.example.com
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:gitlab.example.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123456
An optional company name []:

```

```shell
#查询是否创建成功
[root@localhost ssl]# ls
gitlab.example.com.csr  gitlab.example.com.key
#创建crt证书
[root@localhost ssl]# openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"
#查看创建成功
[root@localhost ssl]# ll
总用量 12
-rw-r--r-- 1 root root 1184 9月  19 23:38 gitlab.example.com.crt
-rw-r--r-- 1 root root 1025 9月  19 23:01 gitlab.example.com.csr
-rw-r--r-- 1 root root 1679 9月  19 22:54 gitlab.example.com.key
#创建PEM证书
[root@localhost ssl]# openssl dhparam -out /etc/gitlab/ssl/dhparans.pem 2048

[root@localhost ssl]# ll
总用量 16
-rw-r--r-- 1 root root  424 9月  19 23:40 dhparans.pem
-rw-r--r-- 1 root root 1184 9月  19 23:38 gitlab.example.com.crt
-rw-r--r-- 1 root root 1025 9月  19 23:01 gitlab.example.com.csr
-rw-r--r-- 1 root root 1679 9月  19 22:54 gitlab.example.com.key
#修改权限
[root@localhost ssl]# chmod 600 *
[root@localhost ssl]# ll
总用量 16
-rw------- 1 root root  424 9月  19 23:40 dhparans.pem
-rw------- 1 root root 1184 9月  19 23:38 gitlab.example.com.crt
-rw------- 1 root root 1025 9月  19 23:01 gitlab.example.com.csr
-rw------- 1 root root 1679 9月  19 22:54 gitlab.example.com.key
#标记gitlab配置文件
vi /etc/gitlab/gitlab.rb
##找到这一行，将http改成https
external_url 'https://gitlab.example.com'
##将其更改为true
nginx['redirect_http_to_https'] = true
#将其改为gitlab.example.com.crt
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
nginx['ssl_dhparam'] = /etc/gitlab/ssl/dhparams.pem

##重新初始化配置
[root@localhost gitlab]# gitlab-ctl reconfigure
```

```shell
#更改nginx文件
[root@localhost gitlab]# vi /var/opt/gitlab/nginx/conf/gitlab-http.conf
#重定向所有gitlab请求
server_name gitlab.example.com;
rewrite ^(.*)$ https://$host$1 permanent;

#重启gitlab
[root@localhost gitlab]# gitlab-ctl restart
```

在自己的电脑上host配置

192.168.94.132 gitlab.example.com

浏览器直接访问地址

第一次进入先设置管理员密码 12345678

输入gitlab默认管理员账号root/12345678

点击d顶部+图标，new project，创建一个新的仓库

创建一个test仓库

```shell
#将仓库clone下来
$ git -c http.sslVerify=false clone https://gitlab.example.com/root/test.git
#新建
$ vi test.java
##推送仓库
Administrator@PC-20190425JQWU MINGW64 /d/gitlab/test (master)
$ git add .
warning: LF will be replaced by CRLF in test.java.
The file will have its original line endings in your working directory.

Administrator@PC-20190425JQWU MINGW64 /d/gitlab/test (master)
$ git commit -m "first"
[master (root-commit) 7e37d48] first
 1 file changed, 2 insertions(+)
 create mode 100644 test.java

Administrator@PC-20190425JQWU MINGW64 /d/gitlab/test (master)
$ git -c http.sslVerify=false push origin master
Username for 'https://gitlab.example.com': root

```

## gitlab应用

## 合并分支

创建一个普通用户：dev

创建一个领导账号：lead

![1569336038203](../image/jkens/1569336038203.png)

进入创建的测试仓库，将创建好的两个用户添加到仓库中，点击**Manage access**进行添加

dev-developer

lead-master

用dev用户domn下来代码

```shell
$ git -c http.sslVerify=false clone https://gitlab.example.com/root/test.git

#创建一个新版本的分支，进入分支
$ cd test/
$ git checkout -b release-1.0
Switched to a new branch 'release-1.0'
#修改java代码，将其推送到release-1.0中
$ git add .
$ git commit -m "re-1.0"
$ git -c http.sslVerify=false push origin release-1.0

```

使用dev登录gitlab，然后点击create merge requests进行合并分支，发送给lead

![](../image/jkens/20190924234856.png)

使用lead账号登录，发现一条合并申请，点击merge进行合并，此时，代码已经合并

# ansible

## 配合安装virtualenv

推荐使用git源代码安装（保证在python独立的环境下工作）

防火墙等关掉

```shell
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# systemctl disable firewalld
[root@localhost ~]# getenforce
Disabled
```

安装python

```shell
[root@localhost ~]# wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz
```

解压python

安装python

```shell
[root@localhost Python-3.6.5]# ./configure --prefix=/usr/local --with-ensurepip=install --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"

[root@localhost Python-3.6.5]# make && make altinstall
#查看安装路径
[root@localhost Python-3.6.5]# which pip3.6
/usr/local/bin/pip3.6
#建立软连接
[root@localhost Python-3.6.5]# ln -s /usr/local/bin/pip3.6 /usr//local/bin/pip
#安装virtualenv
root@localhost ~]# pip install -i https://pypi.tuna.tsinghua.edu.cn/simple virtualenv

##创建一个用户
[root@localhost ~]# useradd deploy
[root@localhost ~]# su - deploy

#创建一个env实例来集成ansible2.5版本
[deploy@localhost ~]$ virtualenv -p /usr/local/bin/python3.6 .py3-a2.5-env

#查看是否安装git
[deploy@localhost root]$ cd /home//deploy/.py3-a2.5-env/
[deploy@localhost .py3-a2.5-env]$ which git
#没安装Git，进入root用户
#安装git
[root@localhost ~]# yum -y install git nss curl
#进入deploy用户，下载ansible源码
[root@localhost ~]# su - deploy
上一次登录：五 10月 11 22:19:20 CST 2019pts/0 上
[deploy@localhost ~]$ git clone https://github.com/ansible/ansible.git
#加载env环境
[deploy@localhost ~]$ source /home/deploy/.py3-a2.5-env/bin/activate
(.py3-a2.5-env) [deploy@localhost ~]$ 
# 安装依赖包
[deploy@localhost ~]$ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple paramiko PyYAML jinja2
#
[deploy@localhost ~]$ mv ansible .py3-a2.5-env/
#将ansible切换到2.5版本
(.py3-a2.5-env) [deploy@localhost ~]$ cd .py3-a2.5-env/ansible/
(.py3-a2.5-env) [deploy@localhost ansible]$ git checkout stable-2.5
分支 stable-2.5 设置为跟踪来自 origin 的远程分支 stable-2.5。
切换到一个新分支 'stable-2.5'
#加载2.5版本环境
[deploy@localhost ansible]$ source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q
#查看版本
[deploy@localhost ansible]$ ansible --version
ansible 2.5.15 
```

如果我们已经安装好，后续想要使用ansible

```shell
#1
[root@localhost ~]# su deploy
#2
[deploy@localhost home]$ source /home/deploy/.py3-a2.5-env/bin/activate
#3
(.py3-a2.5-env) [deploy@localhost home]$ source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q
#4
(.py3-a2.5-env) [deploy@localhost home]$ ansible --version

```

## ansible playbooks

一种ansible编排语言的框架

在ansible主机中

```shell
(.py3-a2.5-env) [deploy@localhost ~]$ mkdir test_playbooks
(.py3-a2.5-env) [deploy@localhost ~]$ cd test_playbooks/
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ ls
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ mkdir inventory
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ mkdir roles
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ cd inventory/
(.py3-a2.5-env) [deploy@localhost inventory]$ vi testenv

```

testenv内容

```shell
[testserevers]
test.example.com

[testserevers:vars]
server_name=test.example.com
user=root
output=/root/test.txt                      
```

创建testbox的主任务文件

```shell
(.py3-a2.5-env) [deploy@localhost roles]$  mkdir testbox
(.py3-a2.5-env) [deploy@localhost roles]$ cd testbox/
(.py3-a2.5-env) [deploy@localhost testbox]$ ls
(.py3-a2.5-env) [deploy@localhost testbox]$ mkdir tasks
(.py3-a2.5-env) [deploy@localhost testbox]$ cd tasks/
(.py3-a2.5-env) [deploy@localhost tasks]$ vi main.yml

```

main.yml内容

```shell
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }} > {{ output }}'"

```

编辑deploy文件

```shell
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ ls
inventory  roles
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ vi deploy.yml

```



```yaml
#对应我们testenv下的testservers子标签
- hosts: "testservers"
  gather_facts: true
  # 告诉ansible我们在目录下使用root权限进行操作
  remote_user: root
  # 任务目录task下为testbox
  roles:
    - testbox

```

查看目录（框架范例）

```shell
[deploy@localhost ~]$ tree .
.
└── test_playbooks
    ├── deploy.yml ## 入口文件
    ├── inventory
    │   └── testenv ##
    └── roles
        └── testbox ## 任务名称
            └── tasks
                └── main.yml #主任务文件，保存我们远程执行的任务

5 directories, 3 files

```

### 免秘钥登录

使用root用户登录ansible主机，修改hosts,192.168.94.129为测试textbox的主机ip

```shell
192.168.94.129 test.example.com

```

进入deploy用户,生成秘钥

```shell
[deploy@localhost ~]$ ssh-keygen -t rsa
[deploy@localhost ~]$ ssh-copy-id  -i /home/deploy/.ssh/id_rsa.pub root@test.example.com 

```

### 执行playbooks命令

```shell
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ ansible-playbook -i inventory/testenv ./deploy.yml
```

## playbooks常用模块

### File模块

在目录主机创建文件或目录，并赋予其系统权限

```yaml
- name: create file ## 描述干什么
# path：目标路径  state:touch  mode:权限 
　file: 'path=/tmp/test.txt state=touch mode=0755 owner=user01 group=user01'
```

### Copy模块

实现Ansible服务端到目标主机的文件传送

```yaml
- name: copy file
  copy: 'remote_src=no src=/tmp/test.txt dest=/tmp/test.txt mode=0644 force=yes'
```

### Stat模块

获取远程文件状态信息

```yaml
- name: check if text.txt exists
  stat: 'path=/tmp/text.txt' ##查询文件类容，赋值给stat
  register: script_stat   # 将stat结果赋值给 script_stat
```

### Debug模块

打印语句到Ansible执行输出

```yaml
- debug: msg=text.txt exists
  when: script_stat.stat.exists　　# 跟Stat模块配合使用
```

### Command/Shell模块

用来执行Linux目标主机命令，区别为：Shell —— 会调用系统中的/bin/bash，这样就可以使用系统中的环境变量，例如重新向，管道符。

```yaml
- name: run a script
  command: 'echo "hello world" > test.txt'
```

### Template模块

实现Ansible服务端到目标主机的jinja2模板传送

```yaml
- name: write the nginx config file
  template: src=/tmp/nginx.conf.j2 dest=/etc/nginx/nginx.conf
```

### Packaging模块

调用目标主机系统包管理工具（yum, apt）进行安装

```yaml
# centos系统
- name: ensure nginx is at the latest version
  yum: pkg=nginx state=latest
# ubuntu
- name: ensure nginx is at the latest version
  apt: pkg=nginx state=latest
```

### Service模块

管理目标主机系统服务

```yaml
- name: start nginx service
  service: name=nginx state=started
```

## playbooks模块使用

编辑main.yml

```shell
(.py3-a2.5-env) [deploy@localhost test_playbooks]$ vi roles/testbox/tasks/main.yml
```

```yaml

- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }} > {{ output }}'"
- name: create file
  file: 'path=/home/111.txt state=touch mode=0755 owner=root group=root'

```

