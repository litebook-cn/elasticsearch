## JAVA实例：Reindex API

------

**ReindexRequest**

 ReindexRequest可用于将文档从一个或多个索引复制到目标索引中。

 它需要一个现有的源索引和一个可能存在也可能不存在的目标索引。Reindex不会尝试设置目标索引，也不会复制源索引的设置。您应该在运行_reindex操作之前设置目标索引，包括设置映射、碎片计数、副本等。

 ReindexRequest的最简单形式如下:

```
ReindexRequest request = new ReindexRequest(); //创建ReindexRequest
request.setSourceIndices("source1", "source2"); //添加要从源中复制的列表
request.setDestIndex("dest");  //添加目标索引
```

 目标元素可以像索引API一样配置来控制乐观并发控制。忽略versionType(如上)或将其设置为内部将导致Elasticsearch盲目地将文档转储到目标中。将版本类型设置为外部将导致Elasticsearch保留源版本，创建任何丢失的文档，并更新目标索引中版本比源索引中版本旧的文档。

```
request.setDestVersionType(VersionType.EXTERNAL); //设置versionType为EXTERNAL
```



 将opType设置为create导致_reindex仅在目标索引中创建丢失的文档。所有现有文档都将导致版本冲突。默认opType是index。



```
request.setDestOpType("create"); //设置versionType为create
```

 默认情况下，版本冲突会中止_reindex进程，但您可以用以下方法来计算它们:

```
request.setConflicts("proceed"); //设置版本冲突时继续
```

 您可以通过添加查询来限制文档。

```
request.setSourceQuery(new TermQueryBuilder("user", "kimchy")); /仅复制字段用户设置为kimchy的文档
```

 也可以通过设置大小来限制已处理文档的数量。

```
request.setSize(10); //只拷贝10个文档
```

 默认情况下_reindex单次可以处理1000个文档。您可以使用sourceBatchSize更改批处理大小。

```
request.setSourceBatchSize(100); //单次处理100个文档
```

 Reindex还可以通过指定管道来使用摄取功能。

```
request.setDestPipeline("my_pipeline"); //设置管线为my_pipeline
```

 如果您想从源索引中获得一组特定的文档，您需要使用sort。如果可能的话，最好选择一个更有选择性的查询，而不是大小和排序。

```
request.addSortField("field1", SortOrder.DESC); //将降序排序添加到`field1`
request.addSortField("field2", SortOrder.ASC); //将升序排序添加到field2
  ReindexRequest还支持修改文档的脚本。它还允许您更改文档的元数据。下面的例子说明了这一点。
request.setScript(
    new Script(
        ScriptType.INLINE, "painless",
        "if (ctx._source.user == 'kimchy') {ctx._source.likes++;}",
        Collections.emptyMap())); //设置脚本来增加用户kimchy的所有文档中的likes字段。
```



 ReindexRequest支持从远程Elasticsearch集群重新索引。当使用远程群集时，应该在RemoteInfo对象中指定查询，而不是使用setSourceQuery。如果同时设置了远程信息和源查询，则会在请求过程中导致验证错误。原因是远程Elasticsearch可能不理解现代查询构建器构建的查询。远程集群支持一直工作到Elasticsearch0.90，从那时起查询语言发生了变化。当达到旧版本时，用JSON手工编写查询更安全。

```
request.setRemoteInfo(
    new RemoteInfo(
        "http", remoteHost, remotePort, null,
        new BytesArray(new MatchAllQueryBuilder().toString()),
        user, password, Collections.emptyMap(),
        new TimeValue(100, TimeUnit.MILLISECONDS),
        new TimeValue(100, TimeUnit.SECONDS)
    )
); //设置远程elastic集群
```

 ReindexRequest也有助于使用切片滚动对_uid进行切片来自动并行化。使用设置切片指定要使用的切片数量。

```
request.setSlices(2); //设置使用的切片数
```

 ReindexRequest使用Scroll参数来控制它保持“搜索上下文”活动的时间。

```
request.setScroll(TimeValue.timeValueMinutes(10)); //设置滚动时间
```

**可选参数**

 除了上述选项之外，还可以选择提供以下参数:

```
request.setTimeout(TimeValue.timeValueMinutes(2)); //等待重新索引请求作为时间值执行的超时
request.setRefresh(true); //调用reindex后刷新索引
```

**同步执行**

 当以下列方式执行ReindexRequest时，客户端在继续执行代码之前，会等待返回BulkByScrollResponse:

```
BulkByScrollResponse bulkResponse =
        client.reindex(request, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。

 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseException作为抑制异常添加到其中。

**异步执行**

 执行ReindexRequest也可以异步方式完成，以便客户端可以直接返回。用户需要通过将请求和侦听器传递给异步reindex方法来指定如何处理响应或潜在故障:

```
client.reindexAsync(request, RequestOptions.DEFAULT, listener); //要执行的ReindexRequest和执行完成时要使用的ActionListener
```

 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 reindex的典型侦听器如下:

```
listener = new ActionListener<BulkByScrollResponse>() {
    @Override
    public void onResponse(BulkByScrollResponse bulkResponse) {
        //成功的时候执行
    }
    @Override
    public void onFailure(Exception e) {
        //失败的时候执行
    }
};
```

**重新索引任务提交**

 也可以使用Task API提交一个eindexRequest，而不是等待它完成。这相当于将等待完成标志设置为false的REST请求。

```
ReindexRequest reindexRequest = new ReindexRequest(); //ReindexRequest的构造方式与同步方法相同
reindexRequest.setSourceIndices(sourceIndex);
reindexRequest.setDestIndex(destinationIndex);
reindexRequest.setRefresh(true);
TaskSubmissionResponse reindexSubmission = highLevelClient()
    .submitReindexTask(reindexRequest, RequestOptions.DEFAULT); //提交方法返回包含任务标识符的TaskSubmissionResponse。
String taskId = reindexSubmission.getTask(); //任务标识符可用于获取已完成任务的响应。
ReindexResponse
```

 返回的BulkByScrollResponse包含有关已执行操作的信息，并允许迭代每个结果，如下所示:

```
TimeValue timeTaken = bulkResponse.getTook(); //获取总时间
boolean timedOut = bulkResponse.isTimedOut(); //检查请求是否超时
long totalDocs = bulkResponse.getTotal(); //获取处理的文档总数
long updatedDocs = bulkResponse.getUpdated(); //已更新的文档数
long createdDocs = bulkResponse.getCreated(); //创建的文档数
long deletedDocs = bulkResponse.getDeleted(); //已删除的文档数
long batches = bulkResponse.getBatches(); //已执行的批次数量
long noops = bulkResponse.getNoops(); //跳过的文档数
long versionConflicts = bulkResponse.getVersionConflicts(); //版本冲突的数量
long bulkRetries = bulkResponse.getBulkRetries(); //请求必须重试批量索引操作的次数
long searchRetries = bulkResponse.getSearchRetries(); //请求必须重试搜索操作的次数
TimeValue throttledMillis = bulkResponse.getStatus().getThrottled(); //如果当前处于睡眠状态，此请求限制自身的总时间不包括当前限制时间
TimeValue throttledUntilMillis =
        bulkResponse.getStatus().getThrottledUntil(); //任何当前油门休眠的剩余延迟，或者如果没有休眠，则为0
List<ScrollableHitSource.SearchFailure> searchFailures =
        bulkResponse.getSearchFailures(); //搜索阶段失败
List<BulkItemResponse.Failure> bulkFailures =
        bulkResponse.getBulkFailures(); //批量索引操作期间失败
```