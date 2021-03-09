## JAVA实例：术语向量(Term Vectors)API

------

 术语向量API返回特定文档字段中术语的信息和统计信息。文档可以存储在索引中或者由用户人工提供。

**TermVectorsRequest**

 一个TermVectorsRequest需要一个索引、一个类型和一个id来指定某个文档以及为其检索信息的字段。

```
TermVectorsRequest request = new TermVectorsRequest("authors", "1");
request.setFields("user");
```

 也可以为人工文档生成术语向量，即索引中不存在的文档:

```
XContentBuilder docBuilder = XContentFactory.jsonBuilder();
docBuilder.startObject().field("user", "guest-user").endObject();
TermVectorsRequest request = new TermVectorsRequest("authors",
    docBuilder);
```

 一个人工文档作为XContentBuilder对象提供，XContentBuilder对象是生成JSON内容的Elasticsearch内置帮助器。

**可选参数**

```
request.setFieldStatistics(false); //将字段统计设置为false(默认为true)以忽略文档统计信息。
request.setTermStatistics(true); //将术语统计设置为true(默认为false)，以显示总术语频率和文档频率。
request.setPositions(false); //将位置设置为false(默认为true)，以忽略位置输出。
request.setOffsets(false); //将偏移量设置为false(默认为true)，以忽略偏移量的输出。
request.setPayloads(false); //将有效载荷设置为false(默认为true)，以忽略有效载荷的输出。

Map<String, Integer> filterSettings = new HashMap<>();
filterSettings.put("max_num_terms", 3);
filterSettings.put("min_term_freq", 1);
filterSettings.put("max_term_freq", 10);
filterSettings.put("min_doc_freq", 1);
filterSettings.put("max_doc_freq", 100);
filterSettings.put("min_word_length", 1);
filterSettings.put("max_word_length", 10);

request.setFilterSettings(filterSettings);  //设置过滤器设置，根据tf-idf分数过滤可返回的术语。

Map<String, String> perFieldAnalyzer = new HashMap<>();
perFieldAnalyzer.put("user", "keyword");
request.setPerFieldAnalyzer(perFieldAnalyzer);  //将perFieldAnalyzer设置为指定与该字段不同的分析仪。

request.setRealtime(false); //将实时设置为假(默认值为真)以接近实时地检索术语向量。
request.setRouting("routing"); //设置路由参数
```

**同步执行**

 当以下列方式执行术语请求时，客户端在继续执行代码之前，等待返回术语响应:

```
TermVectorsResponse response =
        client.termvectors(request, RequestOptions.DEFAULT);
```

 同步调用可能会在高级REST客户端中解析REST响应失败、请求超时或类似服务器没有响应的情况下抛出IOException。



 在服务器返回4xx或5xx错误代码的情况下，高级客户端会尝试解析响应主体错误详细信息，然后抛出一个通用的ElasticsearchException，并将原始ResponseException作为抑制异常添加到其中。

**异步执行**

 也可以异步方式执行TermVectorsRequest，以便客户端可以直接返回。用户需要通过将请求和侦听器传递给异步术语向量方法来指定如何处理响应或潜在故障:

```
client.termvectorsAsync(request, RequestOptions.DEFAULT, listener); //要执行的术语向量请求和执行完成时要使用的操作侦听器
```

 异步方法不会阻塞并立即返回。如果执行成功，则使用onResponse方法回调操作侦听器，如果执行失败，则使用onFailure方法回调操作侦听器。失败场景和预期异常与同步执行情况相同。

 术语向量的典型监听器如下所示:

```
listener = new ActionListener<TermVectorsResponse>() {
    @Override
    public void onResponse(TermVectorsResponse termVectorsResponse) {
        //成功的时候调用
    }
    @Override
    public void onFailure(Exception e) {
         //失败的时候调用
    }
};
```

**TermVectorsResponse**

```
String index = response.getIndex(); //文档的索引名称。
String type = response.getType(); //文档的类型名称。
String id = response.getId(); //文档的id。
boolean found = response.getFound(); //指示是否找到文档。
```

**检查术语向量**

 如果术语向量响应包含非空的术语向量列表，则可以使用以下方法获得关于每个术语向量的信息:

```
for (TermVectorsResponse.TermVector tv : response.getTermVectorsList()) {
    String fieldname = tv.getFieldName(); //当前字段的名称
    int docCount = tv.getFieldStatistics().getDocCount(); //当前字段-文档计数的字段统计
    long sumTotalTermFreq =
            tv.getFieldStatistics().getSumTotalTermFreq(); //当前字段的字段统计信息-总术语频率之和
    long sumDocFreq = tv.getFieldStatistics().getSumDocFreq(); //当前字段的字段统计信息-文档频率的总和
    if (tv.getTerms() != null) {当前字段的术语
        List<TermVectorsResponse.TermVector.Term> terms =
                tv.getTerms(); //
        for (TermVectorsResponse.TermVector.Term term : terms) {
            String termStr = term.getTerm(); //术语的名称
            int termFreq = term.getTermFreq(); //术语的术语频率
            int docFreq = term.getDocFreq(); //记录术语的频率
            long totalTermFreq = term.getTotalTermFreq(); //术语的总术语频率
            float score = term.getScore(); //分数
            if (term.getTokens() != null) {
                List<TermVectorsResponse.TermVector.Token> tokens =
                        term.getTokens(); //该术语的令牌
                for (TermVectorsResponse.TermVector.Token token : tokens) {
                    int position = token.getPosition(); //令牌的位置
                    int startOffset = token.getStartOffset(); //令牌的起始偏移量
                    int endOffset = token.getEndOffset(); //令牌的结束偏移量
                    String payload = token.getPayload(); //令牌的有效负载
                }
            }
        }
    }
}
```