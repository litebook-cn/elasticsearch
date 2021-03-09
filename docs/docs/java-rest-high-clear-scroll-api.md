## JAVA实例：Clear Scroll API

------

**Clear Scroll API**

 当滚动超时时，Search Scroll API使用的搜索上下文会自动删除。但是建议使用Clear Scroll API，一旦不再需要搜索上下文，就立即释放它们。

**ClearScrollRequest**

 可以按如下方式创建一个ClearScrollRequest:

```
ClearScrollRequest request = new ClearScrollRequest(); //创建一个新的ClearScrollRequest
request.addScrollId(scrollId); //将滚动id添加到要清除的滚动标识符列表中
```

**提供滚动标识符**

 ClearScrollRequest允许在单个请求中清除一个或多个滚动标识符。

 滚动标识符可以一个接一个地添加到请求中:

```
request.addScrollId(scrollId);
```

 或者一起使用:

```
request.setScrollIds(scrollIds);
```

**同步方式执行**

```
ClearScrollResponse response = client.clearScroll(request, RequestOptions.DEFAULT);
```

**异步方式执行**

 ClearScrollRequest的异步执行需要将ClearScrollRequest实例和ActionListener实例都传递给异步方法:

```
client.clearScrollAsync(request, RequestOptions.DEFAULT, listener);
```



 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。

 一个典型的ClearScrollResponse监听器像下面这样：

```
ActionListener<ClearScrollResponse> listener =
        new ActionListener<ClearScrollResponse>() {
    @Override
    public void onResponse(ClearScrollResponse clearScrollResponse) {
        //成功的时候调用
    }

    @Override
    public void onFailure(Exception e) {
        //失败的时候调用
    }
};
```

**ClearScrollResponse**

 返回的ClearScrollResponse允许检索关于发布的搜索上下文的信息:

```
boolean success = response.isSucceeded(); //如果请求成功返回true
int released = response.getNumFreed(); //返回发布的搜索上下文的数量
```