## 基础概念

------

 ElasticSearch是一个基于Apache Lucene的搜索服务器。它由谢伊·巴农(Shay Banon)开发，于2010年出版。它现在由ElasticSearch公司维护。截止到2019年6月19日，它的最新版本是7.1.1。

 ElasticSearch是一个实时分布式开源全文搜索和分析引擎。它可以从RESTful网络服务接口访问，并使用无模式JSON (JavaScript对象符号)文档来存储数据。它建立在Java编程语言之上，使ElasticSearch能够在不同的平台上运行。它使用户能够以非常高的速度浏览非常大量的数据。

**ElasticSearch的特点**

-  ElasticSearch最多可扩展到千兆字节的结构化和非结构化数据。
-  ElasticSearch可以用来替代像MongoDB和RavenDB这样的文档存储。
-  ElasticSearch使用反规范化来提高搜索性能。
-  ElasticSearch(ElasticSearch)是流行的企业搜索引擎之一，目前正被维基百科、卫报、StackOverflow、GitHub等许多大组织使用。
-  ElasticSearch是开源的，在Apache许可版本2.0下可用。

**ElasticSearch关键概念**

-  Node（节点）:它指的是ElasticSearch的单个运行实例。单个物理和虚拟服务器可容纳多个节点，具体取决于其物理资源(如内存、存储和处理能力)的能力。
-  Cluster（集群）:它是一个或多个节点组成的集合。集群为整个数据提供跨节点的索引和搜索功能。
-  Index（索引）:包含一堆有相似结构的文档数据，比如可以有一个客户索引，商品分类索引，订单索引，索引有一个名称。一个index包含很多document，一个index就代表了一类类似的或者相同的Document。比如说建立一个商品索引，里面可能就存放了所有的商品数据。
-  Type/Mapping（类型/映射):它是共享同一索引中一组公共字段的文档集合。例如，索引包含社交网络应用程序的数据，然后可以有特定类型的用户简档数据、另一种类型的消息数据和另一种类型的评论数据。
-  Document(文档):它是以JSON格式定义的特定方式的字段集合。每个文档都属于一个类型，并驻留在一个索引中。每个文档都有一个唯一的标识符，称为UID。
-  Shard(分片):单台机器无法存储大量数据，ElasticSearch可以将一个索引中的数据切分为多个Shard，分布在多台服务器上存储。有了Shard就可以横向扩展，存储更多数据，让搜索和分析等操作分布到多台服务器上去执行，提升吞吐量和性能。
-  Replicas(副本):服务器随时可能故障或宕机，此时Shard可能就会丢失，因此可以为每个Shard创建多个Replica副本。Replica可以在Shard故障时提供备用服务，保证数据不丢失，多个Replica还可以提升搜索操作的吞吐量和性能。

**ElasticSearch的优势**

-  ElasticSearch是在Java上开发的，这使得它在几乎每个平台上都是兼容的。
-  ElasticSearch是实时的，换句话说，一秒钟后添加的文档可以在这个引擎中搜索。
-  ElasticSearch是分布式的，这使得它易于在任何大组织中扩展和集成。
-  通过使用ElasticSearch中的网关概念，创建完整备份很容易。
-  与Apache Solr相比，在ElasticSearch中处理多用户非常容易。
-  ElasticSearch使用JSON对象作为响应，这使得用大量不同的编程语言调用ElasticSearch服务器成为可能。
-  ElasticSearch支持几乎所有文档类型，除了那些不支持文本呈现的。

**ElasticSearch的缺点**

-  ElasticSearch在处理请求和响应数据方面没有提供多语言支持(仅在JSON中可能)，这与Apache Solr不同，Apache Solr中可能有CSV、XML和JSON格式。
-  ElasticSearch也有分裂的问题，但在极少数情况下。

**ElasticSearch与关系数据库系统的比较**

 在ElasticSearch中，索引是类型的集合，正如数据库是关系数据库管理系统中表的集合一样。每个表都是行的集合，就像每个映射都是JSON对象ElasticSearch的集合一样。

| ElasticSearch | RDBMS    |
| :------------ | :------- |
| Index         | Database |
| Shard         | Shard    |
| Mapping       | Table    |
| Field         | Field    |
| JSON Object   | Tuple    |