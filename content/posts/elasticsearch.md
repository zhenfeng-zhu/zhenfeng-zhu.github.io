---
date: "2018-05-20 14:17:07"
title: elasticsearch
---

以前没有好好学的东西，现在在工作中慢慢的补回来了。

# 基础概念

- 索引

es是将数据存储在一个或者多个索引（index）中。

索引就像是数据库。

- 文档

文档是es的实体。由字段构成，每个字段包含字段名和一个或者多个字段值。

文档就像数据库中的一条条记录。

- 类型

每个文档都有一个类型与之相对应。

类型就像数据库中的表。

- 映射

所有文档在被写入到es中，都会被分析。由用户设置一些参数决定如何分割词条、哪些字应该被过滤掉等等。

- 节点

单个es服务实例就是一个节点。

- 集群

多个协同工作的es节点的集合就是集群。

- 分片

es将数据分散到多个物理的Lucene索引上，这些物理Lucene索引被称为分片。

- 副本

副本就是每个分片都做冗余处理，一个宕机之后，不影响服务。

# 快速入门

## 安装

es的安装很简单，我这里使用的是mac，下载下来zip包，解压即可使用。

```
[elasticsearch-6.2.4] pwd
/Users/zhuzhenfeng/Documents/software/elasticsearch-6.2.4
[elasticsearch-6.2.4] ./bin/elasticsearch
Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
[2018-05-20T17:18:37,619][INFO ][o.e.n.Node               ] [] initializing ...
[2018-05-20T17:18:37,766][INFO ][o.e.e.NodeEnvironment    ] [M41310-] using [1] data paths, mounts [[/ (/dev/disk1s1)]], net usable_space [136gb], net total_space [233.4gb], types [apfs]
[2018-05-20T17:18:37,767][INFO ][o.e.e.NodeEnvironment    ] [M41310-] heap size [990.7mb], compressed ordinary object pointers [true]
```

这样就将es启动了，然后在chrome中，输入http://localhost:9200，即可查看有没有启动成功。

```on
{
    "name": "M41310-",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "58U11tViTYuXpI2b5SiGrg",
    "version": {
        "number": "6.2.4",
        "build_hash": "ccec39f",
        "build_date": "2018-04-12T20:37:28.497551Z",
        "build_snapshot": false,
        "lucene_version": "7.2.1",
        "minimum_wire_compatibility_version": "5.6.0",
        "minimum_index_compatibility_version": "5.0.0"
    },
    "tagline": "You Know, for Search"
}
```

## 常用的命令

使用postman来模拟发送请求。

### 创建index

PUT

```
http://127.0.0.1:9200/myindex
```

response

```on
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "myindex"
}
```

创建了一个叫myindex的索引

### 删除index

DELETE

```
http://127.0.0.1:9200/myindex
```

response

```on
{
    "acknowledged": true
}
```

使用delete方法就可以删除索引，而且可以发现es的response特别人性化。

### 创建maping

POST

```
http://localhost:9200/myindex/fulltext/_mapping
```

body

```on
{
  "properties": {
    "content": {
      "type": "text",
      "analyzer": "ik_max_word",
      "search_analyzer": "ik_max_word"
    }
  }
}
```

response

```on
{
    "acknowledged": true
}
```

在这里在创建一个type是fulltext的同时，指定了这个fulltext类型的字段映射。在mapping中，一般是设置字段是什么类型的，比如bool，text等。analyzer是给文档建索引的分词方法，search_analyzer是搜索时对搜索的内容进行分词的方法。这里都是用了ik的分词器。

### 新增doc

POST

```
http://localhost:9200/myindex/fulltext/1
```

body

```on
{ 
    "content": "中国崛起哦" 
}
```

response

```on
{
    "_index": "myindex",
    "_type": "fulltext",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

如果没有指定id的话，每次新增的时候都会用es自动给的id。如果不注意的话，可能会出现重复新增，所以我们一般情况下会使用自己给的的id。

### 搜索

POST

```
http://127.0.0.1:9200/myindex/fulltext/_search
```

body

```on
{
  "query": {
    "match": {
      "content": {
        "query": "中国"
      }
    }
  }
}
```

response

```on
{
    "took": 98,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.2876821,
        "hits": [
            {
                "_index": "myindex",
                "_type": "fulltext",
                "_id": "1",
                "_score": 0.2876821,
                "_source": {
                    "content": "中国崛起哦"
                }
            }
        ]
    }
}。
```

前面都没啥，主要就是如何把数据灌到es中。es作为一个搜索引擎，肯定搜索才是最重要的。这里只是用的最简单的搜索。关于es的搜索，我在实际生产中主要使用的是，多字段的搜索，使用了bool操作符。

# 原理

## 相关性得分

Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。Elasticsearch中的 *相关性* 概念非常重要，也是完全区别于传统关系型数据库的一个概念，数据库中的一条记录要么匹配要么不匹配。

# 搜索

## 轻量级搜索

_search

_search?q=content:中国

这种搜索方式比较简单，很轻量。

## 查询表达式

即dsl形式的。使用的是POST方法，在body中，写搜索的dsl。

### 简单的dsl

```
{
  "query": {
    "match": {
      "content": {
        "query": "中国"
      }
    }
  }
}
```

### bool操作符的DSL

```on
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
```

我们添加了一个 *过滤器* 用于执行一个范围查询，并复用之前的 `match` 查询。

### 短语搜索

```on
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

如果不用短语搜索的话，会将只含有rock或者climbing的返回。为了能让二者是短语形式，es中新增了短语搜索dsl。

### 高亮

许多应用都倾向于在每个搜索结果中 *高亮* 部分文本片段，以便让用户知道为何该文档符合查询条件。在 Elasticsearch 中检索出高亮片段也很容易。

```on
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

加上highlight关键字即可。

### 聚合

 Elasticsearch 有一个功能叫聚合（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 `GROUP BY` 类似但更强大。

```on
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

