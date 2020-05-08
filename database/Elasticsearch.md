# 功能

- 分布式的搜索引擎

如：百度等

- 全文检索，结构化检索
  - 文章的关键字检索
  - 电商的分类查询

# 核心概念

- Document&field

文档，es中的最小数据单元，一个document可以是一条客户数据，一条商品分类数据，一条订单数据，通常用JSON数据结构表示

- type

每个索引都有多个或者一个type，一个type下的document，有着相同的字段

- index:索引

包含一堆有相似结构文档的数据，如：订单索引

| es       | 数据库 |
| -------- | ------ |
| document | 行     |
| type     | 表     |
| index    | 数据库 |

- share

如果一个index有3t的数据，它把他分为三份，每一个share存1t，这样，查询就增加了速度

share是最小的工作单元

每个share其实就是一个Lucene实例

share分为primary share和replica share，每个document只能存在于某个primary share以及对应的replica share中

- replica share

其实就是share的一个副本

优点：查询的时候，也可以查到replica上面

​			share挂了，可以让replica顶上

**share不能和replica同一个服务器，所以es一般两个服务器以上**

- master

master 选举，将一个node变为master

新master间隔primary share的replica 变为primary

# 特性

- 对复杂的分布式机制的透明隐藏特性
  - 我们不需要考虑数据怎么进行分片，数据分配到了哪个shard占用，
  - 集群可以自己发现node
  - shard复杂均衡

- 垂直扩容和水平扩容，扩容对应用程序透明
  - 垂直扩容：购置强大的服务器，将新的服务器代替老服务器
  - 水平扩容：购置服务器，加入老的集群之中
- 增加或者减少节点，会自动将数据平衡
- master节点
  - 创建、删除索引；增加删除节点
  - 默认自动选择一台节点作为master
- 节点平等的分布式架构
  - 每个节点都可以接受所有请求
  - 可能请求a share，但a可以发给b，让b给数据给a，最后返回

# 安装

## 安装ES

国内镜像

<https://thans.cn/mirror/elasticsearch.html>

修改配置文件

```yaml
cluster.name: myes
node.name: node-1
cluster.initial_master_nodes: ["node-1"]
network.host: 192.168.1.131
```

可能需要修改

```shell
[root@localhost ~]# vim /etc/security/limits.conf
myes soft nofile 65536
myes hard nofile 100000
[root@localhost ~]# vim /etc/sysctl.conf 
vm.max_map_count=655360

```



```shell
#建立myes用户，es要非root用户启动
[root@localhost home]# adduser myes
[root@localhost home]# passwd myes

[root@localhost home]# chown -R myes:myes elasticsearch-7.3.2
#进入myes启动
[root@localhost home]# su myes
[myes@localhost home]$ ./elasticsearch-7.3.2/bin/elasticsearch
```

访问<http://192.168.1.131:9200/>

## 安装Kibana

国内镜像

<https://www.newbe.pro/Mirrors/Mirrors-Kibana/#toc-heading-1>

修改配置文件

```shell
[root@localhost ~]# vim /home/kibana-7.3.2/config/kibana.yml 
```

```yaml
server.host: "192.168.1.131"
elasticsearch.hosts: ["http://192.168.1.131:9200"]
```

启动

```shell
myes@localhost home]$ ./kibana-7.3.2/bin/kibana
```

访问dev_tool

<http://192.168.1.131:5601/app/kibana#/dev_tools/console?_g=()>

## 横向扩容

扩容之后，当个节点的share变少，性能变得更好

---

如果只有6个share（三个primary， 三个replica），怎么突破瓶颈（极限：有多少个share，就多少个机器），增加到9台机器

增加replica share，因为primary share是不能变得， 如：replica=2,

---



# 简单的CRUD

## 简单的集群管理

- 快速检查进群的健康状态

```
GET /_cat/health?v
```

- 查看集群index

```
GET /_cat/indices?v
```

## 新增操作

```
PUT /index/type/documentid
{
	json 数据
}
```

新增一条数据
```shell
PUT /order/product/1
{
  "name":"iphone",
  "price":"3000",
  "desc": "Simple to use and cheap to use"
}
```

不指定documentid 新增(采用GUID算法生成)

```shell
POST /order/product/
{
  "name":"phone",
  "price":"2000",
  "ooprice":2000,
  "desc": "Simple to use and cheap to use"
}
```

- PUT 和POST用法

```
PUT是幂等方法，POST不是。
– PUT，DELETE操作是幂等的。所谓幂等是指不管进行多少次操作，结果都一样。比如我用PUT修改一篇文章，然后在做同样的操作，每次操作后的结果并没有不同，DELETE也是一样。
– POST操作不是幂等的，比如常见的POST重复加载问题：当我们多次发出同样的POST请求后，其结果是创建出了若干的资源。
– 还有一点需要注意的就是，创建操作可以使用POST，也可以使用PUT，区别在于POST是作用在一个集合资源之上的（/articles），而PUT操作是作用在一个具体资源之上的（/articles/123），比如说很多资源使用数据库自增主键作为标识信息，而创建的资源的标识信息到底是什么只能由服务端提供，这个时候就必须使用POST。
```

## 修改操作

- 替换的方式

这有一点不好，就是需要将所有字段都带上

```
PUT /order/product/1
{
  "name":"iphone",
  "price":"4000",
  "desc": "Simple to use and cheap to use"
}
```

返回

```json
"_index" : "order",
"_type" : "product",
"_id" : "5", //可以不手动设置，es会自动设置
```

- 修改的方式（post）

```
POST /order/product/1/_update
{
  "doc":{
    "name":"huawei iphone"
  }
}
```

## 查询

```
GET /order/product/1
```

## 删除

不是物理删除，只是标记，如果数据越来越多，则会后台自动物理删除

```shell
DELETE /order/product/1
```

# 多重查询方式

## search

- 查询全部

```shell
GET /order/product/_search
```

返回参数解释

took： 消耗时间

timed_out:是否超时

hits.total : 查询结果数量

hits.max_score: 相关度匹配

- 查询全部(带查询条件)

```shell
GET /order/product/_search?q=name:iphone
关键字必须包含
GET /order/product/_search?q=name:+iphone
关键字必须不包含
GET /order/product/_search?q=name:-iphone
任意的字段都包含关键字
GET /order/product/_search?q=iphone
```

- 默认的情况下是没有timeout的，所以如果查询时间慢，那么会一直等待，timeout机制，指定每个shard，在指定的时间内，返回搜索到的数据

```json
GET /_search?timeout=10m
```



## query DSL

就是将条件放入body中

- 查询全部

```
GET /order/product/_search
{
  "query":{
    "match_all": {}
  }
}
```

- 分页

```
GET /order/product/_search
{
  "query":{
    "match_all": {}
  },
  "from": 0,
  "size": 2
}
```

- 带条件查询

```
GET /order/product/_search
{
  "query":{
    "match": {
      "name":"iphone"
    }
  }
}
```

- 查指定字段

只包含name，price两个字段

```
GET /order/product/_search
{
  "query":{
    "match": {
      "name":"iphone"
    }
  },
  "_source":["name", "price"]
}
```

## query filter

must:必须满足

filter： 过滤要求

```shell
GET /order/product/_search
{
  "query":{
    "bool": {
      "must": [
        {"match":{"name": "iphone"}}
      ],
      "filter": [
      {"range":{"price":{"gt":"3000"}}}  
      ]
    }
  }
}
```

- 全文检索

他会将搜索的词进行拆分，然后在进行匹配

```shell
GET /order/product/_search
{
  "query":{
    "match": {
      "desc":"simple use"
    }
  }
}
```

- phrase query（短语检索）

搜索的词不会进行拆分,并且对desc字段进行高亮

```json
GET /order/product/_search
{
  "query":{
    "match_phrase": {
      "desc":"simple"
    }
  },
  "highlight":{
    "fields": {
      "desc": {}
    }
  }
}
```

## 分组查询

- 查询数量

aggs：聚合

group_by_price:给聚合起个名字

size：指定size的个数，默认为10，即返回10条聚合查询结果

结果按照聚合信息返回price的数量

```json
GET /order/product/_search
{
  "size": 0,
  "aggs":{
    "group_by_price":{
      "terms": {
        "field": "price"
      }
    }
  }
}
```

执行完后会报错： Set fielddata=true on [price] ，这个时候，我们需要修改这个字段的属性

```json
PUT /order/_mapping
{
  "properties":{
    "price":{
      "type":"text",
      "fielddata":true
    }
  }
}
```

- 先分组，再计算查询平均值（嵌套聚合）

先按照name分组，然后在按照ooprice进行算平均值，然后在按照平均值进行排序

```json
GET /order/product/_search
{
  "size": 0,
  "aggs":{
    "group_by_price":{
      "terms": {
        "field": "name",
        "order": {
          "avg_price": "desc"
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "ooprice"
          }
        }
      }
    }
  }
}
```

## 批量查询

```json
GET /_mget
{
  "docs":[
    {"_index": "order", "_type": "product", "_id": "1"}
  ]
}
```

或者查同一个index下

```json
GET /order/_mget
{
  "docs":[
    {"_type": "product", "_id": "1"}
  ]
}
```

# 批量增删改

1）delete:删除，只需要一个json串

2）create 相当于put  /_create， 强制创建

3）index 相当于put， 可创建可全量替换

4)  update 相当于 partial update， 修改

**除了delete，其他语法必须两个json串**

{“action”: {metadata}}

{data}

**bulk api 每个json串不能换行，但json串和json串之间，必须换行**,执行过程，有一个报错，不影响其他执行

```json
POST /_bulk
{"delete":{"_index" : "order","_type" : "product","_id" : "1"}}
{"create":{"_index" : "order_test","_type" : "product","_id" : "1"}}
{"name":"test_create_bulk"}
```



# 并发处理方案

ES中用的是乐观锁并发方案

每次修改，ES都会去修改_version

如果A线程操作数据，version=1，回写数据后version=2

B线程修改数据，version=1，回写数据发现version不相等，则会将这条数据扔掉，不会让后修改的数据覆盖

## 乐观锁

A线程操作数据，A拿到vesion1的数据，B线程拿到的也是vesion1的数据

A线程修改数据后，判断库中vesion与自己拿到的数据是否一致，如果一致则回写成功，数据vesion+1

B线程修改数据，回写发现vesion不一致，重新读取数据，再次操作，判断版本号，进行回写

## 悲观锁

A线程操作数据，其他线程无法获取数据，其他线程阻塞，直到A线程操作完并且将数据回写之后

## ES乐观锁处理

- 手动处理

如果执行失败，则我们需要去重新获取version,再次执行，_version=1时，修改才能成功

```json
PUT /order/product/4?version=1
```

- 只有当你提供的version比es中的_version大的时候，才能完成修改

```json
?version=1&version_type=external
```

- partial update会内置乐观锁,下面的语句表示乐观锁重试5次

```json
post、index/type/id/_update?retry_on_conflict=5
```

## ES写一致性原理

consistency，one（primary shard），all（all shard），quorum（default）

- 我们在发送任何一个增删改操作的时候，比如说put /index/type/id，都可以带上一个consistency参数，指明我们想要的写一致性是什么？

```bash
put /index/type/id?consistency=quorum
```

one：要求我们这个写操作，只要有一个primary shard是active活跃可用的，就可以执行
 all：要求我们这个写操作，必须所有的primary shard和replica shard都是活跃的，才可以执行这个写操作
 quorum：默认的值，要求所有的shard中，必须是大部分的shard都是活跃的，可用的，才可以执行这个写操作

# 深入查询

## 搜索模式

告诉你如何一次性搜索多个index和多个type下的数据

/_search：所有索引，所有type下的所有数据都搜索出来
/index1/_search：指定一个index，搜索其下所有type的数据
/index1,index2/_search：同时搜索两个index下的数据
/*1,*2/_search：按照通配符去匹配多个索引
/index1/type1/_search：搜索一个index下指定的type的数据
/index1/type1,type2/_search：可以搜索一个index下多个type的数据
/index1,index2/type1,type2/_search：搜索多个index下的多个type的数据
/_all/type1,type2/_search：_all，可以代表搜索所有index下的指定type的数据

## 分页搜索

从第二页查询，每页2条数据

```json
GET /order*/_search?from=2&size=2
```

- deep paging问题（深度分页）

# mapping

当我们PUT /order/product/1一条数据时，es会自动给我们建立一个dynamic mapping，里面包括了分词或者搜索的行为

在es中，搜索分两种

-  exact value
  - 搜索的时候，搜索词必须全部匹配，才能搜索出来
- full text （全文检索）
  - 缩写：如cn=china
  - 格式转化: 日like，也可以将likes搜索出来
  - 大小写
  - 同义词：如果like，也可以将love搜索出来