基础部分

## k8s功能

- 自动装箱
- 自动修复
- 水平拓展
- 负载均衡，服务发现
- 滚动更新
  - 加应用的时候，检测没有问题才进行服务

## k8s 的组件

- master（主控节点）
  - apiserver：对外统一入口，并负责接收、校验并响应所有的rest请求，结果状态被持久存储于etcd中
  - scheduler：节点调度，选择可用的节点，进行应用部署
  - controller manager：处理集群常规后台任务，一个资源对应一个controller，
  - etcd：保存了整个集群的状态
- node（工作节点）
  - kubelet：管理node节点的容器状态（生命周期）
  - kube-proxy：提供网络代理，提供负载均衡功能

![](../image/server/k8s/2019122614055974.png)

## 核心概念

- Prod
  - 最小部署单元
  - 一组容器的集合
  - 他们共享网络
  - 生命周期是短暂的
- contoller
- service
  - 定义一组prod的访问规则

# 部署

## kubeadmin方式

- 准备三台服务器， master， node1, node2

- 系统初始化操作（除特点说明，其他都要在三台服务器执行）

```shell
	## 三台服务器关闭防火墙
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# systemctl disable firewalld.service 
## 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config
## 关闭 swap
sed -ri 's/.*swap.*/#&/' /etc/fstab  
## 设置规划主机名
[root@localhost ~]# hostnamectl set-hostname k8smaster
# 查看
[root@localhost ~]# hostname
k8smaster
# 在master节点上添加host（需要与修改的hostname一致）
cat >> /etc/hosts <<EOF 
192.168.1.140 k8smaster
192.168.1.141 k8snode1
192.168.1.142 k8snode2
EOF
## 对网络进行设置
##将桥接的IPV4流量传递到iptables的链
cat >> /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# 生效
[root@localhost ~]# sysctl --system

##时间同步
[root@localhost ~]# yum install ntpdate -y
[root@localhost ~]# ntpdate ntp1.aliyun.com
```

- 安装docker
  - 安装yum install -y docker-ce-18.06.3.ce-3.el7版本
- 设置k8s的阿里源

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

- 安装k8s

```shell
# 搜索各个版本的k8s
[root@k8snode2 ~]# yum search kubelet --showduplicates | sort -r
[root@k8snode2 ~]# yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubect1-1.18.0
#开机启动
[root@k8smaster ~]# systemctl enable kubelet
```

- 部署k8s master(在master节点上执行)（192.168.1.140执行）
  - 当前节点ip
  - 使用阿里云镜像（从阿里云拉取很多组件镜像）
  - 指定版本
  - 连接访问的ip（不冲突就行）

```shell
kubeadm init \
--apiserver-advertise-address=192.168.1.140 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.18.0 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16
```

出现类似警告

detected "cgroupfs" as the Docker cgroup driver

需要修改docker

```shell
vim /etc/docker/daemon.json
##追加内容
{
 "exec-opts":["native.cgroupdriver=systemd"]
}
##生效
systemctl restart docker
systemctl status docker
```

可以在master节点上查看images

```shell
[root@k8smaster ~]# docker images
```

在master部署k8s成功后，我们看到如下图

![](..\image\server\k8s\20201115004538.png)

- 使用kubectl工具
  - 执行途中的mkdir命令

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```shell
##查看节点
[root@k8smaster ~]# kubectl get nodes
NAME        STATUS     ROLES    AGE     VERSION
k8smaster   NotReady   master   6m46s   v1.18.0
```

- 将node加入k8smaster中
  - 在另外两个node执行图中命令

```shell
kubeadm join 192.168.1.140:6443 --token e21hc2.oeigv6v2appxkh6c \
    --discovery-token-ca-cert-hash sha256:e46f08a3df79c9548ba8d1a75b2bb10ec3e6b7a40af361ccd9c878342919c994
```

```shell
##此时查看node加入了
[root@k8smaster ~]# kubectl get nodes
NAME        STATUS     ROLES    AGE     VERSION
k8smaster   NotReady   master   9m27s   v1.18.0
k8snode1    NotReady   <none>   45s     v1.18.0
k8snode2    NotReady   <none>   37s     v1.18.0

```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下:

```shell
kubeadm token create --print-join-command
```

- 部署CNI网络插件(master节点)

```shell
[root@k8smaster ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
## 查看pods有没有运行
[root@k8smaster ~]# kubectl get pods -n kube-system
##如果查看到有那个pods运行异常，可以下载镜像
 docker pull quay.io/coreos/flannel:v0.12.0-amd64
 ##之后重写加载插件
 kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

- 重写查看节点，正常

```shell'
[root@k8smaster ~]# kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
k8smaster   Ready    master   11h   v1.18.0
k8snode1    Ready    <none>   11h   v1.18.0
k8snode2    Ready    <none>   11h   v1.18.0

```

- 测试k8s集群

在k8szhogn 创建一个pod，验证是否正常运行

```shell
## 拉取nginx镜像
[root@k8smaster ~]# kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
##等到nginx运行之后，对外暴露端口
[root@k8smaster ~]# kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
nginx-f89759699-nrg87   1/1     Running   0          77s
##
[root@k8smaster ~]# kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed
```

```shell
##查看nginx对外端口 31303
[root@k8smaster ~]#  kubectl get pod,svc
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-f89759699-nrg87   1/1     Running   0          4m7s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        13h
service/nginx        NodePort    10.110.148.58   <none>        80:31303/TCP   76s
```

访问：http://192.168.1.140:31303/

## 二进制搭建集群

### 准备三台服务器

1. 如上准备环境（初始化系统）

### 部署etcd集群

1. etcd是分布式键值存储系统，k8s使用他存储，所以首先搭建etcd数据库，使用3台集群，则可以容忍1台故障

2. 生成 SSL证书（在master上生成证书）

- 安装cfssl

```shell
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
```

- 生成 json模板

```shell
# 建立存放生成证书的地方
[root@k8sm ~]# mkdir -p /home/ssl
[root@k8sm ~]# cd /home/ssl/
```

- 编写脚本cerificate.sh，执行

```shell
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
         "expiry": "87600h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
              "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca -

#-----------------------

cat > server-csr.json <<EOF
{
    "CN": "kubernetes",
    "hosts": [
      "10.0.0.1",
	  "127.0.0.1",
	  "kubernetes",
	  "kubernetes.default",
	  "kubernetes.default.svc",
	  "kubernetes.default.svc.cluster",
	  "kubernetes.default.svc.cluster.local",
      "192.168.1.143",
      "192.168.1.144",
      "192.168.1.145"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server

#-----------------------

cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing",
      "ST": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin

#-----------------------

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing",
      "ST": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy

```

- 生成了这些文件

```shell
[root@k8sm ssl]# ll
total 72
-rw-r--r-- 1 root root 1009 Nov 20 23:58 admin.csr
-rw-r--r-- 1 root root  229 Nov 20 23:58 admin-csr.json
-rw------- 1 root root 1679 Nov 20 23:58 admin-key.pem
-rw-r--r-- 1 root root 1399 Nov 20 23:58 admin.pem
-rw-r--r-- 1 root root  294 Nov 20 23:58 ca-config.json
-rw-r--r-- 1 root root 1001 Nov 20 23:58 ca.csr
-rw-r--r-- 1 root root  266 Nov 20 23:58 ca-csr.json
-rw------- 1 root root 1679 Nov 20 23:58 ca-key.pem
-rw-r--r-- 1 root root 1359 Nov 20 23:58 ca.pem
-rwxrwxrwx 1 root root 2065 Nov 20 23:57 cerificate.sh
-rw-r--r-- 1 root root 1009 Nov 20 23:58 kube-proxy.csr
-rw-r--r-- 1 root root  230 Nov 20 23:58 kube-proxy-csr.json
-rw------- 1 root root 1675 Nov 20 23:58 kube-proxy-key.pem
-rw-r--r-- 1 root root 1403 Nov 20 23:58 kube-proxy.pem
-rw-r--r-- 1 root root 1062 Nov 20 23:58 server.csr
-rw-r--r-- 1 root root  354 Nov 20 23:58 server-csr.json
-rw------- 1 root root 1675 Nov 20 23:58 server-key.pem
-rw-r--r-- 1 root root 1436 Nov 20 23:58 server.pem

```

- 证书使用组价说明

| **组件**       | **使用的证书**                           |
| -------------- | ---------------------------------------- |
| etcd           | ca.pem server.pem server-key.pem         |
| flannel        | ca.pem server.pem server-key.pem         |
| kube-apiserver | ca.pem server.pem server-key.pem         |
| kubelet        | ca.pem ca-key.pem                        |
| kube-proxy     | ca.pem kube-proxy.pem kube-proxy-key.pem |
| kubectl        | ca.pem admin.pem admin-key.pem           |

3. 搭建etcd集群（三个节点都要配置）

- 软件包下载

下载地址：https://repo.huaweicloud.com/etcd/v2.3.1/etcd-v2.3.1-linux-amd64.tar.gz

- 创建部署环境

```shell
## 创建环境
mkdir /opt/k8s -p
mkdir /opt/k8s/{bin,conf,ssl}

##拷贝下载下来的可执行文件
[root@k8sm ~]# cp etcd-v2.3.1-linux-amd64/etcd /opt/k8s/bin/
[root@k8sm ~]# cp etcd-v2.3.1-linux-amd64/etcdctl /opt/k8s/bin/
```

- 创建配置文件,使用配置文件时需要将注释删除。

```shell
[root@k8sm ~]# vim /opt/k8s/conf/etcd.conf
```

```shell
#[Member]
## 每个节点都不同
ETCD_NAME="etcd01"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#集群通信端口
ETCD_LISTEN_PEER_URLS="https://192.168.1.143:2380"
#监听的数据端口
ETCD_LISTEN_CLIENT_URLS="https://192.168.1.143:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.143:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.1.143:2379"
#多个用,隔开
ETCD_INITIAL_CLUSTER="etcd01=https://192.168.1.143:2380,etcd02=https://192.168.1.144:2380,etcd03=https://192.168.1.145:2380"
#认证token
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#集群建立状态
ETCD_INITIAL_CLUSTER_STATE="new" 
```

- 创建启动文件,服务unit

```shell
[root@k8sm ~]# vi /usr/lib/systemd/system/etcd.service
```

```shell
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=-/opt/k8s/conf/etcd.conf
ExecStart=/opt/k8s/bin/etcd \
--name=${ETCD_NAME} \
--data-dir=${ETCD_DATA_DIR} \
--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
--advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
--initial-cluster=${ETCD_INITIAL_CLUSTER} \
--initial-cluster-token=${ETCD_INITIAL_CLUSTER} \
--initial-cluster-state=new \
--cert-file=/opt/k8s/ssl/server.pem \
--key-file=/opt/k8s/ssl/server-key.pem \
--peer-cert-file=/opt/k8s/ssl/server.pem \
--peer-key-file=/opt/k8s/ssl/server-key.pem \
--trusted-ca-file=/opt/k8s/ssl/ca.pem \
--peer-trusted-ca-file=/opt/k8s/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

- 将之前生成的证书cp到ssl文件夹中

```shell
[root@k8sm ssl]# cp {ca,server,server-key}.pem /opt/k8s/ssl/
```

- 重载system服务，启动etcd

```shell
[root@k8sm ssl]# systemctl daemon-reload
[root@k8sm ssl]# systemctl start etcd
## 查看节点状态
[root@k8sn2 ~]# systemctl status etcd.service
● etcd.service - Etcd Server
   Loaded: loaded (/usr/lib/systemd/system/etcd.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-11-29 09:10:58 EST; 1min 32s ago
```

- 其他两台node1，node2 也做同样操作

```shell
[root@k8sm ssl]# scp -r /opt/k8s/ root@k8sn1:/opt/
[root@k8sm ssl]# scp /usr/lib/systemd/system/etcd.service root@k8sn1:/usr/lib/systemd/system/
```

### 为apiserver自签证书

- 采用可信用ip列表

server-csr.json 这个文件是master使用的

```shell
[root@k8sm ssl]# pwd
/home/ssl
[root@k8sm ssl]# cat server-csr.json 
```

这些地址就是可信用的ip列表

```shell
[root@k8sm ssl]# cat  server-csr.json 
{
    "CN": "kubernetes",
    "hosts": [
      "10.0.0.1",
  "127.0.0.1",
  "kubernetes",
  "kubernetes.default",
  "kubernetes.default.svc",
  "kubernetes.default.svc.cluster",
  "kubernetes.default.svc.cluster.local",
      "192.168.1.143",
      "192.168.1.144",
      "192.168.1.145"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },

```

执行命令生成server相关证书(ps:上面已经生成了)

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
```



### 部署master组件

1. 从这个地址下载

https://gitee.com/mirrors/Kubernetes/blob/master/CHANGELOG/CHANGELOG-1.19.md

对应版本（1.19），下载[Server binaries](https://gitee.com/mirrors/Kubernetes/blob/master/CHANGELOG/CHANGELOG-1.19.md#server-binaries)中的[kubernetes-server-linux-amd64.tar.gz](https://dl.k8s.io/v1.19.4/kubernetes-server-linux-amd64.tar.gz)

ps：可以去华为镜像源中下载

2. 进入此目录获取相关文件

```shell
[root@k8sm k8s]# pwd
/opt/k8s
[root@k8sm k8s]# cp /home/kubernetes/server/bin/{kube-scheduler,kube-apiserver,kube-controller-manager,kubectl} bin/
## 检查
[root@k8sm k8s]# ls bin/
etcd  etcdctl  kube-apiserver  kube-controller-manager  kubectl  kube-scheduler

```

3. 复制证书到ssl中

将之前etcd生成的pem证书cp到对应目录

```shell
[root@k8sm ssl]# cp *.pem /opt/k8s/ssl/
```



3. 配置文件
   - KUBE_APISERVER_OPTS：日志
   - etcd-servers：etcd集群地址
   - bind-address：当前监听地址
   - secure-port：https端口
   - enable-bootstrap-token-auth：为同组中自动授权

```shell
cat > /opt/k8s/conf/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \
--v=2 \
--log-dir=/opt/k8s/logs \
--etcd-servers=https://192.168.1.143:2379,https://192.168.1.144:2379,https://192.168.1.145:2379 \
--bind-address=192.168.1.143 \
--secure-port=6443 \
--advertise-address=192.168.1.143 \
--allow-privileged=true \
--service-cluster-ip-range=10.0.0.0/24 \
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \
--authorization-mode=RBAC,Node \
--enable-bootstrap-token-auth=true \
--token-auth-file=/opt/k8s/conf/token.csv \
--service-node-port-range=30000-32767 \
--kubelet-client-certificate=/opt/k8s/ssl/server.pem \
--kubelet-client-key=/opt/k8s/ssl/server-key.pem \
--tls-cert-file=/opt/k8s/ssl/server.pem \
--tls-private-key-file=/opt/k8s/ssl/server-key.pem \
--client-ca-file=/opt/k8s/ssl/ca.pem \
--service-account-key-file=/opt/k8s/ssl/ca-key.pem \
--etcd-cafile=/opt/k8s/ssl/ca.pem \
--etcd-certfile=/opt/k8s/ssl/server.pem  \
--etcd-keyfile=/opt/k8s/ssl/server-key.pem \
--audit-log-maxage=30 \
--audit-log-maxbackup=3 \
--audit-log-maxsize=100 \
--audit-log-path=/opt/k8s/logs/k8s-audit.log"
EOF
```

```shell
cat > /opt/k8s/conf/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=false \
--v=2 \
--log-dir=/opt/k8s/logs \
--leader-elect=true \
--master=127.0.0.1:8080 \
--address=127.0.0.1 \
--allocate-node-cidrs=true \
--cluster-cidr=10.244.0.0/16 \
--service-cluster-ip-range=10.0.0.0/24 \
--cluster-signing-cert-file=/opt/k8s/ssl/ca.pem \
--cluster-signing-key-file=/opt/k8s/ssl/ca-key.pem \
--root-ca-file=/opt/k8s/ssl/ca.pem \
--service-account-private-key-file=/opt/k8s/ssl/ca-key.pem \
--experimental-cluster-signing-duration=87600h0m0s"
EOF
```

```shell
cat > /opt/k8s/conf/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--logtostderr=false \
--v=2 \
--log-dir=/opt/k8s/logs \
--leader-elect \
--master=127.0.0.1:8080 \
--address=127.0.0.1"
EOF
```



5. 自启脚本

```shell
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/opt/k8s/conf/kube-apiserver.conf
ExecStart=/opt/k8s/bin/kube-apiserver $KUBE_APISERVER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

```shell
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/opt/k8s/conf/kube-controller-manager.conf
ExecStart=/opt/k8s/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

```shell
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/opt/k8s/conf/kube-scheduler.conf
ExecStart=/opt/k8s/bin/kube-scheduler $KUBE_SCHEDULER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

6. 在conf目录下生成token

```shell
echo "`head -c 16 /dev/urandom | od -An -t x | tr -d ' '`,kubelet-bootstrap,10001,\"system:kubelet-bootstrap\"" > token.csv
```



6. 启动

```shell
[root@k8sm conf]# systemctl start kube-apiserver
#查看进程
[root@k8sm logs]# ps -ef | grep k8s
#设置开机自启
[root@k8sm logs]# for i in $(ls /opt/k8s/bin);do systemctl enable $i;done
```

ps:如果报错发现日志，可以查看系统日志

```shell
cat /var/log/messages|grep kube-apiserver|grep -i error
```

