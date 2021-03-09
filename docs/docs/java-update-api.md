## JAVA实例：更新文档

------

**UpdateRequest**

```
UpdateRequest request = new UpdateRequest(
        "posts", //索引
        "1");   //文档id
```

 Update API允许通过使用脚本或传递部分文档来更新现有文档。

**用脚本更新** 

 该脚本可以作为内嵌脚本提供:

```
Map<String, Object> parameters = singletonMap("count", 4); //作为对象映射提供的脚本参数
Script inline = new Script(ScriptType.INLINE, "painless",
        "ctx._source.field += params.count", parameters);  //使用painless语言和前面的参数创建内嵌脚本
request.script(inline);  //将脚本设置为更新请求
```

 或者作为存储的脚本:

```
Script stored = new Script(
        ScriptType.STORED, null, "increment-field", parameters);  //引用painless语言中存储在名称增量字段下的脚本
request.script(stored);  //在更新请求中设置脚本
```

**用部分文档更新**

 对部分文档使用更新时，部分文档将与现有文档合并。

 部分文档可以以不同的方式提供:

```
UpdateRequest request = new UpdateRequest("posts", "1");
String jsonString = "{" +
        "\"updated\":\"2017-01-01\"," +
        "\"reason\":\"daily update\"" +
        "}";
request.doc(jsonString, XContentType.JSON); //以JSON格式的字符串形式提供的部分文档源
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("updated", new Date());
jsonMap.put("reason", "daily update");
UpdateRequest request = new UpdateRequest("posts", "1")
        .doc(jsonMap); //作为Map提供的部分文档源会自动转换为JSON格式
```



```
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.timeField("updated", new Date());
    builder.field("reason", "daily update");
}
builder.endObject();
UpdateRequest request = new UpdateRequest("posts", "1")
        .doc(builder);  //作为XContentBuilder对象提供的部分文档源，弹性搜索内置帮助器，用于生成JSON内容
UpdateRequest request = new UpdateRequest("posts", "1")
        .doc("updated", new Date(),
             "reason", "daily update"); //作为键值对提供的部分文档源，转换为JSON格式
```



**Upserts**

 如果文档尚不存在，可以使用upsert方法定义一些将作为新文档插入的内容:

```
String jsonString = "{\"created\":\"2017-01-01\"}";
request.upsert(jsonString, XContentType.JSON);  //以字符串形式提供的Upsert文档源
```

 与部分文档更新类似，可以使用接受字符串、Map、XContentBuilder或键值对的方法来定义upsert文档的内容。

**可选参数**

```
request.routing("routing"); //路由值
request.timeout(TimeValue.timeValueSeconds(1)); //设置超时
request.timeout("1s"); ////以字符串形式设置超时时间
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); //以WriteRequest.RefreshPolicy实例形式设置刷新策略
request.setRefreshPolicy("wait_for");//以字符串形式刷新策略  
request.retryOnConflict(3); //如果要更新的文档在更新操作的获取和索引阶段之间被另一个操作更改，重试更新操作的次数
request.fetchSource(true); //启用源检索，默认情况下禁用
String[] includes = new String[]{"updated", "r*"};
String[] excludes = Strings.EMPTY_ARRAY;
request.fetchSource(
        new FetchSourceContext(true, includes, excludes)); //为特定字段配置源包含
String[] includes = Strings.EMPTY_ARRAY;
String[] excludes = new String[]{"updated"};
request.fetchSource(
        new FetchSourceContext(true, includes, excludes)); //为特定字段配置源排除

request.setIfSeqNo(2L); //ifSeqNo
request.setIfPrimaryTerm(1L); //ifPrimaryTerm
request.detectNoop(false); //禁用noop检测
request.scriptedUpsert(true); //指出无论文档是否存在，脚本都必须运行，即如果文档不存在，脚本负责创建文档。
request.docAsUpsert(true); //指示如果部分文档尚不存在，则必须将其用作upsert文档。
request.waitForActiveShards(2); //设置在继续更新操作之前必须活动的碎片副本数量。
request.waitForActiveShards(ActiveShardCount.ALL); //ActiveShardCount的碎片副本数。可选值：ActiveShardCount.ALL, ActiveShardCount.ONE或者 ActiveShardCount.DEFAULT
```

**同步执行**

 当以下列方式执行更新请求时，客户端在继续执行代码之前，会等待返回更新响应:

```
UpdateResponse updateResponse = client.update(
        request, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。



 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseException作为抑制异常添加到其中。

**异步执行**

 执行更新请求也可以异步方式完成，以便客户端可以直接返回。用户需要通过向异步更新方法传递请求和侦听器来指定如何处理响应或潜在故障:

```
client.updateAsync(request, RequestOptions.DEFAULT, listener); //要执行的更新请求和执行完成时要使用的操作侦听器
```

 异步方法不会阻塞并立即返回。如果执行成功完成，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 典型的更新监听器如下所示:

```
listener = new ActionListener<UpdateResponse>() {
    @Override
    public void onResponse(UpdateResponse updateResponse) {
        //成功的时候调用
    }

    @Override
    public void onFailure(Exception e) {
        //失败的时候调用
    }
};
```

**Update Response**

 返回的更新响应允许检索关于已执行操作的信息，如下所示:

```
String index = updateResponse.getIndex();
String id = updateResponse.getId();
long version = updateResponse.getVersion();
if (updateResponse.getResult() == DocWriteResponse.Result.CREATED) {
    //处理第一次创建文档的情况(upsert)
} else if (updateResponse.getResult() == DocWriteResponse.Result.UPDATED) {
    //处理文档更新的情况
} else if (updateResponse.getResult() == DocWriteResponse.Result.DELETED) {
    //处理文档被删除的情况
} else if (updateResponse.getResult() == DocWriteResponse.Result.NOOP) {
   //处理文档不受更新影响的情况，即没有对文档执行任何操作(noop)   
}
```

 当通过fetchSource方法在更新请求中启用源检索时，响应包含更新文档的源:

```
GetResult result = updateResponse.getGetResult(); //以GetResult形式检索更新的文档
if (result.isExists()) {
    String sourceAsString = result.sourceAsString(); //以字符串形式检索更新文档的来源
    Map<String, Object> sourceAsMap = result.sourceAsMap(); //以Map<String, Object>的形式检索更新文档的源
    byte[] sourceAsBytes = result.source(); //以byte[]的形式检索更新文档的源
} else {
    //处理响应中不存在文档源的情况(默认情况下就是这种情况)
}
```

 也可以检查碎片故障:

```
ReplicationResponse.ShardInfo shardInfo = updateResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    //处理成功碎片数少于总碎片数的情况
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure :
            shardInfo.getFailures()) {//处理潜在的故障
        String reason = failure.reason(); 
    }
}
```

 当对不存在的文档执行UpdateRequest时，响应有404个状态代码，会引发一个ElasticsearchException，需要如下处理:

```
UpdateRequest request = new UpdateRequest("posts", "does_not_exist")
        .doc("field", "value");
try {
    UpdateResponse updateResponse = client.update(
            request, RequestOptions.DEFAULT);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        //处理由于文档不存在而引发的异常
    }
}
```

 如果存在版本冲突，将引发ElasticsearchException:

```
UpdateRequest request = new UpdateRequest("posts", "1")
        .doc("field", "value")
        .setIfSeqNo(101L)
        .setIfPrimaryTerm(200L);
try {
    UpdateResponse updateResponse = client.update(
            request, RequestOptions.DEFAULT);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        //引发的异常表明返回了版本冲突错误。
    }
}
```