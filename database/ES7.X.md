# 基本操作

## 索引

```shell
# 查看所有缩影相关信息
GET /_cat/indices?v

# 查看索引数量
GET table_4182d99ae22f4c55a487d886d71f42df/_count
## 查看索引前十条了解索引结构
POST table_4182d99ae22f4c55a487d886d71f42df/_search
{
}

```

## fields

一个字段多个分词

```json
POST /test_index/_mapping
{
  "properties": {
      "name": {
          "type": "text",
          "fields": {
              "name": {
                  "type": "text",
                  "analyzer": "ik_smart"
              },
              "name1": {
                  "type": "keyword"
              }
          }
      }
   }
}
```

- 查询实例

```json
POST /test_index/_doc
{
  "name": "深圳市腾讯计算机系统有限公司"
}
```

## 高亮

```shell
POST /test_index/_search
{
  "query": {
    "match": {
      "name.name": "深圳市腾讯计算机系统有限公司"
    }
  },
  "highlight": {
    "fields": {
      "name.name": {"pre_tags": "<calss>", "post_tags": "</calss>"}
    }
  }
}
```

## 插入不允许有多余字段

当插入超出mapping多余字段之后，则报错

```json
PUT /sys_org_company
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
```



# 批量查询

- ## 批量查询

```json
## 批量查询
GET /_mget
{
  "docs": [
      {
        "_index": "table_4182d99ae22f4c55a487d886d71f42df",
        "_id":"GHcA-XUBb4eiMhXMyWxb"
      },
      {
        "_index": "table_048cbad320f649858efbc56fbfd7320b",
        "_id":"H44GF3YBSjjQlEnFqXEA"
      }
    ]
}
```

```json
## 批量查询指定索引id集合
GET table_4182d99ae22f4c55a487d886d71f42df/_mget
{
  "ids": ["GHcA-XUBb4eiMhXMyWxb", "H44GF3YBSjjQlEnFqXEA"]
}
```

Search

| 语法                      | 描述                      |
| ------------------------- | ------------------------- |
| GET /_search              | 查询所有index             |
| GET /index1,index2/search | 查询index1, index2的index |
| GET /index*/search        | 查询index开头的           |

- 通过url query来实现搜索

q: 指定查询语句，语法为 Query String Syntax
df: q中不指定字段时默认查询的字段，如果不指定，es会查询所有字段
sort：排序
timeout：指定超时时间，默认不超时
from,size：用于分页

```shell
##指定字段查询和 ?q=2012&df=title一样
GET /movies/_search?q= title:2012&sort=year:asc&from=0&size=10
```

- 通过http请求查询

![1607563675460](../image/es\1607563675460.png)

## Body 查询

- 忽略错误索引，查询两个索引，一个不存在则忽略
  - ignore_unavailable=true表示不存在的索引会忽略

```shell
POST /test_index,404_index/_search?ignore_unavailable=true
{
    "profile": true,
    "query": {
        "match_all": {}
    }
}
```

### 分页

- 从0开始，一页2条

```shell
POST /bs_trail_log/_search
{
  "from": 0,
  "size": 2,
  "query": {
    "match_all": {
      
    }
  }
}
```

### 排序

在使用 ElasticSearch 的时候，如果索引中的字段是 text 类型，针对该字段聚合、排序和查询的时候常会出现 `Fielddata is disabled on text fields by default. Set fielddata=true` 的错误

所以一般排序字段，不使用text类型

```shell
POST index/_search
{
  "sort": [
    {
      "field": {
        "order": "desc"
      }
    }
  ]
}
```

### 脚本字段

 painless脚本字段，这种机制可以通过脚本的方式做一些简单的操作，比如：字符串拼接

```json
POST /sys_org_company/_search
{
  "script_fields": {
    "new_name": {
      "script": {
        "lang": "painless",
        "source": "doc['faRen'].value + '1243'"
      }
    }
  }
}
```



## 查询表达式

 query match中，查询的内容默认是OR的方式。如下所示:

```shell
POST /sys_org_company/_search
{
  "query":  {"match": {
    "companyName": {
      "query": "多有米网络技术有限公司"
    }
  }}
}
```
如果想要分词都匹配，则用operatord的and方式

```json
POST /sys_org_company/_search
{
  "query":  {"match": {
    "companyName": {
      "query": "多有米网络技术有限公司",
      "operator": "and"
    }
  }}
}
```

## 短语查询

使用短语查询的方式。即：`phrase query`。默认短语中间不能有间隔，但是可使用slop=1来表示中间可以间隔一个term（单词）

# Mapping

# 聚合分析

**如果是text字段，则需要"fielddata": true才能聚合**

  Aggregation共分为三种

- Metric Aggregations
  - —些数学运算，可以对文档字段进行统计分析
- Bucket Aggregations
  - 一些列满足特定条件的文档的集合
- Pipeline Aggregations
  - 对其他的聚合结果进行二次聚合
- Matrix Aggregations
  - 支持对多个字段的操作并提供—个结果矩阵

Metric Aggregations主要是做一系列的统计，Bucket Aggregations相当于分组。

```sql
select count(name)  -- metric
from user
group by name    -- bucket
```

## bucket api 

size:不查询出数据

terms：按照字段分桶查询

```shell
POST /sys_org_company/_search
{
  "size": 0, 
  "aggs": {
    "new_count": {
      "terms": {
        "field": "faRen",
        "size": 10
      }
    }
  }
}
```

## Metric api

avg: 平均值

max:最大值

min:最小值

```json
"aggs": {
    "avg_age": {
        "avg": {
            "field": "age"
        }
    }
}
```

stats：求和求最小等一起求出来

```shell
"stats_name" : {
    "count" : 4,
    "min" : 10.0,
    "max" : 40.0,
    "avg" : 20.0,
    "sum" : 80.0
}
```

extended_stats： 其他属性，方差等

cardinality： 获取唯一值，相当于去重

```json
"aggs": {
    "stats_name": {
      "cardinality": {
        "field": "age"
      }
    }
  }
```

value_count:查看当前范围有多有不同的值

```json
"aggs": {
    "stats_name": {
      "value_count": {
        "field": "name"
      }
    }
  }
```



# 嵌套数据

nested: 

因此除了基本数据类型之外，ES也支持使用複杂的数据类型，像是数组、内部对象，而要使用内部对象的话，需要使用`nested`来定义索引，使文档内可以包含一个内部对象

- 为什麽不用object而要使用nested来定义索引的原因是，obejct类型会使得内部对象的关联性丢失
  - 这是因为Lucene底层其实没有内部对象的概念，所以ES会利用简单的列表储存字段名和值，将object类型的对象层次摊平，再传给Lucene
  - 假设user类型是object，当插入一笔新的数据时，ES会将他转换为下面的内部文档，其中可以看见alice和white的关联性丢失

```json
PUT 127.0.0.1/mytest/doc/1
{
    "group": "fans",
    "user": [
        { "first": "John", "last": "Smith" },
        { "first": "Alice", "last": "White" }
    ]
}
```

- 转换后

```json
{
    "group": "fans",
    "user.first": [ "alice", "john" ],
    "user.last": [ "smith", "white" ]
}
```

## 嵌套查询

- 由于嵌套对象被索引在独立的隐藏文档中，因此我们无法直接使用一般的query去查询他，我们必须改使用 "nested查询" 去查询他们
- nested查询的内部必须要包含一个`path`参数，负责指定要用的是哪个nested类型的字段，且要包含一个`query`，负责进行此嵌套对象内的查询

```shell
GET 127.0.0.1/mytest/doc/_search
{
    "query": {
        "nested": {
            "path": "user",
            "query": {
                "bool": {
                    "must": [
                        { "term": { "user.first": "Amy" } },
                        { "term": { "user.last": "White" } }
                    ]
                }
            }
        }
    }
}
```

## 嵌套统计

```shell
"aggs": {
   "NAME": {
     "nested": {
       "path": "user"
     },
     "aggs": {
       "temp": {
         "terms": {
           "field": "user.first"
         }
       }
     }
   }
 }
```

