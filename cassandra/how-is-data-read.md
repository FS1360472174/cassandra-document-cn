# 数据是如何读的 #

为了满足读，Cassandra必须要从存活的memtable和潜在的多个SSTables中联合查询结果。

Cassandra在读的过程中为了找到数据存储在何处，需要在好几个阶段处理数据。从memtables开始，到SSTables结束。

- 检查 memtable
- 如果enabled了,检查row cache
- 检查Bloom filter
- 如果enable了,检查partition key 缓存
- 如果在partition key缓存中找到了partition key,直接去compression offset mao中，如果没有，检查 partition summary
- 根据compression offset map找到数据位置
- 从磁盘的SSTable中取出数据

读请求流程图
![](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/dml_caching-reads_12.png)

行缓存和键缓存请求流程图
![](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/ops_how-cache-works.png)


## MemTable ##
如果memtable有目标分区数据，这个数据会被读出来并且和从SSTables中读出来的数据进行合并。SSTable的数据访问如下面所示的步骤。

## Row Cache ##
通常意义上，对于任何数据库，当读的大部分数据都在内存中读取的时候是非常快的。操作系统的页缓存是最有利提升性能的，经过行缓存对于读占比高的应用有一定性能提升，如读操作占95%。Row Cache对于写占比高的系统是禁用的。如果开启了row cache.就会将一部分存储在磁盘的SSTables数据存储在了内存中。在Cassandra2.2+，它们被存储在堆外内存，使用全新的实现避免造成垃圾回收对JVM造成压力。存在在row cache的子集数据可以在特定的一段时间内配置一定大小的内存。row cache使用LRU(least-recently-userd)进行回收在申请内存，当cache满的时候。

row cache的大小是可以配置的，值是可以存多少行。配置缓存的行数是一个非常有用的功能，使得类似'最后的10条'查询可以很快。如果row cache 开启了。目标的数据就会从row cache中读取，潜在的节省了对磁盘数据的两次检索。存储在row cache中的数据是SSTables中频繁被访问的数据。存储到row cache中后，数据就可以被后续的查询访问。row cache不是写更新。如果写某行了，这行的缓存就会失效，并且不会被继续缓存，直到这行被读到。类似的，如果一个partition更新了，整个partition的cache都会被移除，但目标的数据在row cache中找不到，就会去检查Bloom filter。

## Bloom Filter ##
