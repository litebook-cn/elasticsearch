## JAVA实例：Multi-Search API

------

**Multi-Search API**

 multiSearch API在一个http请求中并行执行多个搜索请求。

**Multi-SearchRequest**

 MultiSearchRequest的构造函数是空的，您可以将所有希望执行的搜索添加到其中:

```
MultiSearchRequest request = new MultiSearchRequest(); //创建一个空的MultiSearchRequest 。
SearchRequest firstSearchRequest = new SearchRequest();//创建一个空的SearchRequest，并像常规搜索一样填充它。
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(QueryBuilders.matchQuery("user", "kimchy"));
firstSearchRequest.source(searchSourceBuilder);
request.add(firstSearchRequest); //将SearchRequest添加到MultiSearchRequest中。
SearchRequest secondSearchRequest = new SearchRequest();  //构建第二个SearchRequest，并将其添加到MultiSearchRequest中。
searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(QueryBuilders.matchQuery("user", "luca"));
secondSearchRequest.source(searchSourceBuilder);
request.add(secondSearchRequest);
```

**可选参数**

 MultiSearchRequest支持所有的SearchRequest参数，例如：

```
SearchRequest searchRequest = new SearchRequest("posts"); //将请求限制为索引
```

**同步方式执行**

```
MultiSearchResponse response = client.msearch(request, RequestOptions.DEFAULT);
```

**异步方式执行**

```
client.searchAsync(searchRequest, RequestOptions.DEFAULT, listener);
```

 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。

 MultiSearchResponse的典型监听器如下所示:

```
ActionListener<MultiSearchResponse> listener = new ActionListener<MultiSearchResponse>() {
    @Override
    public void onResponse(MultiSearchResponse response) {
        //成功的时候调用
    }

    @Override
    public void onFailure(Exception e) {
        //失败的时候调用
    }
};
```

**MultiSearchResponse**

 通过执行MultiSearchRequest方法返回MultiSearchResponse。 如果请求失败，每个MultiSearchResponse.Item都包含getFailure中的异常；如果请求成功，则包含getResponse中的搜索响应:

```
MultiSearchResponse.Item firstResponse = response.getResponses()[0]; //第一次搜索
assertNull(firstResponse.getFailure()); // 它执行成功了，所以getFailure返回null。                 
SearchResponse searchResponse = firstResponse.getResponse();//getResponse中有一个searchResponse。
assertEquals(4, searchResponse.getHits().getTotalHits().value);
MultiSearchResponse.Item secondResponse = response.getResponses()[1];//第二次搜索
assertNull(secondResponse.getFailure());
searchResponse = secondResponse.getResponse();
assertEquals(1, searchResponse.getHits().getTotalHits().value);
```