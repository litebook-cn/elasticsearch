## JAVA实例：Bulk API

------

 Java高级REST客户端提供大容量处理器来帮助处理大容量请求。

**BulkRequest**

 BulkRequest可用于使用单个请求执行多个索引、更新和/或删除操作。

 它需要将至少一个操作添加到批量请求中:

```
BulkRequest request = new BulkRequest(); //创建BulkRequest
request.add(new IndexRequest("posts").id("1")  
        .source(XContentType.JSON,"field", "foo"));//将第一个索引请求添加到批量请求中
request.add(new IndexRequest("posts").id("2")  
        .source(XContentType.JSON,"field", "bar"));//添加第二个索引请求
request.add(new IndexRequest("posts").id("3")  
        .source(XContentType.JSON,"field", "baz"));//添加第三个索引请求
```

警告

 Bulk API只支持JSON或SMILE中编码的文档。以任何其他格式提供文档都将导致错误。

 不同的操作类型可以添加到同一个批量请求中:

```
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest("posts", "3")); //向批量请求添加删除请求
request.add(new UpdateRequest("posts", "2") 
        .doc(XContentType.JSON,"other", "test"));//向批量请求添加更新请求。
request.add(new IndexRequest("posts").id("4")  
        .source(XContentType.JSON,"field", "baz"));//使用SMILE格式添加索引请求
```

**可选参数**

```
request.timeout(TimeValue.timeValueMinutes(2)); //设置超时时间
request.timeout("2m"); //设置超时时间
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); //WriteRequest.RefreshPolicy实例形式设置刷新策略  
request.setRefreshPolicy("wait_for"); //字符串形式设置刷新策略                     
request.waitForActiveShards(2); //设置在继续索引/更新/删除操作之前必须活动的碎片副本的数量。
request.waitForActiveShards(ActiveShardCount.ALL); //作为动态硬装载提供的碎片副本数，可选：ActiveShardCount.ALL, ActiveShardCount.ONE或 ActiveShardCount.DEFAULT
request.pipeline("pipelineId"); //用于所有子请求的全局管道标识
request.routing("routingId"); //用于全局路由所有子请求
BulkRequest defaulted = new BulkRequest("posts"); //具有全局索引的批量请求，用于所有子请求，除非在子请求上被重写。此参数为@Nullable，只能在批量请求创建期间设置。
```

**同步执行**

 当以下列方式执行批量请求时，客户端在继续执行代码之前，会等待返回批量响应:

```
BulkResponse bulkResponse = client.bulk(request, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。



 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseException作为抑制异常添加到其中。

**异步执行**

 也可以异步方式执行BulkRequest，这样客户端就可以直接返回。用户需要通过将请求和侦听器传递给异步大容量方法来指定如何处理响应或潜在故障:

```
client.bulkAsync(request, RequestOptions.DEFAULT, listener); //要执行的批量请求和执行完成时要使用的操作侦听器
```

 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 一个典型的批量监听程序如下:

```
ActionListener<BulkResponse> listener = new ActionListener<BulkResponse>() {
    @Override
    public void onResponse(BulkResponse bulkResponse) {
        //成功的时候调用
    }

    @Override
    public void onFailure(Exception e) {
        //出错的时候调用
    }
};
```

**BulkResponse**

 返回的BulkResponse包含有关已执行操作的信息，并允许迭代每个结果，如下所示: 

```
for (BulkItemResponse bulkItemResponse : bulkResponse) { //迭代所有操作的结果
    DocWriteResponse itemResponse = bulkItemResponse.getResponse(); //检索操作的响应(成功与否)，可以是索引响应、更新响应或删除响应，它们都可以被视为DocWriteResponse实例

    switch (bulkItemResponse.getOpType()) {
    case INDEX:    //处理索引操作的响应
    case CREATE:
        IndexResponse indexResponse = (IndexResponse) itemResponse;
        break;
    case UPDATE:   //处理更新操作的响应
        UpdateResponse updateResponse = (UpdateResponse) itemResponse;
        break;
    case DELETE:   //处理删除操作的响应
        DeleteResponse deleteResponse = (DeleteResponse) itemResponse;
    }
}
```

 批量响应提供了一种快速检查一个或多个操作是否失败的方法:

```
if (bulkResponse.hasFailures()) { // 如果至少有一个操作失败，此方法返回true

}
```

 在这种情况下，有必要迭代所有操作结果，以检查操作是否失败，如果失败，则检索相应的失败:

```
for (BulkItemResponse bulkItemResponse : bulkResponse) {
    if (bulkItemResponse.isFailed()) { //指示给定操作是否失败
        BulkItemResponse.Failure failure =
                bulkItemResponse.getFailure(); //检索失败操作的失败
    }
}
```

**批量处理器**

 批量处理器通过提供一个实用程序类简化了Bulk API的使用，该类允许索引/更新/删除操作在添加到处理器时透明地执行。

 为了执行请求，批量处理器需要以下组件:

```
RestHighLevelClient
```

该客户端用于执行批量请求并检索BulkResponse

```
BulkProcessor.Listener
```

 每次批量请求执行前后或批量请求失败时，都会调用该侦听器

 那么批量处理器. builder方法可以用来构建一个新的批量处理器:

```
BulkProcessor.Listener listener = new BulkProcessor.Listener() { //创建BulkProcessor.Listener
    @Override
    public void beforeBulk(long executionId, BulkRequest request) {
        //每次执行BulkRequest之前都会调用此方法
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request,
            BulkResponse response) {
        //每次执行BulkRequest后都会调用此方法
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request,
            Throwable failure) {
        //当批量请求失败时调用此方法
    }
};
```

 BulkProcessor.Builder 提供了配置批量处理器如何处理请求执行的方法:

```
BulkProcessor bulkProcessor = BulkProcessor.builder(
        (request, bulkListener) ->
            client.bulkAsync(request, RequestOptions.DEFAULT, bulkListener),
        listener).build(); //通过从BulkProcessor.Builder调用build()方法来创建BulkProcessor。resthighleveloclient . BulkAsync()方法将用于在机罩下执行BulkRequest。
  BulkProcessor.Builder 供了配置批量处理器如何处理请求执行的方法:
BulkProcessor.Builder builder = BulkProcessor.builder(
        (request, bulkListener) ->
            client.bulkAsync(request, RequestOptions.DEFAULT, bulkListener),
        listener);
builder.setBulkActions(500); //根据当前添加的操作数设置刷新新批量请求的时间(默认值为1000，-1禁用)
builder.setBulkSize(new ByteSizeValue(1L, ByteSizeUnit.MB)); //根据当前添加的操作大小设置刷新新批量请求的时间(默认为5Mb，-1禁用)
builder.setConcurrentRequests(0); //设置允许执行的并发请求数(默认为1，0仅允许执行单个请求)
builder.setFlushInterval(TimeValue.timeValueSeconds(10L)); //设置一个刷新间隔，如果间隔过去，刷新任何待处理的批量请求(默认为未设置)
builder.setBackoffPolicy(BackoffPolicy
        .constantBackoff(TimeValue.timeValueSeconds(1L), 3)); //设置一个恒定的后退策略，最初等待1秒钟，最多重试3次。有关更多选项，请参见BackoffPolicy.noBackoff(), BackoffPolicy.constantBackoff()和BackoffPolicy.exponentialBackoff(). 
  一旦批量处理器被创建，可以向它添加请求:
IndexRequest one = new IndexRequest("posts").id("1")
        .source(XContentType.JSON, "title",
                "In which order are my Elasticsearch queries executed?");
IndexRequest two = new IndexRequest("posts").id("2")
        .source(XContentType.JSON, "title",
                "Current status and upcoming changes in Elasticsearch");
IndexRequest three = new IndexRequest("posts").id("3")
        .source(XContentType.JSON, "title",
                "The Future of Federated Search in Elasticsearch");
bulkProcessor.add(one);
bulkProcessor.add(two);
bulkProcessor.add(three);
```

 请求将由BulkProcessor执行，它负责为每个批量请求调用BulkProcessor.Listener。

 侦听器提供访问BulkRequest和BulkResponse的方法:

```
BulkProcessor.Listener listener = new BulkProcessor.Listener() {
    @Override
    public void beforeBulk(long executionId, BulkRequest request) {
        int numberOfActions = request.numberOfActions(); //在每次执行BulkRequest之前调用，这个方法允许知道在BulkRequest中将要执行的操作的数量
        logger.debug("Executing bulk [{}] with {} requests",
                executionId, numberOfActions);
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request,
            BulkResponse response) {
        if (response.hasFailures()) { //在每次执行BulkRequest后调用，该方法允许知道BulkResponse是否包含错误
            logger.warn("Bulk [{}] executed with failures", executionId);
        } else {
            logger.debug("Bulk [{}] completed in {} milliseconds",
                    executionId, response.getTook().getMillis());
        }
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request,
            Throwable failure) {
        logger.error("Failed to execute bulk", failure); //如果BulkRequest失败，则调用该方法，该方法允许知道失败
    }
};
```

 一旦所有请求都添加到BulkProcessor中，就需要使用两种可用的关闭方法之一来关闭它的实例。

 awaitClose()方法可用于等待，直到处理完所有请求或经过指定的等待时间:

```
boolean terminated = bulkProcessor.awaitClose(30L, TimeUnit.SECONDS); //如果所有大容量请求都已完成，则该方法返回true如果在所有大容量请求完成之前等待时间已过，则该方法返回false
```

 close()方法可用于立即关闭批量处理器:

```
bulkProcessor.close();
```

 这两种方法都会在关闭处理器之前刷新添加到处理器中的请求，并且禁止向其中添加任何新请求。