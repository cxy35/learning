通过本文学习 Elasticsearch 中的 IK 中文分词器。
<!-- more -->

## 1 为什么需要 IK 中文分词器

Elasticsearch 内置的分词器对中文不友好，只会一个字一个字的分，无法形成词语，比如：

```
POST /_analyze
{
  "text": "我爱北京天安门",
  "analyzer": "standard"
}
```

如果我们使用的是 `standard` 分词器，那么结果就是：

```
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    ...
    {
      "token" : "门",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "<IDEOGRAPHIC>",
      "position" : 6
    }
  ]
}
```

显然这对中文来说并不友好，它显示的每一个汉字，因此需要中文分词器。

## 2 安装

下载并安装与 Elasticsearch 版本号一致的 IK 中文分词器，下载地址如下：[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)，安装命令如下：

```bash
cd /usr/local/elastic/elasticsearch-7.13.1
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.13.1/elasticsearch-analysis-ik-7.13.1.zip

#安装好后，我们可以通过如下的命令来检查是否已经安装好
./bin/elasticsearch-plugin list
  analysis-ik
```

安装成功后，需要重新启动一下我们的 Elasticsearch，以便这个 plugin 能装被加载。

## 3 使用

根据 IK 分词器的文档，它含有如下的部分：

- Analyzer：
  - ik_smart
  - ik_max_word

- Tokenizer:
  - ik_smart
  - ik_max_word

我们先使用先前的句子 “我爱北京天安门” 来检查一下：

```
POST /_analyze
{
  "text": "我爱北京天安门",
  "analyzer": "ik_smart"
}
```

上面的 ik_smart 分词器显示的结果是：

```
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "北京",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "天安门",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 3
    }
  ]
}
```

---

我们接着使用 ik_max_word 分词器来试一下同样的句子：

```
POST /_analyze
{
  "text": "我爱北京天安门",
  "analyzer": "ik_max_word"
}
```

上面的 ik_max_word 分词器显示的结果为：

```
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "北京",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "天安门",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "天安",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "门",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "CN_CHAR",
      "position" : 5
    }
  ]
}
```

从两个输出中我们可以看出来：这两者的区别在于它们提取词项的粒度上，前者 `ik_smart` 提取粒度较粗，而后者 `ik_max_word` 则较细，它给出更多的 token。

---

一般可以按如下方式来创建索引：

```
PUT /user
{
  "mappings": {
    "properties": {
      "address": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

---

- [Elasticsearch: analyzer](https://elasticstack.blog.csdn.net/article/details/100392478)
- [Elasticsearch：IK 中文分词器](https://elasticstack.blog.csdn.net/article/details/100516428)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)