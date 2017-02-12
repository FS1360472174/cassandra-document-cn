#cassandra 如何读写#



为了管理和访问cassandra中的数据,非常有必要理解cassandra 如何存储数据的。

hinted handoff 特征为Cassandra的ACID\(原子性，一致性，隔离性，持久性\)增加了一致和【non-conformance】

数据库属性是理解读和写的主要概念。在Cassandra,一致性指的是如何保证数据最新，以及每一行的数据的各个副本都是同步的

的。



开发应用程序的数据存储和查询的客户端的工具和APIs在\[这边\]\(http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html\)



#数据是如何写入的#



Cassandra写的时候分好几个阶段写处理数据，从立即写一个write操作开始，到把数据写到磁盘中

- 写log到commit log

- 写数据到memtable

- 从memtable中flush数据

- 将数据存储在SSTables中的磁盘中



**写Log及memtable存储**



当一个写发生的时候，Cassandra将数据存储在一个内存结构中，叫memtable,并且提供了可配置

的持久化，同时也追加写操作到磁盘上的commit log中。commit log记载了每一个到Cassandra 节点的写入。

这些持久化的写入永久的存活即使某个节点掉电了。memtable是一个数据分区写回的缓存。Cassandra通过key

来查找。memtables顺序写，直到达到配置的限制，然后flushed.



**从memtable中Flushing数据**



为了flush数据，Cassandra以memtable-sorted的顺序将数据写入到磁盘。同时也会在磁盘上创建一个分区索引，

将数据的token map到磁盘的位置。当memtable 内容超过了配置的阈值或者commitlog的空间超过了

commitlog_total_space_in_mb的值，memtable 会被放入到一个队列中，然后flush到磁盘中。这个队列可以通过

cassandra.yaml文件中memtable_heap_space_in_mb,或者memtable_offheap_space_in_mb来配置。如果待flush的

数据超过了memtable_cleanup_threshold，Cassandra会block住写操作。直到下一次flush成功。你可以手动的flush一张表，

使用nodetool flush 或者nodetool drain(flushes memtables 不需要监听跟其他节点的连接)。为了降低commit log

的恢复时间，建议的最佳实践是在重新启动节点之前，flush memtable.如果一个节点停止了工作，将会从节点停止前开始，将commit log

恢复到memtable中。



当数据从memtable中flush到磁盘的一个SSTable中，对应的commit log数据将会被清除。



** 将数据存储到磁盘中的SSTables中**



Memtables 和 SSTables是根据每张表来维护的。而commit log则是表之间共用的。SSTables是不可改变的，当memtable被flushed后，

是不能够重新写入的。因此，一个分区存储着多个SSTable文件。有几个其他的SSTable 结构存在帮助读操作。



对于每一个SSTable，Cassandra 创建了这些结构:



**Data(Data.db)**



  SSTable的数据



**Primary Index(Index.db)**



  行index的指针，指向文件中的位置



**Bloom filter (Filter.db)**



  一种存储在内存中的结构，在访问磁盘中的SSTable之前,检查行数据是否存在memtable中



**Compression Information(CompressionInfo.db)**



  保存未压缩的数据长度，chunk的起点和其他压缩信息。



**Statistics(Statistics.db)**



  SSTable的内容统计数据元数据。



**Digest(Digest.crc32, Digest.adler32, Digest.sha1)**



  保存adler32 checksum的数据文件



**CRC (CRC.db)**



  保存没有被压缩的文件中的chunks的CRC32



**SSTable Index Summary(SUMMARY.db)**



  存储在内存中的的分区索引的一个样例。



**SSTable Table of Contents(TOC.txt)**



  存储SSTable TOC 中所有的组件的列表。



**Secondary Index(SL_.*.db)**



  内置的secondary index。每个SSTable可能存在多个SIs中。



SSTables是存储在磁盘中的文件。SSTable文件的命名从Cassandra 2.2开始后发生变化为了

缩短文件路径。变化发生在安装的时候,数据文件存储在一个数据目录中。对于每一个keyspace,

一个目录的下面一个数据目录存储着一张表。例如,

/data/data/ks1/cf1-5be396077b811e3a3ab9dc4b9ac088d/la-1-big-Data.db 代表着

一个数据文件.ks1 代表着keyspace 名字为了在streaming或者bulk loading数据的时候区分

keyspace。一个十六进制的字符串，5be396077b811e3a3ab9dc4b9ac088d在这个例子中，被加到

table名字中代表着unique的table IDs.



Cassandra为每张表创建了子目录，允许你可以为每个table创建syslink，map到一个物理驱动或者数据

磁盘中。这样可以将非常活跃的表移动到更快的媒介中，比如SSDs,获得更好的性能，同时也将表拆分到各个

挂载的存储设备中，在存储层获得更好的I/O平衡。



