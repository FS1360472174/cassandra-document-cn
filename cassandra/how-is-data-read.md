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

首先，Cassandra检查Bloom filter去发现哪个SSTables中有可能有请求的分区数据。Bloom filter是存储在堆外内存。每个SSTable都有一个关联的Bloom filter。一个Bloom filter可以建立一个SSTable没有包含的特定的分区数据。同样也可以找到分区数据存在SSTable中的可能性。它可以加速查找partition key的查找过程。然而，因为Bloom filter是一个概率函数，所以可能会得到错误的结果，并不是所有的SSTables都可以被Bloom filter识别出是否有数据。如果Bloom filter不能够查找到SSTable，Cassandra会检查partition key cache.

Bloom filter 大小增长很适宜，每10亿数据1~2GB。在极端情况下，可以一个分区一行。都可以很轻松的将数十亿的entries存储在单个机器上。Bloom filter是可以调节的，如果你愿意用内存来换取性能。

## Partition Key Cache ##

partition key 缓存如果开启了，将partition index存储在堆外内存。key cache使用一小块可配置大小的内存。在读的过程中，每个"hit"保存一个检索。如果在key cache中找到了partition key。就直接到compression offset map中招对应的块。partition key cache热启动后工作的更好，相比较冷启动，有很大的性能提升。如果一个节点上的内存非常受限制，可能的话，需要限制保存在key cache中的partition key数目。如果一个在key cache中没有找到partition key。就会去partition summary中去找。

partition key cache 大小是可以配置的，意义就是存储在key cache中的partition keys数目。

## Partition Summary ##

partition summary 是存储在堆外内存的结构，存储一些partition index的样本。如果一个partition index包含所有的partition keys。鉴于一个partition summary从每X个keys中取样，然后将每X个key map到index 文件中。例如，如果一个partition summary设置了20keys进行取样。它就会存储SSTable file开始的一个key,20th 个key，以此类推。尽管并不知道partition key的具体位置，partition summary可以缩短找到partition 数据位置。当找到了partition key值可能的范围后，就会去找partition index。

通过配置取样频率，你可以用内存来换取性能，当partition summary包含的数据越多，使用的内存越多。可以通过表定义的[index interval](http://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlCreateTable.html#tabProp)属性来改变样本频率。固定大小的内存可以通过[index_summary_capacity_in_mb](http://docs.datastax.com/en/cassandra/3.0/cassandra/configuration/configCassandra_yaml.html#configCassandra_yaml__index_summary_capacity_in_mb)属性来设置，默认是堆大小的5%。

## Partition Index ##

partition index驻扎在磁盘中，索引所有partition keys和偏移量的映射。如果partition summary 已经查到partition keys的范围，现在的检索就是根据这个范围值来检索目标partition key。需要进行单次检索和顺序读。根据找到的信息。然后去compression offset map中去找磁盘中有这个数据的块。如果partition index必须要被检索，则需要检索两次磁盘去找到目标数据。

## Compression offset map ##

compression offset map存储磁盘数据准确位置的指针。存储在堆外内存，可以被partition key cache或者partition index访问。一旦compression offset map识别出来磁盘中的数据位置，就会从正确的SStable(s)中取出数据。查询就会收到结果集。

**注:** 在一个分区里，所有的行查询代价并不是一致的。在一个分区的开始（第一行，根据clustering key定义）想对来说代价更小，应为没有必要执行partition-level的index。

compression offset map 每TB增长量为1~3GB。压缩的数据越多，压缩块的数量越多，压缩偏移表就会越大。Compression 默认是开启的，即使压缩过程中会消耗CPU资源。开启compression使得页缓存更加的高效,通常来说是值得的。

**注:**
> cassandra读取的数据是memtable中的数据和SStables中数据的合并结果。读取SSTables中的数据就是查找到具体的哪些的SSTables以及数据在这些SSTables中的偏移量(SSTables是按主键排序后的数据块)。首先如果row cache enable了话，会检测缓存。缓存命中直接返回数据。没有查找Bloom filter，查找可能的SSTable。然后有一层Partition key cache，找partition key的位置。如果有根据找到的partition去压缩偏移量映射表找具体的数据块。如果缓存没有，则要经过Partition summary,Partition index去找partition key。然后经过压缩偏移量映射表找具体的数据块。