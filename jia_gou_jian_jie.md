# 架构简介
cassandra为跨多个节点、没有单节点失败、大数据量负载设计的。它的架构考虑了系统和硬件可能发生的故障。Cassandra 通过在多个相似的节点构建点对点的分布式系统来解决这个问题，将数据分布在集群中的所有节点上。每个节点定时的和集群中其他节点通过点对点的gossip通信协议交换自身信息。每个节点上通过顺序写commit log 来捕获写事件，从而实现数据的持久化。然后对数据进行索引，写入到内存结构，memtable.相当于写回缓存。每次memtable满了，数据就会被写到磁盘，一个SSTables的文件中。所有的写操作都自动分区、在集群中复制。Cassandra 通过compaction 任务定时合并SSTables,删除掉通过墓碑标记为删除的过失数据。多种repair机制来确保数据能够在集群中保持一致。

> 写回缓存

Cassandra 是一个分区行储存的数据库，其中行通过一个不可少的主键组成表。Cassandra 架构允许授权用户通过CQL在任何数据中心的节点上连接到集群的任何节点，为了易使用，CQL使用了类似SQL的语法。和cassandra进行交互的最基础的工具就是CQL shell,cqlsh.使用cqlsh,你可以创建keyspaces,tables.增删改查表数据。Cassandra3.x需要CQL2.2+支持。如果你喜欢图像化工具,可以使用[DataStax DevCenter](http://docs.datastax.com/en/developer/devcenter/doc/devcenter/features.html)，生产环境中,DataStax 提供一系列的[driver](http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html)。一般情况下，一个应用在集群中有一个keyspace,有很多不同的表组成

