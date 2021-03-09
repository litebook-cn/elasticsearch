## JAVA实例：搜索API

------

 大多数搜索API是多索引、多类型的，Explain API端点除外。

路由

 执行搜索时，它将被广播到所有索引/索引碎片(副本之间的循环)。可以通过提供路由参数来控制搜索哪些碎片。例如，索引tweets时，路由值可以是用户名:

```
POST /twitter/tweet?routing=kimchy
{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

 在这种情况下，如果我们只想在tweets中搜索特定用户，我们可以将其指定为路由，导致搜索只命中相关碎片:

```
POST /twitter/tweet/_search?routing=kimchy
{
    "query": {
        "bool" : {
            "must" : {
                "query_string" : {
                    "query" : "some query string here"
                }
            },
            "filter" : {
                "term" : { "user" : "kimchy" }
            }
        }
    }
}
```

 路由参数可以是用逗号分隔的字符串表示的多值。这将导致命中路由值匹配的相关碎片。

**统计组**

 搜索可以与统计组相关联，统计组维护每个组的统计聚合。稍后可以特别使用索引统计应用编程接口来检索它。例如，这里有一个搜索主体请求，它将请求与两个不同的组相关联:

```
POST /_search
{
    "query" : {
        "match_all" : {}
    },
    "stats" : ["group1", "group2"]
}
```

**全局搜索超时**

 作为搜索请求的一部分，单个搜索可以设置超时。由于搜索请求可以来自许多来源，Elasticsearch具有动态集群级全局搜索超时的设置，适用于在搜索请求正文中未设置超时的所有请求。默认值是无全局超时。设置键是search.default_search_timeout，可以使用群集更新设置端点进行设置。将该值设置为-1将全局搜索超时重置为无超时。

**搜索取消**

 我们可以使用标准任务取消机制取消搜索。默认情况下，正在运行的搜索只检查它是否在段边界上被取消，因此取消可能会被较大的段延迟。通过将动态集群级设置search . low_level_cancel设置为true，可以提高搜索取消响应速度。然而，它伴随着更频繁的取消检查的额外开销，这在大型快速运行的搜索查询中是显而易见的。更改此设置只会影响更改后开始的搜索。