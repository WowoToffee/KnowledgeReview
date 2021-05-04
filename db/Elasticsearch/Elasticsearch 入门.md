# Elasticsearch 入门

## 基本概念

Elasticsarch 都是通过RESTful API 方式进行通信的

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：

```js
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

被 `< >` 标记的部件：

| `VERB`         | 适当的 HTTP *方法* 或 *谓词* : `GET`、 `POST`、 `PUT`、 `HEAD` 或者 `DELETE`。 |
| -------------- | ------------------------------------------------------------ |
| `PROTOCOL`     | `http` 或者 `https`（如果你在 Elasticsearch 前面有一个 `https` 代理） |
| `HOST`         | Elasticsearch 集群中任意节点的主机名，或者用 `localhost` 代表本地机器上的节点。 |
| `PORT`         | 运行 Elasticsearch HTTP 服务的端口号，默认是 `9200` 。       |
| `PATH`         | API 的终端路径（例如 `_count` 将返回集群中文档数量）。Path 可能包含多个组件，例如：`_cluster/stats` 和 `_nodes/stats/jvm` 。 |
| `QUERY_STRING` | 任意可选的查询字符串参数 (例如 `?pretty` 将格式化地输出 JSON 返回值，使其更容易阅读) |
| `BODY`         | 一个 JSON 格式的请求体 (如果请求需要的话)                    |

### 索引

```sense
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

注意，路径 `/megacorp/employee/1` 包含了三部分的信息：

- **`megacorp`**

  索引名称

- **`employee`**

  类型名称

- **`1`**

  特定雇员的ID



### 系统设计

并发：Elasticsearch 使用乐观锁控制并发，通过 `_version` 号来确保变更以正确顺序得到执行。

### 搜索

搜索关键字：
`_search`返回信息：`hits`,`took` 搜索耗时，`_shards` 参与查询分片数

### 技巧

```sense
/_search
在所有的索引中搜索所有的类型
/gb/_search
在 gb 索引中搜索所有的类型
/gb,us/_search
在 gb 和 us 索引中搜索所有的文档
/g*,u*/_search
在任何以 g 或者 u 开头的索引中搜索所有的类型
/gb/user/_search
在 gb 索引中搜索 user 类型
/gb,us/user,tweet/_search
在 gb 和 us 索引中搜索 user 和 tweet 类型
/_all/user,tweet/_search
在所有的索引中搜索 user 和 tweet 类型
```

#### 轻量级搜索

```sense
GET /megacorp/employee/_search
```

#### 查询表达式

```sense
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

#### 复杂查询

```sense
GET /megacorp/employee/_search
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

#### 全文查询

```sense
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

`Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。`

#### 短语搜索

```sense
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

#### 高亮查询

```sense
GET /megacorp/employee/_search
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

`highlight` 封装了查询数据

#### 聚合

```sense
# 统计 相当于 GROUP BY + count
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { 
        "field": "interests.keyword"
      }
    }
  }
}

# 汇总 相当于 GROUP BY + SUM
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests.keyword" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

`注意：` 一定要在 `interests` 后面加上`keyword` 不然会报错



#### 分页

```sense
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

#### 倒序索

传统的是存储ID作为索引，倒序索是将value值根据分词器拆分，重新进行排序。



### 数据输入和输出

#### 索引文件

一个文档的 `_index` 、 `_type` 和 `_id` 唯一标识一个文档。 我们可以提供自定义的 `_id` 值，或者让 `index` API 自动生成。

```sense
# 手动自定义id
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}

# 自动生成id
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
  
```

#### 返回

只返回data数据

```sense
GET /website/blog/123?pretty
```

 返回部分文档

```sense
GET /website/blog/123?_source=title,text
```

#### 检测文档是否存在

```js
curl -i -XHEAD http://localhost:9200/website/blog/123
```

#### 更新文档

update

1. 从旧文档构建 JSON
2. 更改该 JSON
3. 删除旧文档
4. 索引一个新文档

```sense
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```

#### 创建新文档

```jssense
1. POST /website/blog/
{ ... }
2. POST /website/blog/123
{ ... }
# 一下两种如果存再都会报错
3. PUT /website/blog/123?op_type=create
{ ... }
4. PUT /website/blog/123/_create
{ ... }
```

#### 删除文档

```sense
DELETE /website/blog/123
```

正如已经在[更新整个文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/update-doc.html)中提到的，删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。

#### 更新部分文档

`update` 请求最简单的一种形式是接收文档的一部分作为 `doc` 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。

```sense
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

脚本可以在 `update` API中用来改变 `_source` 的字段内容， 它在更新脚本中称为 `ctx._source`

```sense
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```

#### 取回多个文档

```js
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```



### 映射和分析

创建数据库表需要设置字段名称，类型，长度，约束等；索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做==映射(mapping)==。

映射数据说明：
字段名：任意填写，下面指定许多属性，例如：title、subtitle、images、price
==type==：类型，Elasticsearch 中支持的数据类型非常丰富，说几个关键的：
	String 类型，又分两种：
		text：可分词

​		keyword：不可分词，数据会作为完整字段进行匹配

==Numerical==：数值类型，分两类
		基本数据类型：long、integer、short、byte、double、float、half_float
		浮点数的高精度类型：scaled_float

==Date==：日期类型

==Array==：数组类型

==Object==：对象

==index==：是否索引，默认为 true，也就是说你不进行任何配置，所有字段都会被索引。
	true：字段会被索引，则可以用来进行搜索
	false：字段不会被索引，不能用来搜索

==store==：是否将数据进行独立存储，默认为 false
原始的文本会存储在_source 里面，默认情况下其他提取出来的字段都不是独立存储
的，是从_source 里面提取出来的。当然你也可以独立的存储某个字段，只要设置
"store": true 即可，获取独立存储的字段要比从_source 中解析快得多，但是也会占用
更多的空间，所以要根据实际业务需求来设置。

==analyzer==：分词器，这里的 ik_max_word 即使用 ik 分词器,后面会有专门的章节学习