## JAVA实例：Multi Term Vectors API

------

 Multi Term Vectors API允许一次获得多个向量。

**Multi Term Vectors Request**

 创建MultiTermVectorsRequest有两种方法。

 第一种方法是创建一个空的MultiTermVectorsRequest，然后向其中添加单独的TermVectorsRequest求。

```
MultiTermVectorsRequest request = new MultiTermVectorsRequest(); //创建一个空的MultiTermVectorsRequest。
TermVectorsRequest tvrequest1 =
    new TermVectorsRequest("authors", "1");
tvrequest1.setFields("user");
request.add(tvrequest1); //将第一个术语向量请求添加到多术语向量请求中。

XContentBuilder docBuilder = XContentFactory.jsonBuilder();
docBuilder.startObject().field("user", "guest-user").endObject();
TermVectorsRequest tvrequest2 =
    new TermVectorsRequest("authors", docBuilder);
request.add(tvrequest2); //将人工文档的第二个术语请求添加到多术语请求中。
```



 当所有TermVectorsRequest共享相同的参数(如索引和其他设置)时，可以使用第二种方法。在这种情况下，可以创建一个模板TermVectorsRequest，并设置所有必要的设置，该模板请求可以与执行这些请求的所有文档id一起传递给MultiTermVectorsRequest。

```
TermVectorsRequest tvrequestTemplate =
    new TermVectorsRequest("authors", "fake_id"); //创建模板TermVectorsRequest。
tvrequestTemplate.setFields("user");
String[] ids = {"1", "2"};
MultiTermVectorsRequest request =
    new MultiTermVectorsRequest(ids, tvrequestTemplate); //将文档的id和模板传递给MultiTermVectorsRequest。
```

**同步执行**

 当以下列方式执行MultiTermVectorsRequest时，客户端会在继续执行代码之前等待返回MultiTermVectorsResponse:

```
MultiTermVectorsResponse response =
    client.mtermvectors(request, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。

 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseException作为抑制异常添加到其中。

**异步执行**

 也可以异步方式执行MultiTermVectorsRequest，以便客户端可以直接返回。用户需要通过将请求和侦听器传递给异步MultiTermVectors方法来指定如何处理响应或潜在故障:

```
client.mtermvectorsAsync(
    request, RequestOptions.DEFAULT, listener);
```

 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 MultiTermVectors的典型监听器如下所示:

```
listener = new ActionListener<MultiTermVectorsResponse>() {
    @Override
    public void onResponse(MultiTermVectorsResponse mtvResponse) {
        
    }
    @Override
    public void onFailure(Exception e) {
        
    }
};
```



**Multi Term Vectors Response**

 MultiTermVectorsResponse允许获得TermVectorsResponse的列表，每个响应都可以按照Term Vectors API中的描述进行检查。

```
List<TermVectorsResponse> tvresponseList =
    response.getTermVectorsResponses(); //获取MultiTermVectorsResponse列表
if (tvresponseList != null) {
    for (TermVectorsResponse tvresponse : tvresponseList) {
    }
}
```