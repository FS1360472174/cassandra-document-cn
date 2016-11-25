# 架构简介
cassandra是为跨多个节点、没有单节点失败、大数据量负载系统设计的。它的架构考虑了系统和硬件可能发生的故障。Cassandra 通过在多个相似的节点构建点对点的分布式通信系统来解决这个问题，将数据分布在集群中的所有节点上。每个节点定时的和集群中其他节点通过点对点的gossip通信协议交换自身信息。每个节点上通过顺序写commit log 来捕获写事件，从而实现数据的持久化。然后对数据进行索引，写入到内存结构，memtable，相当于写回缓存。每次memtable满了，数据就会被写到磁盘上一个SSTable的文件中。所有的写操作都自动分区、在集群中复制。Cassandra 通过compaction 任务定时合并SSTables,删除掉通过墓碑标记为删除的过时数据。多种repair机制来确保数据能够在集群中保持一致。

> 写回缓存

Cassandra 是一个分区行储存的数据库，其中行通过一个不可少的主键唯一标识，多行组成表。Cassandra 架构允许授权用户通过CQL在任何数据中心的节点上连接到集群的任何节点，为了易使用，CQL使用了类似SQL的语法。和cassandra进行交互的最基本的工具就是CQL shell,即cqlsh.使用cqlsh,你可以创建keyspaces,tables.增删改查表数据。Cassandra3.x需要CQL2.2+支持。如果你喜欢图像化工具,可以使用[DataStax DevCenter](http://docs.datastax.com/en/developer/devcenter/doc/devcenter/features.html)，生产环境中,DataStax 提供一系列的[driver](http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html)。一般情况下，一个应用在集群中有一个keyspace,有很多不同的表组成。

客户端的读或写请求可以发送到集群中的任意节点上。当一个客户端通过一个请求连接到某个节点，这个节点就作为这次请求的协作者(coordinator)。协作者相当于存储这次请求数据的cassandra节点和客户端的中间代理。协作者决定应该向ring环中的哪些节点发出请求。

注
> Cassandra 默认分区，会根据partition key将数据分布在各个节点上，每个节点上都存储着这样的一份映射关系，所以coordinator知道向ring环中的哪些集群节点发出请求。


**主要结构**
* 节点(Node)

  存储数据的地方，是Cassandra的基本构成。
* 数据中心

  相关节点的集合。一个数据中心可以是一个物理的数据中心，也可以是虚拟的数据中心。不同的负载应该使用不同的数据中心，物理或者虚拟的数据中心。复制可以在数据中心之间进行。使用不同的数据中心可以阻止节点被其他的负载所影响，另外可以使得请求靠近DB,获取低延迟。取决于复制设置，数据可以被写到多个数据中心。数据中心不要横跨物理区域。
* 集群

  一个集群包含一个或多个数据中心，集群可以跨越物理区域
* 提交日志(commit log)
  
  所有的数据为了持久化储存，都会先写到commit log.当所有的数据都从内存中写到SSTables后，commit log 可以被存档，删除，或者回收
* SSTable

  SSTable(sorted string table)是一个不可改变的数据文件，cassandra会间断性的写数据到memtable.SSTables 是只可追加，顺序存储在磁盘，不同的cassandra table数据单独维护在不同的sstable文件中。
* CQL Table
* 
顺序列的集合，可以通过行来获取。一个table 有列和一个主键组成。

**Cassandra 主要构成**

* Gossip

  一个peer-to-peer的通信协议，为了发现和分享集群中的节点位置和状态信息。Gossip 信息持久化的存储在每个节点的本地，当重启一个节点，可以马上获得。

* Partitioner

  分区器决定了哪个节点将会存储数据的第一份副本，以及如何在集群的其他节点上分布其他副本。数据的每一行有一个唯一标示primary key,有可能和partition key一样。但是也有可能还有clustering 列。分区器是一个hash算法，token值是从行的primary key hash得到。分区器使用这个token值来决定哪些节点存储该行的副本。Murmur3Partitioner 是Cassandra新版本的默认分区策略，也是几乎所有情况下的最好分区策略选择。
  
* 复制因子

  所有的副本跨越整个集群，复制因子为1意味着每一行只有一份副本，存储在一个节点上。复制因子为2，意味着每一行有两份副本，每一份副本在不同的节点上。所有的副本都是一样重要。没有primary或者master副本。可以为每个datacenter分别定义复制因子。通常复制因子的数量应该大于1，但是不要超过集群中的节点数量。
  
* 副本放置策略

  Cassandra将副本存储在多个节点，确保数据的可靠性和故障容忍性。复制策略决定了副本放置在哪些节点上。数据的第一个副本就是第一份复制，没有什么不一样。对于大部分的部署，强烈推荐NetworkTopologyStrategy，因为未来需要扩展的话，它可以很方便的扩展到多个数据中心。
  
* 探测(Snitch)

  Snitch 定义了数据中心和机架中的一组机器，副本放置策略拿来放置副本的机器。
  
  当你创建一个集群的时候你必须配置一个snitch，所有的snitches都使用动态探测层，可以监测性能，为读选择最好的副本。默认是开启的，也是适合大部分的部署的。可以在cassandra.yaml配置文件中为每个节点配置动态的snitch阈值
  
  默认的SimpleSnitch 不能识别数据中心和机架信息。在公有云上可以为单数据中心或者单地区的部署配置这样的策略。生产环境推荐使用GossipingPropertyFileSnitch，定义了一个节点的数据中心和机架信息。使用gossip将信息传递给其他节点。
  
* cassandra.yaml配置文件

  集群初始化属性设置的主要配置文件，table缓存参数，调节属性、资源利用、超时设置、客户端连接、备份、安全。
  
  默认情况下，一个节点存储数据的目录也是在cassandra.yaml文件中配置的。
  
  在生产环境中，你可以将commitlog-directory和data_file_directories放在不同的磁盘中。
  
* 系统keyspace表属性

  你可以使用客户端，比如CQL,编程来设置基于每个keyspace或者每个表的存储配置属性
  






