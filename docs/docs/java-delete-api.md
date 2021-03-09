## JAVA实例：删除文档

------

**DeleteRequest**

 想要删除一个文档，必须构建一个DeleteRquest，如下：

```
DeleteRequest request = new DeleteRequest(
        "posts",    //索引
        "1");       //文档id
```

**可选参数**

 DeleteRequest同样有一些可选参数：

```
request.routing("routing"); //路由值
request.timeout(TimeValue.timeValueMinutes(2)); //以TimeValue形式设置超时
request.timeout("2m");  //以字符串形式设置超时
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); //以WriteRequest.RefreshPolicy实例的形式设置刷新策略
request.setRefreshPolicy("wait_for");  //以字符串的形式设置刷新策略            
request.version(2); //版本
request.versionType(VersionType.EXTERNAL); //版本类型
```

**同步执行**

 当以下列方式执行删除请求时，客户端在继续执行代码之前，会等待返回删除响应:

```
DeleteResponse deleteResponse = client.delete(
        request, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。



 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseException作为抑制异常添加到其中。

**异步执行**

 也可以异步方式执行DeleteRequest，以便客户端可以直接返回而无需等待。用户需要通过向异步删除方法传递请求和侦听器来指定如何处理响应或潜在问题:

```
client.deleteAsync(request, RequestOptions.DEFAULT, listener); //要执行的删除请求和执行完成时要使用的操作侦听器
```

 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 一个典型的监听器如下：

```
listener = new ActionListener<DeleteResponse>() {
    @Override
    public void onResponse(DeleteResponse deleteResponse) {
        //执行成功时调用
    }

    @Override
    public void onFailure(Exception e) {
        //执行失败时调用
    }
};
```

**DeleteResponse**

 返回的DeleteResponse允许检索关于已执行操作的信息，如下所示:

```
String index = deleteResponse.getIndex();
String id = deleteResponse.getId();
long version = deleteResponse.getVersion();
ReplicationResponse.ShardInfo shardInfo = deleteResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    //处理成功分片数少于总分片数的情况
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure :
            shardInfo.getFailures()) {//处理潜在的故障
        String reason = failure.reason(); 
    }
}
```

 还可以检查文档是否被找到:

```
DeleteRequest request = new DeleteRequest("posts", "does_not_exist");
DeleteResponse deleteResponse = client.delete(
        request, RequestOptions.DEFAULT);
if (deleteResponse.getResult() == DocWriteResponse.Result.NOT_FOUND) {
    //如果找不到要删除的文档，执行一些操作
}
```

 如果存在版本冲突，将引发弹性响应异常:

```
try {
    DeleteResponse deleteResponse = client.delete(
        new DeleteRequest("posts", "1").setIfSeqNo(100).setIfPrimaryTerm(2),
            RequestOptions.DEFAULT);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        //引发的异常表明返回了版本冲突错误
    }
}
```