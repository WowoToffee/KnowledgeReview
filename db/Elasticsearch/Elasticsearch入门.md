# Elasticsearch入门

  `一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：`

```xml
 curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
  被 < > 标记的部件：
VERB

适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。

PROTOCOL

http 或者 https（如果你在 Elasticsearch 前面有一个 https 代理）

HOST

Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。

PORT

运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。

PATH

API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。

QUERY_STRING

任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)

BODY

一个 JSON 格式的请求体 (如果请求需要的话)
```

简单操作：

添加数据：

```
curl -X PUT "localhost:9200/megacorp/employee/1?pretty" -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
'
```

查询数据：

```
# ID 查询
curl -X GET "localhost:9200/megacorp/employee/1?pretty"

# 查询所有
curl -X GET "localhost:9200/megacorp/employee/_search?pretty"

# 高亮 搜索
curl -X GET "localhost:9200/megacorp/employee/_search?q=last_name:Smith&pretty"

# JSON 查询
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
'

```

 复制查询：

```
# 同样搜索姓氏为 Smith 的员工，但这次我们只需要年龄大于 30 的
curl -X GET -u undefined:$ESPASS "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
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
'

```



