## JAVA实例：Delete By Query API

------

**DeleteByQueryRequest**

 DeleteByQueryRequest可用于从索引中删除文档。它需要一个要执行删除的现有索引(或一组索引)。

 DeleteByQueryRequest的最简单形式如下，它删除索引中的所有文档:

```
DeleteByQueryRequest request =
        new DeleteByQueryRequest("source1", "source2"); //在一组索引上创建DeleteByQueryRequest。
```

 默认情况下，版本冲突会中止DeleteByQueryRequest进程，但您可以用以下方法对它们进行计数:

```
request.setConflicts("proceed"); //设置版本冲突时继续
```

 您可以通过添加查询来限制文档。

```
request.setQuery(new TermQueryBuilder("user", "kimchy")); //仅复制字段用户设置为kimchy的文档
```

 也可以通过设置大小来限制已处理文档的数量。

```
request.setSize(10); //只拷贝十个文档
```

 默认情况下，DeleteByQueryRequest可以批量处理1000个文档。您可以使用setBatchSize更改批处理大小。

```
request.setBatchSize(100); //单批处理100份文件
```

 DeleteByQueryRequest也可以使用带有设置切片的切片滚动来并行化:

```
request.setSlices(2); //设置要使用的切片数量
```

 DeleteByQueryRequest使用Scroll参数来控制它将“搜索上下文”保持多长时间。

```
request.setScroll(TimeValue.timeValueMinutes(10)); //设置滚动时间
```

 如果您提供了路由，则路由将被复制到滚动查询，从而将该过程限制在与该路由值匹配的碎片上。

```
request.setRouting("=cat"); //设置路由
```

**可选参数**

 除了上述选项之外，还可以选择提供以下参数:

```
request.setTimeout(TimeValue.timeValueMinutes(2)); //等待按查询删除请求作为时间值执行的超时
request.setRefresh(true); //调用查询删除后刷新索引
request.setIndicesOptions(IndicesOptions.LENIENT_EXPAND_OPEN); //设置索引选项
```

**同步执行**

 当以下列方式执行DeleteByQueryRequest时，客户端会在继续执行代码之前等待返回DeleteByQueryResponse:

```
BulkByScrollResponse bulkResponse =
        client.deleteByQuery(request, RequestOptions.DEFAULT);
```



 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。

 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseExceptio作为抑制异常添加到其中。

**异步执行**

 也可以异步方式执行DeleteByQueryRequest，以便客户端可以直接返回。用户需要通过将请求和侦听器传递给异步按查询删除方法来指定如何处理响应或潜在故障:

```
client.deleteByQueryAsync(request, RequestOptions.DEFAULT, listener); //要执行的DeleteByQueryRequest和执行完成时要使用的操作侦听器
```

 异步方法不会阻塞并立即返回。一旦完成，如果执行成功完成，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 delete-by-query的典型侦听器如下所示:

```
listener = new ActionListener<BulkByScrollResponse>() {
    @Override
    public void onResponse(BulkByScrollResponse bulkResponse) {
        //成功的时候调用
    }
    @Override
    public void onFailure(Exception e) {
        //失败的时候调用
    }
};
```

**DeleteByQueryResponse**

 返回的DeleteByQueryResponse包含有关已执行操作的信息，并允许迭代每个结果，如下所示:

```
TimeValue timeTaken = bulkResponse.getTook(); //获取总时间
boolean timedOut = bulkResponse.isTimedOut(); //检查请求是否超时
long totalDocs = bulkResponse.getTotal(); //获取处理的文档总数
long deletedDocs = bulkResponse.getDeleted(); //已删除的文档数
long batches = bulkResponse.getBatches(); //已执行的批次数量
long noops = bulkResponse.getNoops(); //跳过的文档数
long versionConflicts = bulkResponse.getVersionConflicts(); //版本冲突的数量
long bulkRetries = bulkResponse.getBulkRetries(); //请求必须重试批量索引操作的次数
long searchRetries = bulkResponse.getSearchRetries(); //请求必须重试搜索操作的次数
TimeValue throttledMillis = bulkResponse.getStatus().getThrottled(); //如果当前处于睡眠状态，此请求限制自身的总时间不包括当前限制时间
TimeValue throttledUntilMillis =
        bulkResponse.getStatus().getThrottledUntil(); //任何当前休眠的剩余延迟，或者如果没有休眠，则为0
List<ScrollableHitSource.SearchFailure> searchFailures =
        bulkResponse.getSearchFailures(); //搜索阶段失败
List<BulkItemResponse.Failure> bulkFailures =
        bulkResponse.getBulkFailures(); //批量索引操作期间失败
```