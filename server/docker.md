# docker优势

**更快速的交付和部署**

**更高效的虚拟化**

**更轻松的迁移和扩展**

**更简单的管理**

# 系统要求

centos 6.5 （内核高于 用uname -r 命令查看）以上

# 相关概念



docker主机(Host)：安装了Docker程序的机器（Docker直接安装在操作系统之上）；

docker客户端(Client)：连接docker主机进行操作；

docker仓库(Registry)：用来保存各种打包好的软件镜像；

docker镜像(Images)：软件打包好的镜像；放在docker仓库中；

docker容器(Container)：镜像启动后的实例称为一个容器；容器是独立运行的一个或一组应用



步骤：先去仓库下载镜像到本地，运行镜像，启动容器

# linux 使用docker

```shell
1、检查内核版本，必须是3.10及以上
uname -r
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
## 或者也可以搜索对应版本进行安装
[root@k8smaster ~]# yum list docker-ce --showduplicates | sort -r
[root@k8smaster ~]# yum install -y docker-ce-18.06.3.ce-3.el7
3、输入y确认安装
4、启动docker
[root@localhost ~]# systemctl start docker
[root@localhost ~]# docker version
Docker version 1.12.6, build 3e8e77d/1.12.6
5、开机启动docker
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
6、停止docker
systemctl stop docker
```

ps:也可以这样快速安装

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```



# 阿里云镜像加速

阿里云->产品与服务->容器镜像服务->镜像中心->镜像加速器

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
 docker info 
#查看到
Registry Mirrors:
 https://.mirror.aliyuncs.com/

```

## daemon配置

- bip

  - 由于 docker 本身是默认使用 B 类地址（172.xx.0.0/16)， 大部分情况下会和公司网络产生冲突。
  - 为了解决这个问题，需要在 `/etc/docker/daemon.json` 中增加 `"bip":"169.254.31.1/24"` 指定容器使用的网络。

  ```json
  {
      "bip":"169.254.31.1/24"
  }
  ```

# 私有镜像仓库

- 下载harbo源码

```shell
[root@k8sm opt]# mkdir src
[root@k8sm src]# wget https://github.com/goharbor/harbor/releases/download/v1.8.6/harbor-offline-installer-v1.8.6.tgz
```

- 安装

```shell
[root@k8sm src]# mkdir /opt/harbor
[root@k8sm src]# tar -zxvf harbor-offline-installer-v1.10.1.tgz  -C /opt/
## 解压之后，做一个版本标识，然后做软连接，方便后续升级
[root@k8sm opt]# mv harbor harbor-1.10.1
[root@k8sm opt]# ln -s harbor-1.10.1 harbor
```

- 编辑harbor.yml文件

```shell
[root@k8sm harbor]# vim harbor.yml
##修改
hostname: m.host.com
## 端口修改
 port: 180
 
## 密码
harbor_admin_password: Harbor12345

```

- 安装docker环境

```shell
## 安装epel环境
yum install epel-release
#安装docker-compose
[root@k8sm home]# yum install docker-compose -y
#执行
[root@k8sm harbor]# ./install.sh

## 查看相关记录
[root@k8sm harbor]# docker-compose ps

```

- 查看http://192.168.1.143:180/   admin  harbor12345

# docker helloword

```shell
[root@localhost docker]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

# docker是怎么工作的

docker守护进程运行在主机上，客户端通过socket与之相连，守护进程接受客户端命令，管理运行在主机上的容器



# Docker命令


## 镜像命令

| 操作 | 命令                                            | 说明                                                     |
| ---- | ----------------------------------------------- | -------------------------------------------------------- |
| 检索 | docker  search 关键字  eg：docker  search redis | 我们经常去docker  hub上检索镜像的详细信息，如镜像的TAG。 |
| 拉取 | docker pull 镜像名:tag                          | :tag是可选的，tag表示标签，多为软件的版本，默认是latest  |
| 列表 | docker images                                   | 查看所有本地镜像                                         |
| 删除 | docker rmi image-id                             | 删除指定的本地镜像                                       |

操作步骤：

```shell
1、搜索镜像
[root@localhost ~]# docker search tomcat
2、拉取镜像
[root@localhost ~]# docker pull tomcat
3、根据镜像启动容器
docker run --name mytomcat -d tomcat:latest
4、docker ps  
查看运行中的容器
5、 停止运行中的容器
docker stop  容器的id
6、查看所有的容器
docker ps -a
7、启动容器
docker start 容器id
8、删除一个容器
 docker rm 容器id
 强行关闭容器，并且删除
 docker rm -f 容器id
 删除所有镜像
 docker rmi -f $(docker images -q)
9、启动一个做了端口映射的tomcat
[root@localhost ~]# docker run -d -p 8888:8080 tomcat
-d：后台运行
-p: 将主机的端口映射到容器的一个端口    主机端口:容器内部的端口

10、为了演示简单关闭了linux的防火墙
service firewalld status ；查看防火墙状态
service firewalld stop：关闭防火墙
11、查看容器的日志
docker logs container-name/container-id

更多命令参看
https://docs.docker.com/engine/reference/commandline/docker/
可以参考每一个镜像的文档

```

## 容器命令

| 命令                  | 描述                                                         |
| :-------------------- | ------------------------------------------------------------ |
| docker run 镜像名     | 创建并运行容器命令，如果docker主机已经下载过tomcat，则该命令会直接创建一个tomcat的容器实例，否则会先去hub端拉取该tomcat镜像， |
| docker run -it 镜像名 | -i : 表示创建要给交互式容器，-t：表示运行容器的同时创建一个伪终端，-d: 后台守护进程的方式运行； 一般与 -i 一起使用 |
| docker ps             | 查看当前正在运行的容器对象                                   |
| docker ps -l          | -l(小写的L) ： 默认的查看只会查看正在运行中的容器信息，而ps -l 会显示最近运行的一条容器信息 |
| docker ps -a          | -a : 显示所有运行过的镜像信息                                |



# 命令举例

## 拉取镜像

```shell
#获取centos
[root@localhost ~]# docker pull centos
#可以看到，centos只有200M,这是因为
#docker是基于容器虚拟化的，它只包含runtime的环境
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              9f38484d220f        2 months ago        202MB
hello-world         latest              fce289e99eb9        5 months ago        1.84kB
#新建运行一个实例
[root@localhost ~]# docker run --help
Usage:	docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# options
-i: 以交互模式运行容器，通常与-t同时使用
-t: 为容器重新分配一个伪输入终端
-d： 以守护进程运行
```

---

## 运行容器
```shell
##启动容器，发现启动centos，并且进入centos的终端界面，
#这就是-it的效果
[root@localhost ~]# docker run -it centos
[root@132c9b24081e /]# 
```

--name：为容器起一个别名

```shell
#查看所有运行中的容器
[root@localhost ~]# docker ps --help
Usage:	docker ps [OPTIONS]
# options
-a 列出历史和正在运行的容器
-l 显示最近创建的容器
-n 2 显示两个历史和正在运行的容器
```

---

**容器两种退出方式**

exit:容器停止退出

ctrl+p+q:容器不停止退出

---

- 启动容器

```shell
[root@localhost ~]# docker ps -n 2
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
132c9b24081e        centos              "/bin/bash"         13 hours ago        Exited (0) 12 hours ago                       cranky_shirley
95fdf316c2a9        hello-world         "/hello"            37 hours ago        Exited (0) 37 hours ago                       cocky_hofstadter
#根据容器id启动
[root@localhost ~]# docker start 132c9b24081e
132c9b24081e
#温柔停止容器docker stop 容器id/容器名
[root@localhost ~]# docker stop 132c9b24081e
132c9b24081e


[root@localhost ~]# docker start cranky_shirley
cranky_shirley
#强制停止容器
[root@localhost ~]# docker kill cranky_shirley
cranky_shirley

```

## 后台运行容器

```shell
[root@localhost ~]# docker run -d centos
0a4c2dbad8de335ac27fd72644834f99dd4012296f5c221e8d3902a340652966
##我们发现，没有正在运行的容器，那是因为，以-d模式运行容器，如果没有前台进程运行
#会立即kill掉容器
[root@localhost ~]# docker ps
#如果我们在容器中加入前台运行进程（一直循环，每两秒打印）
[root@localhost ~]# docker run -d centos /bin/sh -c "while true;do echo hello;sleep 2; done"
23ae37160eb2757ee7ce7f03b53689c77f919ed308db00828c4b3d505a3656d0
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
23ae37160eb2        centos              "/bin/sh -c 'while t…"   10 seconds ago      Up 8 seconds                            cranky_feistel
#查看容器中的日志
[root@localhost ~]# docker logs 23ae37160eb2
hello
hello

```

 ## 查看容器日志

```shell

[root@localhost ~]# docker logs -f -t --tail 2 23ae37160eb2
2019-06-01T05:49:19.592867461Z hello
2019-06-01T05:49:21.594822647Z hello

-t 加入时间戳
-f 跟随最新的日志打印
--tail 数字  显示最后N条日志
```

## 查看容器内运行的进程

docker top 容器id

## 查看容器内部细节

docker inspect 容器id

## 在容器内命令行交互

```shell
[root@localhost ~]# docker start 132c9b24081e
132c9b24081e
#进入交互界面
[root@localhost ~]# docker attach 132c9b24081e
[root@132c9b24081e /]# 
#执行bash命令，返回给宿主机
[root@localhost ~]# docker exec -t 132c9b24081e ls -l /tmp
total 4
-rwx------. 1 root root 836 Mar  5 17:36 ks-script-eC059Y
-rw-------. 1 root root   0 Mar  5 17:34 yum.log
#也可以直接进入docker容器中
[root@localhost ~]# docker exec -t 132c9b24081e /bin/bash
[root@132c9b24081e /]# 

```

## 将容器内容拷贝到宿主机

```shell
#docker cp 容器id：容器文件目录 宿主机目录
[root@localhost ~]# docker cp 132c9b24081e:/tmp/yum.log /home/

```

## 映射端口

```shell
#将8088端口映射到docker启动的容器的8080端口上，
#直接访问tomcat：http://192.168.94.129:8088/
[root@localhost home]# docker run -p 8088:8080 tomcat
```



## 修改容器挂载

export为镜像，在重新构建新容器

```shell
[root@localhost home]# docker container export -o ./jk.docker jenkins
[root@localhost home]# docker import ./jk.docker jk
##运行导入镜像时要带 command
[root@localhost home]# docker run -d -p 8080:8080 -p 10241:50000 -v /home/jenkins_node:/var/jenkins_home -v /etc/localtime:/etc/localtime -v /var/run/docker.sock:/var/run/docker.sock --name jk jk /bin/bash

```





# docker镜像

轻量级、可执行的独立软件包 

docker镜像时由一层一层的文件系统组成（联合文件系统）

bootfs主要包括bootloader和kernel(内核)，bootloader负责引导加载kernel，docker的最底层就是bootfs

rootfs在bootfs之上，例如：centos的 /dev等文件，rootfs就是各种不同的操作系统发行榜（为啥docker的centos的镜像只有200M，因为其共用了宿主机的内核）



tomcat 镜像为啥由400M，其实就是其联合文件系统的原因：

kernel - centos - jdk8 -tomcat，比如这几层文件系统

docker为啥使用分层的联合文件系统，（可以共享资源，每个镜像的每一层，都可以被共享）

## 利用原有容器，创建新镜像

```shell
#docker commit -a="作者" -m="描述" 容器id 制作后镜像名称:版本
[root@localhost home]# docker commit -a="xiao" -m="xiaomouren" 2540ff7bb4a0 xiaojihao/mytomcat:8.5
sha256:6c7b91829a93149386ea0a27e894ecc89e198e940fb01bd75db5d076878b3915
[root@localhost home]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
xiaojihao/mytomcat   8.5                 6c7b91829a93        10 seconds ago      522MB
tomcat               latest              894b39cf2fa1        2 days ago          522MB
centos               latest              9f38484d220f        2 months ago        202MB
hello-world          latest              fce289e99eb9        5 months ago        1.84kB

```

# 容器数据卷

## 概念

容器运行产生的数据，在容器停止时不会保留下来的。容器数据卷就是将容器运行的数据持久化

数据卷能做到容器到主机，主机到容器间的数据共享

## 特点

\- 数据卷可以在容器之间共享和重用

\- 对数据卷的修改会立马生效

\- 对数据卷的更新，不会影响镜像

\- 卷会一直存在，直到没有容器使用

## 命令

docker run -it -v /宿主机目录：/容器目录 镜像名

给容器目录设置权限，只运行容器读，不允许写

docker run -it -v /宿主机目录：/容器目录:ro 镜像名



# dockerFile

构建镜像的文件

## 示例

```shell
#建立一个文件
[root@localhost home]# vim dockerfile
#构建镜像
[root@localhost home]# docker build -f /home/dockerfile -t mydf/centos .
#运行镜像
[root@localhost home]# docker run -it mydf/centos
#查看容器信息
[root@localhost home]# docker inspect 388a53771eb9
```

可以看到容器与宿主机的挂载信息

```json
 "Mounts": [
            {
                "Type": "volume",
                "Name": "dbbec90170740de1cfb90ada79c1909aebc3774c25c2c07f2fb2ae43e85fd7df",
                "Source": "/var/lib/docker/volumes/dbbec90170740de1cfb90ada79c1909aebc3774c25c2c07f2fb2ae43e85fd7df/_data",
                "Destination": "/datavContainer1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "cc66ad788a98457cf5a629be534e9d7719d8d5555ea34d51488582ad74b5ebf4",
                "Source": "/var/lib/docker/volumes/cc66ad788a98457cf5a629be534e9d7719d8d5555ea34d51488582ad74b5ebf4/_data",
                "Destination": "/datavContainer2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```

## dockerfile内容基础

- 每条保留字指令都必须为大写字母且后面要跟随至少一个参数

- 指令按照从上到下顺序执行
- 每条指令都会创建一个新的镜像层，并对镜像进行提交

 ## 运行大致流程

从基础镜像运行一个容器

执行每条指令对容器进行修改

执行类似 docker commit的操作提交一个新的镜像层

基于刚提交的镜像运行一个新的容器

执行下条指令

## 指令

| 命令                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| FROM image                            | 基础镜像，当前新镜像是基于哪个镜像的                         |
| MAINTAINER username                   | 镜像维护者的姓名和邮箱地址                                   |
| RUN command                           | 容器构建时需要运行的命令                                     |
| EXPOSE  port                          | 当前容器对外暴露出的端口                                     |
| WORKDIR path_dir                      | 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点     |
| ENV   key   value                     | 用来在构建镜像过程中设置环境变量                             |
| ADD  source_dir/file   dest_file/file | 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包 |
| COPY                                  | 类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置 |
| VOLUME                                | 容器数据卷，用于数据保存和持久化工作                         |
| CMD                                   | 指定一个容器启动时要运行的命令,ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数 |
| ENTRYPOINT                            | 指定一个容器启动时要运行的命令;ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数 |
| ONBUILD                               | 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发(子镜像在build的时候出发) |
| ARG <参数名>[=<默认值>]               | 构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效， 也就是说只有 docker build 的过程中有效 |

## 构建命令

- docker build

```shell
--build-arg，设置构建时的环境变量; arg =  xxx
--file, -f，Dockerfile的完整路径，默认值为‘PATH/Dockerfile’
--tag, -t，镜像的名字及tag，通常name:tag或者name格式；可以在一次构建中为一个镜像设置多个tag
```



## dockerFile 示例

```shell
FROM centos
VOLUME ["/datavContainer1", "/datavContainer2"]
CMD echo "finished"
CMD /bin/bash

```

## 制作微服务镜像

- Dockerfile内容

```shell
FROM openjdk:8-jdk-alpine
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
EXPOSE 80
ENTRYPOINT ["java","jar", "/app.jar"]
```

- 构建镜像(JAR_FILE 相对路径)

```shell
[root@localhost home]# docker build --build-arg JAR_FILE=es-jd-1.0-SNAPSHOT.jar -f /home/Dockerfile -t jd:v1 .
```



# docker 安装mysql

```shell
[root@localhost ~]# docker run -p 3306:3306 --name mysql \
> -v /home/mysql/conf:/etc/mysql/conf.d \
> -v /home/mysql/logs:/logs \
> -v /home/mysql/data:/var/lib/mysql \
> -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

```

如果连接太慢

有2种解决办法：
1，把client的ip写在mysql服务器的/etc/hosts文件里，随便给个名字就可以了。
2，在 my.cnf 中加入 –skip-name-resolve 。
对于第一种方法比较笨，也不实用，那么 skip-name-resolve 选项可以禁用dns解析，但是，这样不能在mysql的授权表中使用主机名了

```cnf
[mysqld]
skip-name-resolve
```

# 安装redis

```shell
docker run -p 6379:6379 -v /zzyyuse/myredis/data:/data -v /zzyyuse/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf  -d redis:3.2 redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

# 按照rabbitmq

```shell
# 下载镜像
[root@localhost ~]# docker pull rabbitmq:3-management
## 查询镜像
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
rabbitmq            3-management        46c7d1fde5d3        2 days ago          184MB
##安装镜像
[root@localhost ~]# docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq 46c7d1fde5d3
```

访问<http://192.168.94.134:15672/>    guest/guest

# 安装nginx

```shell
docker run --name nginx -p 80:80 -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx/html:/usr/share/nginx/html  -d nginx
```

在配置文件下配置

```conf
#工作进程数  1 ,不要超过计算机的核数,四核配置4,八核配置8
worker_processes  1;

#工作连接数,也就是线程,一个进程有1024个线程,
events {
    worker_connections  1024;
}

#http请求配置
http {
    default_type  application/octet-stream;
	
	#sendfile为发送文件,要on开启
    sendfile        on;
	
	#keepalive_timeout超时时间
    keepalive_timeout  65;
	
	server {
	    #监听的端口,这里为80
		listen       80;
		#server_name就是域名,
		server_name  localhost;
		
		#location域名代理地址
	    # / 代表所有请求路径
		location / {
			root /usr/share/nginx/html;
			index  index.html;
		}
	}
}
```

# 安装ES集群

- 建立三个配置文件 es1.yml,es2.yml,es3.yml
  - 这里注意端口和映射端口要一致

```yaml
cluster.name: elasticsearch-cluster
node.name: es-node1
network.bind_host: 0.0.0.0
network.publish_host: 127.0.0.1
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true
#集群节点
discovery.seed_hosts: ["192.168.1.134:9300","192.168.1.134:9301", "192.168.1.134:9302"]
#有资格成为主节点的节点配置
cluster.initial_master_nodes: ["es-node1","es-node2","es-node3"]

```

- 建立数据存储目录 data0 data1 data2, 并且授予 777权限

- 修改文件

```shell
[root@localhost ~]# vim /etc/sysctl.conf 
vm.max_map_count=655360
[root@localhost ~]#sysctl -p
```

- 启动

```shell

 docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 -v /home/es/es1.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/es/data0:/usr/share/elasticsearch/data --name es0 elasticsearch:7.9.2

 docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9201:9201 -p 9301:9301 -v /home/es/es2.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/es/data1:/usr/share/elasticsearch/data --name es1 elasticsearch:7.9.2

 docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9202:9202 -p 9302:9302 -v /home/es/es3.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/es/data2:/usr/share/elasticsearch/data --name es2 elasticsearch:7.9.2

```

# 安装Kibana

- 编辑配置文件 kibana.yml

```yaml
server.name: kibana
server.host: "0"
elasticsearch.hosts: ["http://192.168.1.134:9201"]
xpack.monitoring.ui.container.elasticsearch.enabled: true
elasticsearch.pingTimeout: 90000
elasticsearch.requestTimeout: 90000
i18n.locale: "zh-CN"

```

- 运行

```shell
docker run -d --name kibana -p 5601:5601 \
-v /home/es/kibana.yml:/usr/share/kibana/config/kibana.yml \
kibana:7.9.2
```

- 配置文件说明

```yaml
#节点地址和端口 必须是同一个集群的 必须以http或者https开头 填写实际的es地址和端口
elasticsearch.hosts: ['http://172.16.10.202:9200','http://172.16.10.202:9202', 'http://172.16.10.202:9203']
#发给es的查询记录 需要日志等级是verbose=true 
elasticsearch.logQueries: true
#连接es的超时时间 单位毫秒
elasticsearch.pingTimeout: 30000
elasticsearch.requestTimeout: 30000
#是否只能使用server.host访问服务
elasticsearch.preserveHost: true
#首页对应的appid
kibana.defaultAppId: "home"
kibana.index: '.kibana'
#存储日志的文件设置
logging.dest: /usr/share/kibana/logs/kibana.log
logging.json: true
#是否只输出错误日志信息
logging.quiet: false
logging.rotate:
  enabled: true
  #日志文件最大大小
  everyBytes: 10485760
  #保留的日志文件个数
  keepFiles: 7
logging.timezone: UTC
logging.verbose: true
monitoring.kibana.collection.enabled: true
xpack.monitoring.collection.enabled: true
#存储持久化数据的位置
path.data: /usr/share/kibana/data
#访问kibana的地址和端口配置 一般使用可访问的服务器地址即可
server.host: 0
#端口默认5601
server.port: 5601
server.name: "kibana"
#配置页面语言
i18n.locale: zh-CN
```

