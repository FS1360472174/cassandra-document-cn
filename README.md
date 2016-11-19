# cassandra官方文档(3.x)


### cassandra 概况
Apache Cassandra 是一个支持高扩展的开源NoSQL数据库。Cassandra非常适合在云、多数据中心存储结构化、非结构化的数据。Cassandra 支持高可用性，线性扩展，非常简单的在多个server上进行操作，没有单点故障。强大的动态数据模型支持提供了灵活性和快速响应。

### cassandra 工作原理
cassandra 可扩展的架构意味着它有能力处理PB级数据，每秒上千的并发操作

| ------------- |:-------------:|
| 分片行存储DB | Cassandra 架构允许授权用户通过CQL在任何数据中心的节点上连接到集群的任何节点
为了容易使用，CQL使用了类似SQL的语法。和cassandra进行交互的最基础的工具就是CQL shell,cqlsh.使用cqlsh,你可以创建keyspaces,tables.增删改查表数据。Cassandra3.x需要CQL2.2+支持。如果你喜欢图像化工具,可以使用[DataStax DevCenter](http://docs.datastax.com/en/developer/devcenter/doc/devcenter/features.html),生产环境，DataStax 提供一系列的[driver](http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html)|
| 数据自动分布 | Cassandra 为集群内的节点提供跨节点的自动数据分布。数据在集群各个节点的分布是是透明的，不需要开发者或者DBA做任何事情 |
| 内置且可配置的复制 | Cassandra 也提供内置的且可配置的复制，为集群节点环的数据提供跨节点的数据冗余备份。这意味着集群中任何一个节点down了，集群中的其他节点还有down节点的数据备份。复制可以进行配置，跨数据中心，扩地区|
| cassandra支持线性扩展 | Cassandra支持线性扩展，意味着可以通过简单的增加新节点来提高集群能力。例如,如果两个节点能够支持每秒100,000个事务，4个节点就能够支持每秒200,000.8个节点就能支持400,000|

![1](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/intro_cassandra.png)

### cassandra 与关系型数据库的区别

### NoSQL 介绍

### CQL 介绍

### 数据迁移

###cassandra 工具
