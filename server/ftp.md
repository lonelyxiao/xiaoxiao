# 安装

yum -y install vsftpd
chkconfig vsftpd on

- 添加用户

```shell
[root@localhost ftpdata]# adduser -d /data/ftpdata/rlzy/ ftprlzy -s /sbin/nologin

## 新建一个配置文件
# vim /etc/vsftpd/virtusers
username
passworld
## 生成用户数据文件
db_load -T -t hash -f /etc/vsftpd/virtusers /etc/vsftpd/virtusers.db

```

