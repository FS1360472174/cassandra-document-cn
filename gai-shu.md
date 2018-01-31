# cassandra官方文档(3.x)


### cassandra 概况
Apache Cassandra 是一个支持高扩展的开源NoSQL数据库。Cassandra非常适合在云、多数据中心存储结构化、非结构化的数据。Cassandra 支持高可用性，线性扩展，非常简单的在多个server上进行操作，支持单点故障。强大的动态数据模型支持提供了灵活性和快速响应。

### cassandra 工作原理
cassandra 可扩展的架构意味着它有能力处理PB级数据，每秒上千的并发操作

| 特点 | 详解 | 
| ------------- |-------------|
| 分区行存储DB | Cassandra 架构允许授权用户通过CQL在任何数据中心的节点上连接到集群的任何节点，为了易使用，CQL使用了类似SQL的语法。和cassandra进行交互的最基础的工具就是CQL shell,cqlsh.使用cqlsh,你可以创建keyspaces,tables.增删改查表数据。Cassandra3.x需要CQL2.2+支持。如果你喜欢图像化工具,可以使用[DataStax DevCenter](http://docs.datastax.com/en/developer/devcenter/doc/devcenter/features.html)，生产环境中,DataStax 提供一系列的[driver](http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html)|
| 数据自动分布 | Cassandra 为集群内的节点提供跨节点的自动数据分布。数据在集群各个节点的分布是是透明的，不需要开发者或者DBA做任何事情 |
| 内置且可配置的复制 | Cassandra 也提供内置的且可配置的复制，为集群节点环的数据提供跨节点的数据冗余备份。这意味着集群中任何一个节点down了，集群中的其他节点还有down节点的数据备份。复制可以进行配置，从而跨数据中心，扩地区分布|
| cassandra支持线性扩展 | Cassandra支持线性扩展，意味着可以通过简单的增加新节点来提高集群能力。例如,如果两个节点能够支持每秒100,000个事务，4个节点就能够支持每秒200,000.8个节点就能支持400,000|
![1](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/intro_cassandra.png)

**注:**

> 这个地方的线性扩展稍微有误导嫌疑，真实环境中的线性扩展不可能如上这种100%线性扩展，增加了节点，节点之间的通信时间会增加，即增加了节点后，单个节点的处理能力会有所下降。笔者之前的测试显示，差不多是70%.这个可能会和具体的业务场景有关系。



### cassandra 与关系型数据库的区别
Cassandra 是基于peer to peer 通信的分布式数据库。最好的实践是1个查询对应一张表。所以数据是非范式的。正因为如此，跨table的JOIN查询不存在，尽管可以在application中去join。

### NoSQL 介绍
NoSQL 通常意义上是指“不仅仅是SQ",意味着DB可以使用不同于关系型数据库的存储数据方法。目前有很多不同类型的NoSQL数据库，因此即使是直接比较常用类型的NoSQL也没什么用。今天的DBA必须要更加全能，他们必须要知道如何运维关系型数据库和NoSQL数据库。
### CQL 介绍
[Cassandra Query Language(CQL)](http://docs.datastax.com/en/cql/3.3/cql/cqlIntro.html) 是Cassandra 数据库管理系统的主要接口，CQL的用法和SQL类似。CQL和SQL有着相同的表结构，行列式。CQL与SQL主要的不同是CQL不支持join和子查询.相反，Cassandra 强调数据的反范式。

CQL是与Cassandra交互的首选。相比较老的Casssandra APIs,CQL性能和简单易用是大的优势

[CQL 文档](http://docs.datastax.com/en/cql/3.3/index.html)
包括了数据模型,例子,和命令参考

#### 如何和Cassandra交互
cassandra进行交互的最基础的工具就是CQL shell,cqlsh.使用cqlsh,你可以创建keyspaces,tables.增删改查表数据。Cassandra3.x需要CQL2.2+支持。如果你喜欢图像化工具,可以使用DataStax DevCenter，生产环境,DataStax 提供一系列的driver
### 数据迁移
数据可以通过CQL INSERT命令，COPY命令和CSV文件方式，或者[sstableloader](http://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsBulkloader.html) 方式。但是实际上，你需要考虑你的客户端应用程序将如何查询表数据，需要首先定义数据模型。关系型数据库和NoSQL之间的范式改变，意味着直接将关系型数据库数据搬到Cassandra注定会失败。

**注:**

> Cassandra 数据库不同于关系型数据库，它必须首先弄清楚业务的查询场景，才能去设计数据模型。



###cassandra 工具
Cassandra 自动安装了[nodetool](http://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsNodetool.html)
工具，一个非常有用的命令行管理工具。另外一个默认的安装工具是[cassandra-stress](http://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsCStress.html)
可以用来做基本的数据库负载性能测试

**注:**

> nodetool 可以用来查询节点状态，节点配置信息等等，是个非常方便，必不可少的cassandra管理工具。

