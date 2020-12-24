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

## 批量查询

- 批量查询

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

### Search

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

### Body 查询

- 

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

