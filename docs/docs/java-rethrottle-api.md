## JAVA实例：Rethrottle API

------

**RethrottleRequest**

 RethrottleRequest可用于更改正在运行的reindex, update-by-query或delete-by-query的当前限制，或者完全禁用任务的限制，它要求更改任务的任务标识。

 在其最简单的形式中，您可以使用它来禁用运行任务的throttling，方法如下:

```
RethrottleRequest request = new RethrottleRequest(taskId); //创建一个RethrottleRequest，禁用特定任务id的限制
```

 通过提供requestsPerSecond参数，请求将现有任务限制更改为指定值:

```
RethrottleRequest request = new RethrottleRequest(taskId, 100.0f); //将任务限制更改为每秒100个请求
```

 根据reindex, update-by-query和delete-by-query任务是否应该重新排序，可以使用三种适当方法之一来执行重新排序请求:

```
client.reindexRethrottle(request, RequestOptions.DEFAULT);  //执行重新索引重新调用请求
client.updateByQueryRethrottle(request, RequestOptions.DEFAULT); //按查询更新也是如此
client.deleteByQueryRethrottle(request, RequestOptions.DEFAULT); //按查询删除也是如此
```

**异步执行**

 rethrottle请求的异步执行要求将rethrottle请求实例和动作监听器实例都传递给异步方法:

```
client.reindexRethrottleAsync(request,
        RequestOptions.DEFAULT, listener); //异步执行重新索引重新计时
client.updateByQueryRethrottleAsync(request,
        RequestOptions.DEFAULT, listener); //按查询更新也是如此
client.deleteByQueryRethrottleAsync(request,
        RequestOptions.DEFAULT, listener); //按查询删除也是如此
```



 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。典型的监听器是这样的:

```
listener = new ActionListener<ListTasksResponse>() {
    @Override
    public void onResponse(ListTasksResponse response) {
        //成功的时候调用
    }

    @Override
    public void onFailure(Exception e) {
        //失败的时候调用
    }
};
```



**Rethrottle Response**

 Rethrottling以ListTasksResponse的形式返回已经重新启动的任务。List Tasks API将详细描述该响应对象的结构。