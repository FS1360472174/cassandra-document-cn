# 数据是如何删除的 #

Cassandra删除数据的过程是为了提高性能，以及和Cassandra一些内置的属性是实现数据的分布，和故障容忍。

Cassandra将一个删除视为一个插入或者upsert。[DELETE](http://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlDelete.html)命令中添加的数据会被加上一个删除标记，叫做[墓碑](http://docs.datastax.com/en/glossary/doc/glossary/gloss_tombstone.html)。墓碑标记走的也是Cassandra写过程，会被写到一个或多个节点的SSTables。墓碑最主要的不同点在于:它有一个内置的过期日期/时间。在它过期的时候(更多细节在下面),墓碑会作为Cassandra'compaction的一部分过程删除掉。

## 在分布式系统中做删除操作 ##
在一个多节点的集群中，Cassandra可以在2个或者更多的节点上存储一份数据的多个副本。这样可以防止数据丢失，但是会使得删除过程变得复杂。如果一个节点接收到删除，在它存在本地，节点将这个特定记录标记为墓碑，然后尝试将墓碑传递给其他包含这个记录的节点。但是如果某个存有该数据的节点在这个时间点没有应答，不能够马上收到墓碑，因此它仍会有这些记录的删除前的版本。如果在这个节点恢复过来之前，集群中其他被标记的墓碑数据都已经被删除了，Cassandra会将恢复过来的这个节点上的这个记录作为新的数据，并且将它传送到集群的其他节点。这种类型的删除大但是仍存活下来的记录叫做[zombie](http://docs.datastax.com/en/glossary/doc/glossary/gloss_zombie.html)

为了阻止zombies重新出现，Cassandra给每个墓碑一个grace时间段。grace时间段的目的就是给没有应答的节点时间去回复和正常处理墓碑。如果一个客户端在grace period多墓碑数据写入了一个新的更新。Cassandra会覆盖掉墓碑。如果一个客户端在grace时间段，读墓碑记录，Cassandra会无视墓碑，如果可能的话从其他副本中读取记录。

当一个无应答的节点恢复过来，Cassandra使用[hinted handoff](http://docs.datastax.com/en/cassandra/3.0/cassandra/operations/opsRepairNodesHintedHandoff.html)来恢复节点down期间错过的数据的更改。Cassandra不会去为墓碑数据恢复更改在它grace period期间。但是如果在grace period结束后，节点还没有恢复，Cassandra会错过删除。

当墓碑的grace period结束后，Cassandra会在compaction阶段删除墓碑。

一个墓碑的grace period是通过设置`gc_grace_seconds`属性来设置的。默认值是86400秒(10天)。每个表这个属性可以单独设置。

## 更多关于Cassandra删除 ##

细节：

- 墓碑数据过期的日期/时间是它创建的日期/时间加上表属性`gc_grace_seconds`值。

- Cassandra也支持[批量插入和更新](http://docs.datastax.com/en/cql/3.3/cql/cql_using/useBatchTOC.html)。这个过程通过会带来恢复一个记录的插入的风险，当这条记录已经被集群的其他节点移除掉了。Cassandra不会在grace period期间，为一个墓碑数据恢复批量更改。

- 在一个单节点的集群中，你可以设置`gc_grace_seconds`的值为0

- 为了完全阻止zombies记录的再出现，在节点恢复过来后、以及每个表的`gc_grace_seconds`，跑[nodetool repair](http://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsRepair.html)

- 如果一个表的所有记录在创建的时候设置了TTL，以及所有的都允许过期，且不能手动删除，就没有必要为这张表定期跑[nodetool repair](http://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsRepair.html)

- 如果使用了SizeTieredCompactionStrategy 或者 DateTieredCompactionStrateg，你可以通过[手动开启compaction](http://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsCompact.html)立即删除掉墓碑。
**小心：**
如果你强制compaction,Cassandra可能从所有的数据中创建一个非常大的SSTable。Cassandra在很长的时间段中不会再触发另外一个compaction。在强制compaction期间创建的SSTable数据可能会变得非常过时在一个长的没有compaction阶段。

- Cassandra 允许你为整张表设置一个默认的`_time_to_live`属性。列和行都被标记为一个TTLs;但是如果一条记录超过了表级别的TTL，Cassanda会立即将它删除，而没有标记墓碑或者compaction过程。

 - Cassandra支持通过[DROP KEYSPACE](http://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlDropKeyspace.html) 和 [DROP TABLE](http://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlDropTable.html)立即执行删除。