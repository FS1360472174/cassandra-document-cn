# 架构简介
cassandra为跨多个节点、没有单节点失败、大数据量负载设计的。它的架构考虑了系统和硬件可能发生的故障。Cassandra 通过在多个相似的节点构建点对点的分布式系统来解决这个问题，将数据分布在集群中的所有节点上。每个节点定时的和集群中其他节点通过点对点的gossip通信协议交换自身信息。每个节点上通过顺序写commit log 来捕获写事件，从而实现数据的持久化。然后对数据进行索引，写入到内存结构，memtable.相当于写回缓存。每次memtable满了，数据就会被写到磁盘，一个SSTables的文件中。所有的写操作都自动分区、在集群中复制。Cassandra 通过compaction 任务定时合并SSTables,删除掉通过墓碑标记为删除的过失数据。多种repair机制来确保数据能够在集群中保持一致。

> 写回缓存

Cassandra 是一个分区行储存的数据库，其中行通过一个不可少的主键组成表。Cassandra 架构允许授权用户通过CQL在任何数据中心的节点上连接到集群的任何节点，为了易使用，CQL使用了类似SQL的语法。和cassandra进行交互的最基础的工具就是CQL shell,cqlsh.使用cqlsh,你可以创建keyspaces,tables.增删改查表数据。Cassandra3.x需要CQL2.2+支持。如果你喜欢图像化工具,可以使用[DataStax DevCenter](http://docs.datastax.com/en/developer/devcenter/doc/devcenter/features.html)，生产环境中,DataStax 提供一系列的[driver](http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html)。一般情况下，一个应用在集群中有一个keyspace,有很多不同的表组成。

客户端的读或写请求可以发送到集群中的任意节点上。当一个客户端通过一个请求连接到某个节点，这个节点就作为这次请求的协作者(coordinator)。协作者相当于存储这次请求数据的cassandra节点和客户端的中间代理。协作者决定应该向ring环中的哪些节点发出请求。

注
> Cassandra 默认分区，会根据partition key将数据分布在各个节点上，每个节点上都存储着这样的一份映射关系，所以coordinator知道向ring环中的哪些集群节点发出请求。


**主要结构**
* 节点(Node)
  存储数据的地方，是Cassandra的基本构成。
* 数据中心
* 相关节点的集合。一个数据中心可以是一个物理的数据中心，也可以是虚拟的数据中心。不同的负载应该使用不同的数据中心，物理或者虚拟的数据中心。复制可以在数据中心之间进行。使用不同的数据中心可以阻止节点被其他的负载所影响，另外可以使得请求靠近DB,获取低延迟。取决于复制设置，数据可以被写到多个数据中心。数据中心不要横跨物理区域。
* 集群
  一个集群包含一个或多个数据中心，集群可以跨越物理区域
* 提交日志(commit log)
  所有的数据为了持久化储存，都会先写到commit log.当所有的数据都从内存中写到SSTables后，commit log 可以被存档，删除，或者回收
* SSTable
SSTable(sorted string table)是一个不可改变的数据文件，cassandra会间断性的写数据到memtable.SSTables 是只可追加，顺序存储在磁盘，不同的cassandra table单独维护。
* CQL Table




