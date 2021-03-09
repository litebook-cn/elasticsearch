## JAVA实例：添加新文档（indexRequest）

------

 添加新文档需要调用IndexRequest请求，可以直接传递json数据，如下:

```
IndexRequest request = new IndexRequest("posts"); //索引
request.id("1"); //文档id
String jsonString = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";
request.source(jsonString, XContentType.JSON); //以字符串形式提供的文档源
```

 也可以使用Map作为参数，如下

```
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("user", "kimchy");
jsonMap.put("postDate", new Date());
jsonMap.put("message", "trying out Elasticsearch");
IndexRequest indexRequest = new IndexRequest("posts")
    .id("1").source(jsonMap); //以Map形式提供的文档源，可自动转换为JSON格式
```

 还可以使用XConttentBuilder构建内容。

```
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.field("user", "kimchy");
    builder.timeField("postDate", new Date());
    builder.field("message", "trying out Elasticsearch");
}
builder.endObject();
IndexRequest indexRequest = new IndexRequest("posts")
    .id("1").source(builder);
```

 直接用键值对对象构架数据。

```
IndexRequest indexRequest = new IndexRequest("posts")
    .id("1")
    .source("user", "kimchy",
        "postDate", new Date(),
        "message", "trying out Elasticsearch");
```

**可选参数**

 以下是官方文档提供的可选参数。

```
request.routing("routing"); //路由值
request.timeout(TimeValue.timeValueSeconds(1)); //设置超时
request.timeout("1s"); ////以字符串形式设置超时时间
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); //以WriteRequest.RefreshPolicy实例形式设置刷新策略
request.setRefreshPolicy("wait_for");//以字符串形式刷新策略                     
request.version(2); //文档版本
request.versionType(VersionType.EXTERNAL); //文档类型
request.opType(DocWriteRequest.OpType.CREATE); //操作类型
request.opType("create"); //操作类型 可选create或update
request.setPipeline("pipeline"); //索引文档之前要执行的摄取管道的名称
```

**同步执行**

 当以下列方式执行IndexRequest时，客户端在继续执行代码之前，会等待返回索引响应:

```
IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。



 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的弹性响应异常，并将原始响应异常作为抑制异常添加到其中。

**异步执行**

 我们也可以用异步方式执行IndexRequest，以便客户端可以直接返回。用户需要通过向异步索引方法传递请求和侦听器来指定如何处理响应或潜在故障:

```
client.indexAsync(request, RequestOptions.DEFAULT, listener);// listener是执行完成时要使用的侦听器
```

 异步方法不会阻塞并立即返回。一旦完成，如果执行成功完成，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同

 一个典型的listener像下面这样：

```
listener = new ActionListener<IndexResponse>() {
    @Override
    public void onResponse(IndexResponse indexResponse) {//执行成功的时候调用
        
    }

    @Override
    public void onFailure(Exception e) {//执行失败的时候调用
        
    }
};
```

**IndexResponse对象**

 返回的IndexResponse对象允许检索关于已执行操作的信息，如下所示:

```
String index = indexResponse.getIndex();
String id = indexResponse.getId();
if (indexResponse.getResult() == DocWriteResponse.Result.CREATED) {//处理创建文档的情况
    
} else if (indexResponse.getResult() == DocWriteResponse.Result.UPDATED) {//处理文档更新的情况
    
}
ReplicationResponse.ShardInfo shardInfo = indexResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {//处理成功的分片数少于总分片数时的情况
    
}
if (shardInfo.getFailed() > 0) {//处理潜在的故障
    for (ReplicationResponse.ShardInfo.Failure failure :
            shardInfo.getFailures()) {
        String reason = failure.reason(); 
    }
}
```

 如果存在版本冲突，将引发ElasticsearchException:

```
IndexRequest request = new IndexRequest("posts")
    .id("1")
    .source("field", "value")
    .setIfSeqNo(10L)
    .setIfPrimaryTerm(20);
try {
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        //引发的异常表明返回了版本冲突错误
    }
}
```

 如果opType被设置为创建并且已经存在具有相同索引和id的文档，也会发生同样的情况:

```
IndexRequest request = new IndexRequest("posts")
    .id("1")
    .source("field", "value")
    .opType(DocWriteRequest.OpType.CREATE);
try {
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        //引发的异常表明返回了版本冲突错误
    }
}
```

 文档参考地址：https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.1/java-rest-high-document-index.html