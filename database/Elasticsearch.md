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

### 文件目录

- config
  - log4j2 日志文件
  - jvm 
  - elasticsearch.yml 配置文文件

修改配置文件

```yaml
## 集群名字，同一个集群必须相等
cluster.name: myes
## 当前节点名字
node.name: node-1
cluster.initial_master_nodes: ["node-1"]
network.host: 192.168.1.131

## 解决跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"
```

可能需要修改

**注：这一步是为了防止启动容器时，报出如下错误：bootstrap checks failed max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]**

```shell
[root@localhost ~]# vim /etc/security/limits.conf
myes soft nofile 65536
myes hard nofile 100000
[root@localhost ~]# vim /etc/sysctl.conf 
vm.max_map_count=655360
##执行让其生效
[root@localhost ~]#sysctl -p
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

### elasticsearch-.yml详解

- 6.x

```yaml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
# Before you set out to tweak and tune the configuration, make sure you
# understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please see the documentation for further information on configuration options:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration.html>
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称，默认是elasticsearch
# cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称，默认从elasticsearch-2.4.3/lib/elasticsearch-2.4.3.jar!config/names.txt中随机选择一个名称
# node.name: node-1
#
# Add custom attributes to the node:
#
# node.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
# 可以指定es的数据存储目录，默认存储在es_home/data目录下
# path.data: /path/to/data
#
# Path to log files:
# 可以指定es的日志存储目录，默认存储在es_home/logs目录下
# path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
# 锁定物理内存地址，防止elasticsearch内存被交换出去,也就是避免es使用swap交换分区
# bootstrap.memory_lock: true
#
#
#
# 确保ES_HEAP_SIZE参数设置为系统可用内存的一半左右
# Make sure that the `ES_HEAP_SIZE` environment variable is set to about half the memory
# available on the system and that the owner of the process is allowed to use this limit.
#
# 当系统进行内存交换的时候，es的性能很差
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
#
# 为es设置ip绑定，默认是127.0.0.1，也就是默认只能通过127.0.0.1 或者localhost才能访问
# es1.x版本默认绑定的是0.0.0.0 所以不需要配置，但是es2.x版本默认绑定的是127.0.0.1，需要配置
# Set the bind address to a specific IP (IPv4 or IPv6):
#
# network.host: 192.168.0.1
#
#
# 为es设置自定义端口，默认是9200
# 注意：在同一个服务器中启动多个es节点的话，默认监听的端口号会自动加1：例如：9200，9201，9202...
# Set a custom port for HTTP:
#
# http.port: 9200
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html>
#
# --------------------------------- Discovery ----------------------------------
#
# 当启动新节点时，通过这个ip列表进行节点发现，组建集群
# 默认节点列表：
# 127.0.0.1，表示ipv4的回环地址。
# [::1]，表示ipv6的回环地址
#
# 在es1.x中默认使用的是组播(multicast)协议，默认会自动发现同一网段的es节点组建集群，
# 在es2.x中默认使用的是单播(unicast)协议，想要组建集群的话就需要在这指定要发现的节点信息了。
# 注意：如果是发现其他服务器中的es服务，可以不指定端口[默认9300]，如果是发现同一个服务器中的es服务，就需要指定端口了。
# Pass an initial list of hosts to perform discovery when new node is started:
#
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
# discovery.zen.ping.unicast.hosts: ["host1", "host2"]
#
#
#
#
# 通过配置这个参数来防止集群脑裂现象 (集群总节点数量/2)+1
# Prevent the "split brain" by configuring the majority of nodes (total number of nodes / 2 + 1):
#
# discovery.zen.minimum_master_nodes: 3
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html>
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
# 一个集群中的N个节点启动后,才允许进行数据恢复处理，默认是1
# gateway.recover_after_nodes: 3
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-gateway.html>
#
# ---------------------------------- Various -----------------------------------
# 在一台服务器上禁止启动多个es服务
# Disable starting multiple nodes on a single system:
#
# node.max_local_storage_nodes: 1
#
# 设置是否可以通过正则或者_all删除或者关闭索引库，默认true表示必须需要显式指定索引库名称
# 生产环境建议设置为true，删除索引库的时候必须显式指定，否则可能会误删索引库中的索引库。
# Require explicit names when deleting indices:
#
# action.destructive_requires_name: true
```

- 7.x

```yaml
cluster.name: elasticsearch-cluster
node.name: es-node3
network.bind_host: 0.0.0.0
network.publish_host: 192.168.1.134
http.port: 9202
#内部节点通信端口
transport.tcp.port: 9302
#跨域
http.cors.enabled: true
http.cors.allow-origin: "*"
#集群节点
discovery.seed_hosts: ["192.168.1.134:9300","192.168.1.134:9301", "192.168.1.134:9302"]
#有资格成为主节点的节点配置
cluster.initial_master_nodes: ["es-node1","es-node2","es-node3"]

```



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
#语音为中文
i18n.locale: "zh-CN"
```

启动

```shell
myes@localhost home]$ ./kibana-7.3.2/bin/kibana
```

访问dev_tool

<http://192.168.1.131:5601/app/kibana#/dev_tools/console?_g=()>

## 可视化HEAD





## 横向扩容

扩容之后，当个节点的share变少，性能变得更好

---

如果只有6个share（三个primary， 三个replica），怎么突破瓶颈（极限：有多少个share，就多少个机器），增加到9台机器

增加replica share，因为primary share是不能变得， 如：replica=2,

---



# 简单的CRUD

**在7.X后，官方废弃type,默认type为_doc**

故：
ES 的Type 被废弃后，库表合一，Index 既可以被认为对应 MySQL 的 Database，也可以认为对应 table。

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

- 带查询方式的删除

```json
POST /sys_org_company/_delete_by_query
{
   "query": {
        "match_all": {
        }
    }
}
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

- 多个组合条件搜索



- range

```json
GET /order/_search
{
  "query": {
    "range": {
      "ooprice": {
        "gte": 1000,
        "lte": 2000
      }
    }
  }
}
```

## query filter

must:必须满足

should：里面有一个条件满足就可以

filter： 过滤要求

filter只会过滤数据，不会计算相关度，所以，性能要高些

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

当然，如果想一个字段多个条件，也可以用空格隔开，只要有一个结果，就会被查出来

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

## 字段不进行分词

**term**表示字段不能进行分词，一定要全部匹配

```json
GET /order/_search
{
  "query": {
    "term": {
      "desc": {
        "value": "Simple to use"
      }
    }
  }
}
```

## 不合法查询定位

```json
GET /order/_search?explain
{
  "query": {
    "range1": {
      "ooprice": {
        "gte": 1000,
        "lte": 2000
      }
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

## 如何定制排序

默认情况下，是按照score排序的

sort可以定制查询

```shell
GET /order/_search?explain
{
  "query": {
    "match": {
      "desc": "use"
    }
  }
  , "sort": [
    {
      "_id": {
        "order": "desc"
      }
    }
  ]
}
```

## 对已经分词的字符串排序

默认情况下，排序会按照分词后的某个词来排序

如果我们想安装完整的字符串排序，可以建立两个field，一个分词用来搜索，一个不分词用来排序

建立：

"fields": {
            "raw": {
              "type": "string",
              "index": "not_analyzed"
            }
          },
          "fielddata": true

来进行不分词

```json
PUT /website 
{
  "mappings": {
    "article": {
      "properties": {
        "title": {
          "type": "text",
          "fields": {
            "raw": {
              "type": "string",
              "index": "not_analyzed"
            }
          },
          "fielddata": true
        },
        "content": {
          "type": "text"
        },
        "post_date": {
          "type": "date"
        },
        "author_id": {
          "type": "long"
        }
      }
    }
  }
}
```

查询的时候

```json
GET /website/article/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "title.raw": {
        "order": "desc"
      }
    }
  ]
}

```

## bouncing results问题

## 使用scoll滚动查询大量数据

使用scoll搜索一批又一批数据

```
GET /test_index/test_type/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": [ "_doc" ],
  "size": 3
}
```

第二次搜去带scorell_id

```json
GET /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : ""
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



# ES REST风格总结

| method | URL                    | DESC                            |
| ------ | ---------------------- | ------------------------------- |
| PUT    | /index/type/id         | 创建文档（指定文档id）/替换文档 |
| POST   | /index/type            | 创建文档（随机文档id）          |
| POST   | /index/type/id/_update | 修改文档（修改某个字段）        |
| POST   | /index/type/_search    | 查询所有数据                    |
| DELETE | /index/type/id         | s删除                           |
| GET    | /index/type/id         | c查询                           |



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

# mapping（映射规则）

当我们PUT /order/product/1一条数据时，es会自动给我们建立一个dynamic mapping，里面包括了分词或者搜索的行为,mapping会自定义每个field的数据类型

在es中，搜索分两种

-  exact value
  - 搜索的时候，搜索词必须全部匹配，才能搜索出来
- full text （全文检索）
  - 缩写：如cn=china
  - 格式转化: 日like，也可以将likes搜索出来
  - 大小写
  - 同义词：如果like，也可以将love搜索出来

分词之后，会将exact 或full建立到倒排索引中(**不同类型的filed会有不同的搜索类型**)

搜索时候，搜索的词根据的类型，进行分词，搜索到对应的doc中

## 字段类型

text会被分词解析，keyword不会被分词

## 倒排索引分析

通俗的说，就是通过value找key 首先，会分词，分词后做同义词等处理，这时，查找时，先搜索到value，再找到key

## 测试分词

```json
GET /_analyze
{
  "analyzer": "standard", //分词类型
  "text": ["test the analyze"] //待分的词
}
```

返回

```json
{
  "tokens" : [
    {
      "token" : "test",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "the",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "analyze",
      "start_offset" : 9,
      "end_offset" : 16,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
```

## 手动建立mapping

**只能创建index时手动建立mapping，或者新增field mapping，但是不能update field mapping**

type:字段类型

analyzer;分词类型

"index": false 不分词

```json
PUT /website
{
  "mappings": {
    "properties": {
       "author_id": {
          "type": "long"
        },
        "title": {
          "type": "text",
          "analyzer": "english"
        },
        "content": {
          "type": "text"
        },
        "post_date": {
          "type": "date"
        },
        "publisher_id": {
          "type": "text",
          "index": false
        }
    }
  }
}
```

新增已存在的mapping字段

```json
PUT /website/_mapping
{
  "properties":{
    "name" : {
        "type" : "text"
    }
  }
}
```

测试建立的mapping

```JSON
GET /website/_analyze
{
  "field": "title",
  "text": "my-dogs" 
}
```

## dynamic mapping

## 手动创建索引

创建一个my_index的索引

shard数量为1，副本数量为0

```json
PUT /my_index
{
  "settings": {
    "number_of_shards": 1
    , "number_of_replicas": 0
  }
  , "mappings": {
    "properties": {
      "my_field":{
        "type": "text"
      }
    }
  }
}
```

## 自定义dynamic策略

**elasticsearch7默认不在支持指定索引类型，默认索引类型是_doc**

true：遇到陌生字段，就进行dynamic mapping
false：遇到陌生字段，就忽略
strict：遇到陌生字段，就报错

```json
PUT /my_index
{
  "mappings": {
    "dynamic": "strict",
      "properties": {
        "name":{
          "type": "text"
        }
      }
  }
}
```

- put数据，如果超过字段就会报错(mapping set to strict)

```json
PUT /my_index/_doc/1
{
  "name":"xiaoxiao"
}
```



## 定制mapping策略

- date_detection

默认会按照一定格式识别date，比如yyyy-MM-dd。但是如果某个field先过来一个2017-01-01的值，就会被自动dynamic mapping成date，后面如果再来一个"hello world"之类的值，就会报错。可以手动关闭某个type的date_detection，如果有需要，自己手动指定某个field为date类型。

```json
PUT /my_index
{
  "mappings": {
    "date_detection": false
  }
}
```

- 通配符来匹配不同的模板

当某个字段是*en时，用下面这个模板，分词用English

```json
PUT /my_index
{
  "mappings": {
    "dynamic_templates":[
      {
        "en":{
          "match":"*en",
          "match_mapping_type":"string",
          "mapping":{
            "type":"text",
            "analyzer":"english"
          }
        }
      }  
    ] 
  }
}
```

插入数据

```json
PUT /my_index/_doc/1
{
  "name_en":"my name is xiaoxiao",
  "name":"my name is xiaoxiao"
}
```

 如果查询，name_en，is是停用词，是查询不到的，而name是可以查询到的

## 修改索引setting

## 定制自己的分词器

默认的分词器

standard

standard tokenizer：以单词边界进行切分
standard token filter：什么都不做
lowercase token filter：将所有字母转换为小写
stop token filer（默认被禁用）：移除停用词，比如a the it等等

## 修改分词器

```json
PUT /my_indexs
{
  "settings": {
    "analysis": {
      "es_sta":{
        "type": "standard",
         "stopwords": "_english_"
      }
    }
  }
}
```

## 重建索引

索引是不能修改的，如果想要修改，。则需要重新建立索引，然后将旧索引数据导入新的索引

- 建立新索引

```json
PUT /my_index_new
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      }
    }  
  }
}
```

- 采用scoll查询出批量数据，然后再采用bulk将数据批量插入

- 先给新索引取别名，删除旧索引的别名，让java应用能无缝切换新索引

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "my_index_new",
        "alias": "goodindex"
      },
      "remove": {
        "index": "my_index",
        "alias": "goodindex"
      }
    }
  ]
}
```



# TF/IDF算法相关度评分

- 搜索的词条在文本中出现的次数越多，相关度越高
- 搜索的的词条，在整个索引中，所有文档中，次数越多，越不相关



# 优化

数据写入os cache，并被打开搜索的过程，叫做refresh，默认是1秒

如果我们对数据的时效性要求比较低，那么可以时间设长

```json
PUT /my_index
{
  "settings": {
    "refresh_interval": "30s" 
  }
}

```

fsync+清空translog，将os cache数据写入disk，这就是flush，默认30异常，或者translog过大执行一次

tanslog每隔5s会写入磁盘，当且是同步的

如果，我我们允许部分数据丢失，可以设置异步的写入

```json
PUT /my_index/_settings
{
    "index.translog.durability": "async",
    "index.translog.sync_interval": "5s"
}
```

# java 操作

- 引入pom

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

## insert

```json
 public static void main(String[] args) throws Exception {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("192.168.1.131", 9200, "http")));

        Test test = new Test();
        test.sourceIndex(client);
        client.close();
    }

    public IndexResponse sourceIndex(RestHighLevelClient client) {
        try{
            IndexRequest request = new IndexRequest("post");
            Map map = new HashMap();
            map.put("user", "laoxiao");
            request.index("test_index").id("1").source(map, XContentType.JSON);
            IndexResponse response = client.index(request, RequestOptions.DEFAULT);
            return response;
        }catch (Exception e){
        }
        return null;
    }
```

# ELK

ELK是Elasticsearch、Logstash、Kibana的简称，这三者是核心套件，但并非全部

Logstash是一个用来搜集、分析、过滤日志的工具

# 安装ik分词器

将下载的压缩包。放入插件文件夹下，建立文件夹ik

```shell
[root@localhost plugins]# pwd
/home/elasticsearch-7.3.2/plugins
[root@localhost plugins]# mkdir ik

## 查看插件
[root@localhost bin]# ./elasticsearch-plugin list
ik
```

安装插件后，启动es时，也能看到集群对应有

[node-1] loaded plugin [analysis-ik]

字样

ik分词器提供了两种分词算法：

ik_smart ：最少切分

ik_max_word：最细粒度切分

- 测试

```json
GET /_analyze
{
  "analyzer": "ik_smart",
  "text": "我是帅哥"
}
```

```json
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "我是一个帅哥"
}
```

## IK配置

进入配置

```shell
[root@localhost ~]# cd /home/elasticsearch-7.3.2/plugins/ik/config/
[root@localhost config]# vim IKAnalyzer.cfg.xml
```

ext_dict:同目录下一个xx.dic文件，

```xml
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <!-- <entry key="remote_ext_dict">words_location</entry> -->
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```

# Spring Boot Api

## 基础配置

```java
@Bean
public RestHighLevelClient restHighLevelClient(){
    return new RestHighLevelClient(
        RestClient.builder(
            new HttpHost("192.168.1.131",9200, "http")
        )
    );
}
```

## 创建索引

```java
@Test
void creatIndex() throws IOException {
    //创建索引
    CreateIndexRequest request = new CreateIndexRequest(INDEX_NAME);
//指向客户端请求，获取响应
CreateIndexResponse response = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
}
```

## 删除索引

```java
@Test
void deleteIndex() throws IOException {
    DeleteIndexRequest request = new DeleteIndexRequest(INDEX_NAME);
    AcknowledgedResponse response = restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
    System.out.println(response.isAcknowledged());
}
```

## 判断索引是否存在

```java
@Test
void getIndex() throws IOException {
   //判断索引是否存在
   GetIndexRequest request = new GetIndexRequest(INDEX_NAME);
   Boolean response = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
   System.out.println(response);
}
```

## 新增文档

```java
@Test
void addDocument() throws IOException {
   User user = new User("小肖", "123456");
   //put /index/_doc/1
   IndexRequest request = new IndexRequest(INDEX_NAME);
   request.id("1")//id
         .timeout(TimeValue.timeValueSeconds(1));//超时时间
   request.source(JSONUtil.toJsonStr(user), XContentType.JSON);
   IndexResponse response = restHighLevelClient.index(request, RequestOptions.DEFAULT);
   System.out.println(response.status());
}
```

## 批量新增

```java
@Test
void bulkDocument() throws IOException {
   BulkRequest bulkRequest = new BulkRequest();
   bulkRequest.timeout(TimeValue.timeValueSeconds(1));
   for(int i=0; i<10; i++){
      bulkRequest.add(new IndexRequest(INDEX_NAME)
      .source(JSONUtil.toJsonStr(new User("name+"+i, String.valueOf(i))),XContentType.JSON));
   }
   restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
}
```