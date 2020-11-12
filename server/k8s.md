# 基础部分

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
## 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config
## 关闭 swap
sed -ri 's/.*swap.*/#&/' /etc/fstab  
## 设置规划主机名
[root@localhost ~]# hostnamectl set-hostname k8smaster
# 查看
[root@localhost ~]# hostname
k8smaster
# 在master节点上添加host
cat >> /etc/hosts <<EOF 
192.168.1.140 master
192.168.1.141 node1
192.168.1.142 node2
EOF
## 对网络进行设置
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

