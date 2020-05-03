# 初始化配置文件

```shell
## 配置用户和邮箱
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
#查看配置
git config --list
```

# 概念

## 区域

- 工作区

相当于我们本地的代码的一个库，我们在工作区新增删除文件

- 暂存区

在工作区做修改，就是到暂存区

- 版本库

最终代码提交的库

## 对象

- git对象（kv组成键值对，）

git核心部分是一个简单的键值对数据库

```shell
## 获取这个字符串的唯一hash
## --stdin 
E:\code\gitcode>echo "test content" | git hash-object --stdin
575b08edcc0046f2cdffe2819a7d55472da88013
# 加一个-w表示他向库中存入这个字符串，能够在object文件夹中看到对应文件
#57\5b08edcc0046f2cdffe2819a7d55472da88013
# 他将这个hash的头两位作为文件夹名，后面的作为文件名
E:\code\gitcode>echo "test content" | git hash-object --stdin -w
575b08edcc0046f2cdffe2819a7d55472da88013
## 查看目录
find .git/objects -type f
.git/objects/57/5b08edcc0046f2cdffe2819a7d55472da88013
# 获取hash内容
git cat-file -p 575b08edcc0046f2cdffe2819a7d55472da88013
"test content"

```

```shell
# 新增一个文件
echo "test v1" > testv1.txt
# 将这个文件交给git管理，存入库中
git hash-object -w ./testv1.txt
915c628f360b2d8c3edbe1ac65cf575b69029b61
```

我们往老文件新添加文件,发现新生成一个文件

```shell
echo "v2" > testv1.txt
git hash-object -w ./testv1.txt
8c1384d825dbbe41309b7dc18ee7991a9085c46e
find .git/objects -type f
.git/objects/57/5b08edcc0046f2cdffe2819a7d55472da88013
.git/objects/8c/1384d825dbbe41309b7dc18ee7991a9085c46e
.git/objects/91/5c628f360b2d8c3edbe1ac65cf575b69029b61

```

**object没保存文件名，只保存文件内容**，git对象是存内容的

- 树对象

树对象和暂存区有关系

git对象代表文件的一次次版本，树对象代表项目的一次次版本

它能解决文件名保存的问题，也允许我们将多个文件组织到一起

**暂存区的覆盖是按照文件名的覆盖**



![](../image/network/2020417234553.png)

文件模式：100644 普通文件

​					100755 可执行文件

​					120000 符号链接

```shell
# 查看git在暂存区的文件
git ls-files -s
# 生成暂存区，此时，暂存区文件还没有存入版本库
# --add:该文件并不存在暂存区，需要新增
# --cacheinfo:添加的文件位于git数据库中，而非当前目录，需要添加该选项（将版本库文件存入暂存区）
git update-index --add --cacheinfo 100644 e028ca8d1dfdfad8e058f854d5038725a5451a09 t.text
# 查看暂存区
git ls-files -s
100644 e028ca8d1dfdfad8e058f854d5038725a5451a09 0       t.text
# 生成暂存区快照，写入版本库，生成树对象
git write-tree
f684a60c3006b1adf54af3db093a82ad2ff70bec
# 查看版本库git对象类型
git cat-file -t f684a60c3006b1adf54af3db093a82ad2ff70bec
tree
#将第一个树对象合并
git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579

```

查看树对象内容，发现里面存的是git对象的hash值等

```shell
$ git cat-file -p f684a60c3006b1adf54af3db093a82ad2ff70bec
100644 blob e028ca8d1dfdfad8e058f854d5038725a5451a09    t.text

```

- 提交对象

提交对象是对树对象的封装

第二次提交需要-p 来加上他的父提交对象

所以，每个提交对象就能知道他的父对象是谁

```shell
# 提交树对象，生成提交对象
echo 'first commit' | git commit-tree f684a60c3006b1adf54af3db093a82ad2ff70bec
7731a2068aa38b8285421ef38ce4ee59bcc605aa
# 查看对象类型为提交对象
git cat-file -t 7731a2068aa38b8285421ef38ce4ee59bcc605aa
commit
# 查看对象内容，为树对象hash和其他属性信息
git cat-file -p 7731a2068aa38b8285421ef38ce4ee59bcc605aa
tree f684a60c3006b1adf54af3db093a82ad2ff70bec
author xiao xiao <you@example.com> 1587140341 +0800
committer xiao xiao <you@example.com> 1587140341 +0800

first commit
```

- 回滚

在提交对象找到对应版本的hash，直接跳过去就可以了

# 底层命令

- 初始化仓库

新建一个文件，在文件目录先执行,发现新建了一个git的隐藏目录，

```shell
E:\code\gitcode>git init
Initialized empty Git repository in E:/code/gitcode/.git/
```

```shell
## 目前包含客户端的服务端的钩子脚本(提交前提交后脚本等等)
- hooks
# 全局排除文件
- info
# 存储所有数据
- objects
# 存放分支指针
- refs
# 配置文件
config.
# 目前
HEAD
# 文件保存的暂存信息
index
```

# 高层命令

- git add . 

工作目录->版本库（git 对象是增量的）->暂存区

git hash-object -w 文件名（修改了多少文件，就执行多少次）

git update-index (更新暂存区)

```shell
echo 'aaaa' >t.txt

# 查看暂存区和版本库都没有
git ls-files -s
find .git/objects/ -type f

#执行add命令
git add .
# 版本库多出git对象
find .git/objects/ -type f
.git/objects/5d/308e1d060b0c387d452cf4747f89ecb9935851
# 暂存区多出数据
git ls-files -s
100644 5d308e1d060b0c387d452cf4747f89ecb9935851 0       t.txt
# git对象为刚才的文件内容
git cat-file -p 5d308e1d060b0c387d452cf4747f89ecb9935851
aaaa

```

- git commit 
  - git commit -a :可以将以记录的文件跳过暂存区直接提交，也就是省了add 这个操作

```shell
git commit -m 'fisrt'
## commit新生成两个对象，一个数对象，一个提交对象
find .git/objects/ -type f
.git/objects/20/e61b2a24b050f57547feec226ed385d500bb64
.git/objects/32/bfd7949246fb9067e8b070dfeb3eb478b3f932
.git/objects/5d/308e1d060b0c387d452cf4747f89ecb9935851

```

- 查看当前git状态，每次提交前最好看看

git status

- 查看已暂存和为暂存的更新

  - 当前哪些更新还没有暂存

  git diff

  - 当前哪些暂存的更新没有提交

  git diff --cached
  
- 删除

  - 相当于新增一个文件，删除一个文件
  - 如果在工作目录删除，则需要add操作
  - 如果直接 git rm 则省略了add操作，直接执行commit就可以了

- 更换名字

  - git mv  更换文件名

- 查看日志

```shell
# 翻页
git log
## 格式化，前面是提交对象hash
git log --pretty=oneline
9524e40369ec2430d28685d70e9523a93a7e3d67 (HEAD -> master) fisrt
32bfd7949246fb9067e8b070dfeb3eb478b3f932 fisrt
git log --oneline
9524e40 (HEAD -> master) fisrt
32bfd79 fisrt

```

# 分支操作

我们在refs/head/master里面可以看到提交对象的hash

所以master指向最新提交的提交对象

## 创建分支

```shell
#创建分支
git branch xiao1
#此时git head还指向master
git log --oneline
9524e40 (HEAD -> master, xiao1) fisrt
32bfd79 fisrt

```

## 切换分支

切换分支会修改：暂存区，工作目录和head

当我们有未提交的文件是，切换分支会将这个未提交的文件带过去

```shell
git checkout xiao1
#分支已切换
git log --oneline
9524e40 (HEAD -> xiao1, master) fisrt
32bfd79 fisrt
# 查看分支
git branch
  master
* xiao1

```

## 删除分支

- 不能自己删自己，必须切到其他分支

```shell
#先切换分支
git checkout master
#
git branch -d xiao1
#强制删除
git branch -D xiao1
```

## git分支历史

```shell
git log --oneline --decorate --graph --all
* 35dfc1a (xiao1) 1
* 9524e40 (HEAD -> master) fisrt
* 32bfd79 fisrt

```

## 配置命令别名

```shell
git config --global alias.t1 'log --oneline --decorate --graph --all'
```

## 新建分支指向对应的提交对象

```shell
## 查看历史
git t1
* 9524e40 (HEAD -> master) fisrt
* 32bfd79 fisrt
# 指向first那一次提交
git branch f 32bfd79
##查看已指向
git t1
* 9524e40 (HEAD -> master) fisrt
* 32bfd79 (f) fisrt
#切换分支
git checkout f

```

也可以直接新建直接切换到对应分支

```shell
git checkout -b f2 32bfd79
##查看
git t1
* 9524e40 (master) fisrt
* 32bfd79 (HEAD -> f2, f) fisrt
```

## 合并分支

### 快进合并

模拟案例

- 现在修改一个问题，创建一个分支iss1

```shell
# 在主分支上创建iss1的分支修改问题
git checkout -b iss1
#在分支iss1上新增文件，并提交
echo 'iss1'>iss1.txt
git add .
git commit -m 'iss1'
#可以看到新增iss1分支已经多出一个版本
git t1
* 9e1dc1e (HEAD -> iss1) iss1
* 9524e40 (master) fisrt
* 32bfd79 fisrt
```

- 分支iss1问题处理一半，现在有一个问题要在master紧急处理

```shell
#在master创建分支iss2
git checkout master
git checkout -b iss2
#能够看到现在head指向iss2
git t1
* 9e1dc1e (iss1) iss1
* 9524e40 (HEAD -> iss2, master) fisrt
* 32bfd79 fisrt
# 修改内容提交
vim t.txt
git add .
git commit -m 'iss2'
#内容修改完成，切回主分支
git checkout master
#可以看到现在有两个分支
git t1
* 77d31f1 (iss2) iss2
| * 9e1dc1e (iss1) iss1
|/
* 9524e40 (HEAD -> master) fisrt
* 32bfd79 fisrt
# 合并master和iss2分支，可以看到Fast-forward的标识，这就是快进合并
git merge iss2
Updating 9524e40..77d31f1
Fast-forward
 t.txt | 1 +
 1 file changed, 1 insertion(+)
#删除分支
git branch -d iss2

```

### 解决冲突

现在，iss1和master是两条并行分支了，如果修改iss1的文件，就会产生冲突

```shell
#切换到iss1分支
git checkout iss1
#修改t.txt文vim t.txt件
vim t.txt
git add .
git commit -m 'iss1 -v2'
# iss1修改完毕，切回master，合并分支
# 我们可以看到，它提示t.txt文件有冲突
git merge iss1
Auto-merging t.txt
CONFLICT (content): Merge conflict in t.txt
Automatic merge failed; fix conflicts and then commit the result.
# 编辑冲突文件
vim t.txt
# 提交文件，就解决冲突了
git add .
git commit -m 'merge iss1'
[master f98ae39] merge iss1
#查看，已解决
git t1
*   f98ae39 (HEAD -> master) merge iss1
|\
| * 6790539 (iss1) iss1 -v2
| * 9e1dc1e iss1
* | 77d31f1 iss2
|/
* 9524e40 fisrt
* 32bfd79 fisrt

```

## Git存储

当项目做到一半，有其他bug要做，当时这是本分支不想提交，因为提交会增加提交对象，这时就可以同Git存储

```shell
#切换到iss3分支
git checkout -b iss3
#编写文件，然后add，之后存储
vim iss3.txt
git add .
git stash

#切换主分支，做一系列操作
git checkout master

# 切换iss3分支
git checkout iss3
# 将存储的数据拉回来
git stash apply stash@{0}
#删除存储
git stash drop stash@{0}
#也可以用弹出的方式，进行恢复
git stash pop
```



## 后悔药

- 还没有add的文件进行恢复

```shell
 git checkout -- iss1.txt
```

- 已经add 存入暂存区的文件进行恢复（**这个HEAD可以是每次提交对象的hash**）

```shell
## 恢复暂存区文件
git reset HEAD iss1.txt
## 恢复工作区文件（将文件从暂存区拉回）
git checkout -- iss1.txt
```

## reset

- 移动head

先提交三次对象，模拟第三次提交对象出现失误，我们需要回退1个版本

提交回滚，代码不变，会退到某个未提交的状态（撤销提交）

```shell
#查看暂存区的对象
git cat-file -p 4b1d4d4b660c400875dd122de217e942f481a378
v1
v2
v3
# 指向回撤命令
git reset --soft HEAD~
#查看head指向，head指向指向了2，但是工作区和暂存区文件没变
git t1
* 2915d2f (HEAD -> master) 2
* 730c3c8 1
git reflog
2915d2f (HEAD -> master) HEAD@{0}: reset: moving to HEAD~
6cc231b HEAD@{1}: commit: 3
2915d2f (HEAD -> master) HEAD@{2}: commit: 2
730c3c8 HEAD@{3}: commit (initial): 1
##但是我们查看head
git cat-file -p HEAD
tree 38f199bf7a3bf6497a591f349d5d2ba241a549c3
parent 730c3c85352563f8bd63396709e1b22759f6cec4
author xiao xiao <you@example.com> 1587477993 +0800
committer xiao xiao <you@example.com> 1587477993 +0800
# 查看树对象里面的git对象的内容
git cat-file -p 38f199bf7a3bf6497a591f349d5d2ba241a549c3
100644 blob 2139d8be699ff388ab3ec85008a3906675ac67cb    t
git cat-file -p 2139d8be699ff388ab3ec85008a3906675ac67cb
v1
v2
#所以，这个命令只移动了head，相当于撤销了上次提交
```

- head到某个hash

```shell
git reset --soft 730c3c8
```

- head移动，并且带动暂存区，

```shell
git reset --mixed 2915d2f
#可以看到这个命令动了head，还动了暂存区
git cat-file -p 2139d8be699ff388ab3ec85008a3906675ac67cb
v1
v2
```

- head移动，改变暂存区和工作区(**别用**）

```shell
git reset --hard 730c3c8
```

- reset 带文件（**只会动暂存区**）

```shell
# t是文件名
git reset --mixed 2915d2f t
# 更改对应工作区的文件
git checkout -- t
```

**git如果发现那个提交有问题，可以checkout到对应提交，新增分支进行修改，然后与当前开发分支进行合并，而不要reset硬重置**

## checkout和reset区别

- checkout只懂head， --hard动head，还动分支
- checkout是工作区安全的（不删文件）

# 打tag

git可以给历史中的某一个提交打上标签以示重要，如**标记发布节点**

 ```shell
## 查看标签
git tag
 ```

- 轻量标签

很像一个不会改变的分支，只是一个特定提交的引用

```shell
git tag v1.1 730c3c8
##查看
git t1
* ec011a8 (HEAD -> master) a
* 6cc231b 3
* 2915d2f 2
* 730c3c8 (tag: v1.1) 1

```

- 查看标签

```shell
git show v1.1
```

- 检出标签

```shell
# 我们发现他提示处于头部分离标识
git checkout v1.1
You are in 'detached HEAD' state.
#我们需要创建分支
git checkout -b 'v11'
```

# 远程仓库

## 初始化远程仓库

新建远程仓库，在本地目录初始化

```shell
# 当我们提交远程仓库后，发现多了一个远程仓库的名称
git t1
* f0b493a (HEAD -> master, testgit/master) first commitgit init
#新建文件
touch README.md
git add README.md
git commit -m "first commit"
#配置远程仓库别名
git remote add origin git@gitee.com:aloneDr/testgit.git
git push -u origin master
## 查看别名
git remote -v
origin  git@gitee.com:aloneDr/testgit.git (fetch)
origin  git@gitee.com:aloneDr/testgit.git (push)

```

当我们提交远程仓库后，发现多了一个远程仓库的名称，这叫跟踪分支

```shell
git t1
* f0b493a (HEAD -> master, testgit/master) first commit
```

## 从远程仓库克隆

```shell
#默认clone下来的别名为origin
git clone http://192.168.1.134/lead/testgit.git
```

- 从远处仓库拿文件

```shell
# 将远程文件拉到testgit/master分支下
git fetch testgit
git t1
* b932ec3 (testgit/master) Add new file
* f0b493a (HEAD -> master) first commit
#合并分支
git merge testgit/master
```

- 一个本地分支，怎么去跟踪一个远程跟踪分支

```shell
# 新建其他分支，指定想要跟踪的远程跟踪分支， testgit/master是我们的远程分支名
git checkout -b testgit testgit/master
# 也可以将当前分支指定远程跟踪分支
git branch -u testgit/master
#只要处于当前跟踪分支，那么就能在这个分支使用push
git branch -vv
  master b932ec3 [testgit/master] Add new file
* ttt    b932ec3 [testgit/master] Add new file

```

## 解决冲突

```shell
# 在a文件夹创建分支
 git checkout -b comment
# 将分支推送到远程
git push origin comment
```

```shell
# 在b文件夹下面
git fetch origin comment
# 建立本地分支关联远程分支
git checkout --track origin/comment
git branch -vv
* comment b932ec3 [origin/comment] Add new file
  master  b932ec3 [origin/master] Add new file
#这个时候想拿comment分支数据，可以
git pull
```

- push有冲突

```shell
#在b文件夹有冲突时
 git push
To http://192.168.1.134/lead/testgit.git
 ! [rejected]        comment -> comment (fetch first)
# 我们pull下来，发现合并内容
git pull
CONFLICT (content): Merge conflict in init.txt

#解决冲突
vim init.txt
git add .
git commit -m 'merge'
#这个时候再push上去
git push
```

- 当没有commit时，pull可能会产生冲突

## 删除远程分支



## pull request

如果你想要参与某个项目，当时没有推送权限，这时可以对这个项目进行“派生”(fork)，派生：jiang 在自己的空间中创建一个你的项目副本

```shell
#先fork一个项目， 修改后push到fork的项目中
#提交之后再fork库点击merge request
```

```shell
#收到通知在主项目
git fetch http://192.168.1.134/dev/testgit.git comment
git checkout -b dev/testgit-comment FETCH_HEAD
#进行合并
```

# ssh协议

本地生成公钥,将pub文件添加

```shell
ssh-keygen -t rsa -C xiao@qq.com
#按照提示，三次回车，不用输入什么
```

进行clone

```shell
git clone git@192.168.1.134:lead/testgit.git
```



# gitlab安装

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

邮件发送方式

```shell
#开启postfix
[root@localhost ~]# systemctl start postfix
[root@localhost ~]# systemctl enable postfix
```



## yum安装方式

```shell
##下载gitlab
[root@localhost ~]# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
#安装gitce
[root@localhost ~]# yum -y install gitlab-ce 
```

## rpm方式安装

```shell
rpm -ivh gitlab-ce-10.2.3-ce.0.el7.x86_64.rpm
```

此时，gitlab安装到了**/opt/gitlab/**下面

```shell
[root@localhost gitlab]# cd /opt/gitlab/
[root@localhost gitlab]# ls
bin                       etc      LICENSES  version-manifest.json
dependency_licenses.json  init     service   version-manifest.txt
embedded                  LICENSE  sv
```

如果要修改配置文件，修改此处

```shell
vim /etc/gitlab/gitlab.rb
```

## 将gitlab设置成https访问模式