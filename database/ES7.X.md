# 基本操作



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

# 批量查询

- ## 批量查询







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



## 数据刷新

默认情况下`ElasticSearch`索引的`refresh_interval`为`1`秒，这意味着数据写`1`秒才就可以被搜索到。

因为上述表现，所以称ElasticSearch是`近实时`搜索引擎。

如果需要调整数据刷新方案，则有三种途径：

- 设置数据刷新间隔：refresh_interval。
- 调用数据刷新接口：`_refresh`。
- 设置数据刷新策略：RefreshPolicy。
- 刷新间隔为`-1`代表关闭数据刷新。
- 刷新间隔设置时应改携带单位，如：1h、2m、3s。

### 调整时间间隔

```json
# 调整所有index的刷新间隔位5分钟
PUT
{
  "settings": {
    "refresh_interval": "5m" 
  }
}

# 调整指定index的刷新间隔为180秒
PUT /order_log
{
  "settings": {
    "refresh_interval": "180s" 
  }
}
```

### 关闭时间间隔

例如：现在需要将mysql中的数据做一次ElasticSearch全量更新，此时可以先关闭自动刷新，全量更新完成之后再打开。

```json
# 关闭全部index的数据刷新
PUT
{
  "settings": {
    "refresh_interval": -1 
  }
}

# 调整指定index的刷新间隔为1秒
PUT /user
{
  "settings": {
    "refresh_interval": "1s" 
  }
}

```

### 手动刷新

可以手动调用ElasticSearch提供的API进行数据刷新，如下

```json
# 刷新全部index的数据
POST /_refresh 

# 刷新指定index的数据
POST /user/_refresh
```



# Mapping

- 新增一个mapping

```jsonconsole
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}
```

- 新增一个已存在的mapping

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}
```

## mapping类型

https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html

- 范围类型
  - integer_range、float_range、long_range、double_range 以及 date_range

```json

#创建索引
PUT /range_test
{
  "mappings": {
    "_doc": {
      "properties": {
        "count": {
          "type": "integer_range"
        },
        "create_date": {
          "type": "date_range", 
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}
 
#添加数据
POST /range_test/_doc/1
{
  "count" : { 
    "gte" : 1,
    "lte" : 100
  },
  "create_date" : { 
    "gte" : "2019-02-1 12:00:00", 
    "lte" : "2019-03-30"
  }
}
#检索 其中5 在 1-100之间可以被检索出来
GET /range_test/_doc/_search
{
    "query":{
        "term":{
            "count":5
        }
    }
}
```

## 分词

- index设置为false，不可被搜索

```json

# index属性控制 字段是否可以被索引
PUT user_test
{
  "mappings": {
    "properties": {
      "mobile" :{
        "type": "text",
        "index": false
      }
    }
  }
}
```

- copy_to



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

因此除了基本数据类型之外，ES也支持使用复杂的数据类型，像是数组、内部对象，而要使用内部对象的话，需要使用`nested`来定义索引，使文档内可以包含一个内部对象

- 为什麽不用object而要使用nested来定义索引的原因是，obejct类型会使得内部对象的关联性丢失
  - 这是因为Lucene底层其实没有内部对象的概念，所以ES会利用简单的列表储存字段名和值，将object类型的对象层次摊平，再传给Lucene
  - 假设user类型是object，当插入一笔新的数据时，ES会将他转换为下面的内部文档，其中可以看见alice和white的关联性丢失

```json
PUT /mytest/doc/1
{
    "group": "fans",
    "user": [
        { "first": "John", "last": "Smith" },
        { "first": "Alice", "last": "White" }
    ]
}
```

- 转换后(内部隐式转换)
  - 所以，在检索John，Smith一起满足时，不会将{ "first": "John", "last": "Smith" }看成一条数据

```json
{
    "group": "fans",
    "user.first": [ "alice", "john" ],
    "user.last": [ "smith", "white" ]
}
```

- 定义nested索引
  - 定义之后，我们就可以用嵌套查询去查询一条完整的数据

```json
PUT /mytest
{
  "mappings": {
    "properties": {
      "user" : {
        "type": "nested"
      }
    }
  }
}
```

## 嵌套查询

- 由于嵌套对象被索引在独立的隐藏文档中，因此我们无法直接使用一般的query去查询他，我们必须改使用 "nested查询" 去查询他们
- nested查询的内部必须要包含一个`path`参数，负责指定要用的是哪个nested类型的字段，且要包含一个`query`，负责进行此嵌套对象内的查询

```shell
GET /mytest/_doc/_search
{
    "query": {
        "nested": {
            "path": "user",
            "query": {
                "bool": {
                    "must": [
                        { "match": { "user.first": "Amy" } },
                        { "match": { "user.last": "White" } }
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
         "match": {
           "field": "user.first"
         }
       }
     }
   }
 }
```

