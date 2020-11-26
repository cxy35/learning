---
title: Elasticsearch 常用命令
date: 2020-10-21 11:02:47
categories: Elasticsearch
tags: [Elastic, Elasticsearch, 命令]
toc: true
---
通过本文学习 Elasticsearch 常用命令，以下所有命令都在 Kibana 的 Dev Tools 中执行。
<!-- more -->

## 1 全局

- 查看集群健康状态：`GET /_cat/health?v`
- 查看节点状态：`GET /_cat/nodes?v`
- 查看所有索引信息：`GET /_cat/indices?v`

## 2 索引

- 新增索引：`PUT /user`
- 新增索引-自定义`settings`：
```
PUT /user
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```
- 新增索引-自定义`mappings`：
```
PUT /user
{
  "mappings": {
    "properties": {
      "username": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "address" : {
        "type" : "text",
        "analyzer" : "ik_max_word",
        "search_analyzer" : "ik_smart"
      },
      "location": {
        "type": "geo_point"
      },
      ......
    }
  }
}
```
- 查看索引：`GET /user`
- 查看索引中的`settings`：`GET /user/_settings`
- 查看索引中的`mappings`：`GET /user/_mappings`
- 删除索引：`DELETE /user`

## 3 文档

- 新增文档-指定id（id不存在则新增，存在则更新）：
```
POST /user/_doc/1
{
  "id": 1,
  "username": "zhangsan",
  "password": "123456",
  "enabled": true,
  "locked": false,
  "address": "浙江杭州",
  "nickName": "张三",
  "createTime": 1602836232511,
  "updateTime": 1602836232511
}
```
- 新增文档-指定id（id不存在则新增，存在则报错）：
```
POST /user/_create/1
{
  "id": 1,
  "username": "zhangsan",
  "password": "123456",
  "enabled": true,
  "locked": false,
  "address": "浙江杭州",
  "nickName": "张三",
  "createTime": 1602836232511,
  "updateTime": 1602836232511
}
```
- 新增文档-自动生成id：
```
POST /user/_doc
{
  "id": 2,
  "username": "lisi",
  "password": "123456",
  "enabled": true,
  "locked": false,
  "address": "浙江宁波",
  "nickName": "李四",
  "createTime": 1602836232511,
  "updateTime": 1602836232511
}
```
- 新增文档-批量`_bulk`：
```
POST /user/_bulk
{"index":{"_id":"1"}}
{"id":1,"username":"zhangsan"}
{"index":{"_id":"2"}}
{"id":2,"username":"zhangsan2"}
```
- 查看文档：`GET /user/_doc/1`
- 查看文档-批量`_mget`：
```
GET /user/_mget
{
  "ids": ["1", "2"]
}
```
- 修改文档-指定id（id不存在则新增，存在则替换）-整体修改（因此需要传所有字段）：
```
POST /user/_doc/1
{
  "id": 1,
  "username": "zhangsan",
  "password": "654321",
  "enabled": true,
  "locked": false,
  "address": "浙江杭州",
  "nickName": "张三",
  "createTime": 1602836232511,
  "updateTime": 1602836232511
}
```
- 修改文档-指定id（id不存在则报错，存在则合并）-局部修改（因此只需要传修改的字段）：
```
POST /user/_update/1
{
  "doc": {
    "password": "121212"
  }
}
```
```
- 修改文档-指定id（id不存在则创建，存在则合并）-局部修改（因此只需要传修改的字段）：
```
POST /user/_update/1
{
  "doc": {
    "password": "121212"
  },
  "doc_as_upsert": true
}
```
- 修改文档-指定id-局部修改-脚本方式：
```
POST /user/_update/1
{
  "script" : {
      "source": "ctx._source.password = params.password;ctx._source.address = params.address",
      "lang": "painless",
      "params": {
        "password": "232323",
        "address": "浙江宁波"
      }
  }
}
```
- 修改文档-`update_by_query`-局部修改-脚本方式：
```
POST /user/_update_by_query
{
  "query": {
    "match": {
      "username": "zhangsan"
    }
  },
  "script": {
    "source": "ctx._source.password = params.password;ctx._source.address = params.address,
    "lang": "painless",
    "params": {
      "password": "654321",
      "address": "浙江杭州"
    }
  }
}
```
- 检查文档是否存在：`HEAD /user/_doc/1`
- 删除文档-指定id：`DELETE /user/_doc/1`
- 删除文档-批量`_bulk`：
```
POST /_bulk
{ "delete" : { "_index" : "user2", "_id": 1 }}
{ "delete" : { "_index" : "user2", "_id": 6 }}
```
- 删除文档-`delete_by_query`：
```
POST /user/_delete_by_query
{
  "query": {
    "match": {
      "username": "zhangsan"
    }
  }
}
```

## 4 搜索

### 4.1 普通

- 搜索-文档总数`_count`：`GET /user/_count`
- 搜索-文档数量`_count`：
```
GET /user/_count
{
  "query": {
    "match": {
      "address": "浙江杭州"
    }
  }
}
```

---

- 搜索`_search`：
```
GET /_search
GET /user,user2/_search
GET /user/_search
GET /user/_search?size=20
GET /user/_search?size=20&from=3
GET /user/_search?q=address:"浙江杭州"
GET /user/_search?filter_path=hits.total
GET /user/_search?_source=username,address
```
- 搜索-所有文档`match_all`：
```
GET /user/_search
{
  "query": {
    "match_all": {}
  }
}
```
- 搜索-分页`from/size`：
```
GET /user/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2
}
```
- 搜索-排序`sort`：
```
GET /user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": {
    "updateTime": {
      "order": "desc"  
    }
  }
}
```
- 搜索-指定返回字段`_source`：
```
GET /user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["username", "address"]
}
```

### 4.2 匹配

- 搜索-匹配`match`（数值类型是精确匹配，文本类型是模糊匹配）：
```
GET /user/_search
{
  "query": {
    "match": {
      "enabled": 1
    }
  }
}

# GET /user/_search?q=enabled:1
```
- 搜索-多字段匹配`multi_match`（或）：
```
POST /user/_search
{
  "query": {
    "multi_match": {
      "query": "zhang",
      "fields": [
        "username",
        "nickName"
      ]
    }
  }
}
```
- 搜索-短语匹配`match_phrase`（同时包含多个短语）：
```
GET /user/_search
{
  "query": {
    "match_phrase": {
      "address": "zhejiang hangzhou"
    }
  }
}
```

### 4.3 组合

- 搜索-组合`bool/must`（同时满足多个条件）：
```
GET /user/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "zhejiang" } },
        { "match": { "address": "hangzhou" } }
      ]
    }
  }
}
```
- 搜索-组合`bool/should`（满足其中任意一个条件）：
```
GET /user/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "zhejiang" } },
        { "match": { "address": "hangzhou" } }
      ]
    }
  }
}
```
- 搜索-组合`bool/must_not`（同时不满足多个条件）：
```
GET /user/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "zhejiang" } },
        { "match": { "address": "hangzhou" } }
      ]
    }
  }
}
```
- 搜索-组合`bool/filter`（过滤，keyword 字段用于一般用于精确搜索、聚合、排序）：
```
GET /user/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "address.keyword": "浙江杭州"
        }
      }
    }
  }
}
```
- 搜索-多种组合：
```
GET /user/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "enabled": 1 } }
      ],
      "must_not": [
        { "match": { "address": "hangzhou" } }
      ]
    }
  }
}
```

### 4.4 过滤

- 搜索-过滤`filter`：
```
GET /user/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "id": {
            "gte": 3,
            "lte": 6
          }
        }
      }
    }
  }
}
```

### 4.5 聚合

- 搜索-聚合`aggs`（类似于 MySQL 中的 `group by`）：

```
GET /user/_search
{
  "size": 0,
  "aggs": {
    "myagg_enabled": {
      "terms": {
        "field": "enabled.keyword"
      }
    }
  }
}
```
- 搜索-嵌套聚合`aggs`：
```
GET /user/_search
{
  "size": 0,
  "aggs": {
    "myagg_enabled": {
      "terms": {
        "field": "enabled.keyword"
      },
      "aggs": {
        "myagg_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}
```
- 搜索-嵌套聚合`aggs`并对结果进行排序`order`：
```
GET /user/_search
{
  "size": 0,
  "aggs": {
    "myagg_enabled": {
      "terms": {
        "field": "enabled.keyword",
        "order": {
          "myagg_age": "desc"
        }
      },
      "aggs": {
        "myagg_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}
```
- 搜索-分段聚合`aggs/range`：
```
GET /user/_search
{
  "size": 0,
  "aggs": {
    "myagg_id": {
      "range": {
        "field": "id",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "myagg_enabled": {
          "terms": {
            "field": "enabled.keyword"
          },
          "aggs": {
            "myagg_age": {
              "avg": {
                "field": "age"
              }
            }
          }
        }
      }
    }
  }
}
```

### 4.6 官方搜索例子

```
# 准备数据
POST twitter/_bulk
{ "index" : { "_id": 1 } }
{"user":"双榆树-张三","message":"今儿天气不错啊，出去转转去","uid":2,"age":20,"city":"北京","province":"北京","country":"中国","address":"中国北京市海淀区","location":{"lat":"39.970718","lon":"116.325747"}}
{ "index" : { "_id": 2 } }
{"user":"东城区-老刘","message":"出发，下一站云南！","uid":3,"age":30,"city":"北京","province":"北京","country":"中国","address":"中国北京市东城区台基厂三条3号","location":{"lat":"39.904313","lon":"116.412754"}}
{ "index" : { "_id": 3 } }
{"user":"东城区-李四","message":"happy birthday!","uid":4,"age":30,"city":"北京","province":"北京","country":"中国","address":"中国北京市东城区","location":{"lat":"39.893801","lon":"116.408986"}}
{ "index" : { "_id": 4 } }
{"user":"朝阳区-老贾","message":"123,gogogo","uid":5,"age":35,"city":"北京","province":"北京","country":"中国","address":"中国北京市朝阳区建国门","location":{"lat":"39.718256","lon":"116.367910"}}
{ "index" : { "_id": 5 } }
{"user":"朝阳区-老王","message":"Happy BirthDay My Friend!","uid":6,"age":50,"city":"北京","province":"北京","country":"中国","address":"中国北京市朝阳区国贸","location":{"lat":"39.918256","lon":"116.467910"}}
{ "index" : { "_id": 6 } }
{"user":"虹桥-老吴","message":"好友来了都今天我生日，好友来了,什么 birthday happy 就成!","uid":7,"age":90,"city":"上海","province":"上海","country":"中国","address":"中国上海市闵行区","location":{"lat":"31.175927","lon":"121.383328"}}

# 查询
## _search
GET _search
GET /twitter,twitter2/_search
GET /twitter/_search
GET /twitter/_search?size=20
GET /twitter/_search?size=20&from=3
GET /twitter/_search?filter_path=hits.total
GET /twitter/_search?_source=user,city


## _count
GET /twitter/_count
GET /twitter/_count
{
  "query": {
    "match": {
      "city": "北京"
    }
  }
}

## 查看/设置 settings
GET /twitter/_settings
PUT twitter
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}

## 查看/设置 mapping
GET /twitter/_mapping
PUT twitter/_mapping
{
  "properties": {
    "address": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "age": {
      "type": "long"
    },
    "city": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "country": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "location": {
      "type": "geo_point"
    },
    "message": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "province": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "uid": {
      "type": "long"
    },
    "user": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    }
  }
}

## 普通查询
### match
GET /twitter/_search
{
  "query": {
    "match": {
      "city": "北京"
    }
  }
}

GET /twitter/_search?q=city:"北京"

### keyword字段用于精确搜索，aggregation和排序（sorting）
GET /twitter/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "city.keyword": "北京"
        }
      }
    }
  }
}

GET /twitter/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "city.keyword": {
            "value": "北京"
          }
        }
      }
    }
  }
}

### 匹配“朝”，“阳”，“区”，“老”及“贾”这5个字中的任何一个将被显示
GET /twitter/_search
{
  "query": {
    "match": {
      "user": {
        "query": "朝阳区-老贾",
        "operator": "or"
      }
    }
  }
}

### 至少要匹配“朝”，“阳”，“区”，“老”及“贾这5个中的3个字才可以
GET /twitter/_search
{
  "query": {
    "match": {
      "user": {
        "query": "朝阳区-老贾",
        "operator": "or",
        "minimum_should_match": 3
      }
    }
  }
}

### 需要同时匹配索引的5个字才可以
GET /twitter/_search
{
  "query": {
    "match": {
      "user": {
        "query": "朝阳区-老贾",
        "operator": "and"
      }
    }
  }
}

### multi：同时对三个fields: user，adress及message进行搜索
GET /twitter/_search
{
  "query": {
    "multi_match": {
      "query": "朝阳",
      "fields": [
        "user",
        "address^3",
        "message"
      ],
      "type": "best_fields"
    }
  }
}

### prefix：查询user字段里以“朝”为开头的所有文档
GET /twitter/_search
{
  "query": {
    "prefix": {
      "user": {
        "value": "朝"
      }
    }
  }
}

### term：使用user.keyword来对“朝阳区-老贾”进行精确匹配查询相应的文档
GET /twitter/_search
{
  "query": {
    "term": {
      "user.keyword": {
        "value": "朝阳区-老贾"
      }
    }
  }
}

### terms：查询user.keyword里含有“双榆树-张三”或“东城区-老刘”的所有文档
GET /twitter/_search
{
  "query": {
    "terms": {
      "user.keyword": [
        "双榆树-张三",
        "东城区-老刘"
      ]
    }
  }
}

## 复合查询
### must：查询的是必须是 北京城市的，并且年刚好是30岁的
GET /twitter/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "city": "北京"
          }
        },
        {
          "match": {
            "age": "30"
          }
        }
      ]
    }
  }
}

### must_not：寻找不在北京的所有的文档
GET /twitter/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "city": "北京"
          }
        }
      ]
    }
  }
}

### should：表述“或”的意思，也就是有就更好，没有就算了。age必须是30岁，但是如果文档里含有“Hanppy birthday”，相关性会更高
GET /twitter/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "age": "30"
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "message": "Happy birthday"
          }
        }
      ]
    }
  }
}

### 位置查询 geo_distance ：查找在地址栏里有“北京”，并且在以位置(116.454182, 39.920086)为中心的5公里以内的所有文档，并按照远近大小进行排序
GET /twitter/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "北京"
          }
        }
      ]
    }
  },
  "post_filter": {
    "geo_distance": {
      "distance": "5km",
      "location": {
        "lat": 39.920086,
        "lon": 116.454182
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location": "39.920086,116.454182",
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}

### 范围查询 range ：查询年龄介于30到40岁的文档
GET /twitter/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 40
      }
    }
  }
}

### 范围查询 range ：查询年龄介于30到40岁的文档，并对它们进行排序
GET /twitter/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 40
      }
    }
  },"sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}

### Exists 查询：查询 district 字段是否存在
GET /twitter/_search
{
  "query": {
    "exists": {
      "field": "district"
    }
  }
}

### 匹配查询
PUT twitter/_doc/8
{
  "user": "朝阳区-老王",
  "message": "Happy",
  "uid": 6,
  "age": 50,
  "city": "北京",
  "province": "北京",
  "country": "中国",
  "address": "中国北京市朝阳区国贸",
  "location": {
    "lat": "39.918256",
    "lon": "116.467910"
  }
}

### 匹配查询：match 查询时不分大小写，不分先后顺序
### 默认“或”，匹配到一个单词即可
GET /twitter/_search
{
  "query": {
    "match": {
      "message": "happy birthday"
    }
  }
}

### “与”，需要同时匹配两个单词
GET /twitter/_search
{
  "query": {
    "match": {
      "message": {
        "query": "happy birthday",
        "operator": "and"
      }
    }
  }
}

### minimum_should_match 至少应该要匹配两个单词
GET /twitter/_search
{
  "query": {
    "match": {
      "message": {
        "query": "happy birthday",
        "minimum_should_match": 2
      }
    }
  }
}

### 匹配查询：match_phrase 查询时不分大小写，但区分先后顺序
GET /twitter/_search
{
  "query": {
    "match_phrase": {
      "message": "happy birthday"
    }
  },
  "highlight": {
    "fields": {
      "message": {}
    }
  }
}

### SQL 查询
GET /_sql?
{
  "query": """
    SELECT * FROM twitter 
    WHERE age = 30
  """
}

### Multi Search：减少请求次数
### 同时获取两个文档
GET /twitter/_doc/_mget
{
  "ids": ["1", "2"]
}

### 同时到多个索引中查询
GET _msearch
{"index":"twitter"}
{"query":{"match_all":{}},"from":0,"size":1}
{"index":"twitter"}
{"query":{"bool":{"filter":{"term":{"city.keyword":"北京"}}}}, "size":1}
{"index":"twitter2"}
{"query":{"match_all":{}}}

GET /twitter,twitter2/_search
GET /twitter*/_search

### Profile：调试工具，它添加了有关执行的详细信息搜索请求中的每个组件，它为用户提供有关搜索的每个步骤的洞察力
GET /twitter/_search
{
  "profile": "true", 
  "query": {
    "match": {
      "city": "北京"
    }
  }
}


## 聚合查询 aggregation 及 analyzer
### 准备数据
DELETE twitter
PUT twitter
{
  "mappings": {
    "properties": {
      "DOB": {
        "type": "date"
      },
      "address": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "age": {
        "type": "long"
      },
      "city": {
        "type": "keyword"
      },
      "country": {
        "type": "keyword"
      },
      "location": {
        "type": "geo_point"
      },
      "message": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "province": {
        "type": "keyword"
      },
      "uid": {
        "type": "long"
      },
      "user": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
POST _bulk
{"index":{"_index":"twitter","_id":1}}
{"user":"张三","message":"今儿天气不错啊，出去转转去","uid":2,"age":20,"city":"北京","province":"北京","country":"中国","address":"中国北京市海淀区","location":{"lat":"39.970718","lon":"116.325747"}, "DOB": "1999-04-01"}
{"index":{"_index":"twitter","_id":2}}
{"user":"老刘","message":"出发，下一站云南！","uid":3,"age":22,"city":"北京","province":"北京","country":"中国","address":"中国北京市东城区台基厂三条3号","location":{"lat":"39.904313","lon":"116.412754"}, "DOB": "1997-04-01"}
{"index":{"_index":"twitter","_id":3}}
{"user":"李四","message":"happy birthday!","uid":4,"age":25,"city":"北京","province":"北京","country":"中国","address":"中国北京市东城区","location":{"lat":"39.893801","lon":"116.408986"}, "DOB": "1994-04-01"}
{"index":{"_index":"twitter","_id":4}}
{"user":"老贾","message":"123,gogogo","uid":5,"age":30,"city":"北京","province":"北京","country":"中国","address":"中国北京市朝阳区建国门","location":{"lat":"39.718256","lon":"116.367910"}, "DOB": "1989-04-01"}
{"index":{"_index":"twitter","_id":5}}
{"user":"老王","message":"Happy BirthDay My Friend!","uid":6,"age":26,"city":"北京","province":"北京","country":"中国","address":"中国北京市朝阳区国贸","location":{"lat":"39.918256","lon":"116.467910"}, "DOB": "1993-04-01"}
{"index":{"_index":"twitter","_id":6}}
{"user":"老吴","message":"好友来了都今天我生日，好友来了,什么 birthday happy 就成!","uid":7,"age":28,"city":"上海","province":"上海","country":"中国","address":"中国上海市闵行区","location":{"lat":"31.175927","lon":"121.383328"}, "DOB": "1991-04-01"}

### range 聚合：把用户进行年龄分段，查出来在不同的年龄段的用户
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      }
    }
  }
}

### date_range 聚合：统计在某个时间段里的文档数
POST twitter/_search
{
  "size": 0,
  "aggs": {
    "birth_range": {
      "date_range": {
        "field": "DOB",
        "format": "yyyy-MM-dd",
        "ranges": [
          {
            "from": "1989-01-01",
            "to": "1990-01-01"
          },
          {
            "from": "1991-01-01",
            "to": "1992-01-01"
          }
        ]
      }
    }
  }
}

### terms 聚合：寻找在所有的文档出现”happy birthday”里按照城市进行分类的一个聚合
GET /twitter/_search
{
  "query": {
    "match": {
      "message": "happy birthday"
    }
  },
  "size": 0,
  "aggs": {
    "city": {
      "terms": {
        "field": "city",
        "size": 10
      }
    }
  }
}

### terms 聚合：我们也可以使用 script 来生成一个在索引里没有的术语来进行统计。比如，我们可以通过如下的script来生成一个对文档人出生年份的统计：
POST twitter/_search
{
  "size": 0,
  "aggs": {
    "birth_year": {
      "terms": {
        "script": {
          "source": "2019 - doc['age'].value"
        }, 
        "size": 10
      }
    }
  }
}

### histogram 聚合：根据值动态构建固定大小（也称为间隔）的存储桶
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "age_distribution": {
      "histogram": {
        "field": "age",
        "interval": 2
      }
    }
  }
}

### date_histogram 聚合：这种聚合类似于正常的直方图，但只能与日期或日期范围值一起使用。这里我们按照每隔一年这样的时间间隔来进行
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "age_distribution": {
      "date_histogram": {
        "field": "DOB",
        "interval": "year"
      }
    }
  }
}

### cardinality 聚合：统计有多少个城市
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "number_of_cities": {
      "cardinality": {
        "field": "city.keyword"
      }
    }
  }
}

### Metric 聚合：我们可以使用Metrics来统计我们的数值数据
### 对年龄字段全方位统计，包括数据总条数、平均值、最大/小值、求和
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "age_stats": {
      "stats": {
        "field": "age"
      }
    }
  }
}

### 只得到平均值
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "average_age": {
      "avg": {
        "field": "age"
      }
    }
  }
}

POST twitter/_search
{
  "size": 0,
  "query": {
    "match": {
      "city": "北京"
    }
  },
  "aggs": {
    "average_age_beijing": {
      "avg": {
        "field": "age"
      }
    }
  }
}

POST twitter/_search
{
  "size": 0,
  "query": {
    "match": {
      "city": "北京"
    }
  },
  "aggs": {
    "average_age_beijing": {
      "avg": {
        "field": "age"
      }
    },
    "average_age_all": {
      "global": {},
      "aggs": {
        "age_global_avg": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}

### 使用 script，计算平均值再乘以2倍的结果
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "average_age_2": {
      "avg": {
        "field": "age",
        "script": {
          "source": "_value * params.correction",
          "params": {
            "correction": 2
          }
        }
      }
    }
  }
}

### 或者直接使用script的方法来进行聚合
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "average_2_times_age": {
      "avg": {
        "script": {
          "source": "doc['age'].value * params.times",
          "params": {
            "times": 2.0
          }
        }
      }
    }
  }
}

### percentile：得到25%，50%、75%及100%的人在什么年龄范围
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "age_quartiles": {
      "percentiles": {
        "field": "age",
        "percents": [
          25,
          50,
          75,
          100
        ]
      }
    }
  }
}

### 更为复杂的聚合：结合上面的 bucket 聚合及 metric 聚合
### 我们首先通过terms来生成每个城市的桶聚合，然后在每个桶里计算所有文档的平均年龄，并根据平均年龄来进行降序排序
GET /twitter/_search
{
  "size": 0,
  "aggs": {
    "cities": {
      "terms": {
        "field": "city",
        "order": {
          "average_age": "desc"
        }, 
        "size": 5
      },
      "aggs": {
        "average_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}

## Analyzer 分为三个部分：Char Filters, Tokenizer及 Token Filter
### happy、birthday
GET /twitter/_analyze
{
  "text": [
    "Happy Birthday"
  ],
  "analyzer": "standard"
}

### happi、birthdai（词根）
GET /twitter/_analyze
{
  "text": [
    "Happy Birthday"
  ],
  "analyzer": "english"
}

### Happy、Birthday
GET /twitter/_analyze
{
  "text": [
    "Happy Birthday"
  ],
  "analyzer": "whitespace"
}

### happy、birthday
GET /twitter/_analyze
{
  "text": [
    "Happy.Birthday"
  ],
  "analyzer": "simple"
}

### 生、日、快、乐
GET /twitter/_analyze
{
  "text": [
    "生日快乐"
  ],
  "analyzer": "standard"
}

### 也可以只使用 Analyzer 中的 Tokenizer 部分
### Happy、Birthday
GET /twitter/_analyze
{
  "text": [
    "Happy Birthday"
  ],
  "tokenizer": "standard"
}

### Happy Birthday
GET /twitter/_analyze
{
  "text": [
    "Happy Birthday"
  ],
  "tokenizer": "keyword"
}

### happy birthday
GET /twitter/_analyze
{
  "text": [
    "Happy Birthday"
  ],
  "tokenizer": "keyword",
  "filter": ["lowercase"]
}

```

---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)