## JAVA实例：获取文档

------

**GetRequest**

 要获取一个文档，需要使用GetRequest对象，GetRequest的调用如下:

```
GetRequest getRequest = new GetRequest(
        "posts", //索引名称
        "1");   //文档id
```

**可选参数**

```
request.fetchSourceContext(FetchSourceContext.DO_NOT_FETCH_SOURCE); //禁用源检索，默认情况下启用

String[] includes = new String[]{"message", "*Date"};
String[] excludes = Strings.EMPTY_ARRAY;
FetchSourceContext fetchSourceContext =
        new FetchSourceContext(true, includes, excludes);
request.fetchSourceContext(fetchSourceContext); //为特定字段配置源包含

String[] includes = Strings.EMPTY_ARRAY;
String[] excludes = new String[]{"message"};
FetchSourceContext fetchSourceContext =
        new FetchSourceContext(true, includes, excludes);
request.fetchSourceContext(fetchSourceContext); //为特定字段配置源排除

request.storedFields("message"); //配置特定存储字段的检索(要求字段在映射中单独存储)
GetResponse getResponse = client.get(request, RequestOptions.DEFAULT);
String message = getResponse.getField("message").getValue(); //检索消息存储字段(要求该字段单独存储在映射中)

request.routing("routing"); //路由值

request.preference("preference"); //偏好值

request.realtime(false); //将realtime标志设置为false

request.refresh(true); //在检索文档之前执行刷新(默认为false)

request.version(2); //版本

request.versionType(VersionType.EXTERNAL); //版本类型
```

**同步执行**

 当以下列方式执行GetRequest时，客户端会在继续执行代码之前等待GetResponse返回:

```
GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。



 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseException作为抑制异常添加到其中。

**异步执行**

 执行GetRequest也可以异步方式完成，这样客户端就可以直接返回。用户需要通过向异步get方法传递请求和侦听器来指定如何处理响应或潜在故障:

```
client.getAsync(request, RequestOptions.DEFAULT, listener); //要执行的GetRequest和执行完成时要使用的ActionListener
```

 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 一个典型的listener如下：

```
ActionListener<GetResponse> listener = new ActionListener<GetResponse>() {
    @Override
    public void onResponse(GetResponse getResponse) {//执行成功的时候调用
        
    }
    @Override
    public void onFailure(Exception e) {//出错的时候调用
        
    }
}
```

**GetResponse对象**

 返回的GetResponse允许检索请求的文档及其元数据和最终存储的字段。

```
String index = getResponse.getIndex();
String id = getResponse.getId();
if (getResponse.isExists()) {
    long version = getResponse.getVersion();
    String sourceAsString = getResponse.getSourceAsString(); //以字符串形式检索文档  
    Map<String, Object> sourceAsMap = getResponse.getSourceAsMap(); //以Map<String, Object>的形式检索文档
    byte[] sourceAsBytes = getResponse.getSourceAsBytes(); //以byte[]形式检索文档   
} else {
    
}
```

 处理找不到文档的情况。请注意，虽然返回的响应有404个状态代码，但返回的是有效的GetResponse，而不是引发的异常。这种响应不包含任何源文档，其isExists方法返回false。

 当对不存在的索引执行get请求时，响应具有404状态代码，会引发一个ElasticsearchException，需要按如下方式处理:

```
GetRequest request = new GetRequest("does_not_exist", "1");
try {
    GetResponse getResponse = client.get(request, RequestOptions.DEFAULT);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        //处理因索引不存在而引发的异常
    }
}
```

 如果请求了特定的文档版本，并且现有文档具有不同的版本号，则会引发版本冲突: 

```
try {
    GetRequest request = new GetRequest("posts", "1").version(2);
    GetResponse getResponse = client.get(request, RequestOptions.DEFAULT);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        //引发的异常表明返回了版本冲突错误
    }
}
```