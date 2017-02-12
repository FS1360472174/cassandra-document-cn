# 数据是如何维护  #
Cassandra 写入过程中将数据存入到的文件叫做SSTables.SSTables 是不可更改的。Cassandra在写入或者更新时不是去覆盖已有的行，而是写入一个带有新的时间戳版本的数据到新的SSTables中。Cassandra删除操作不是去移除数据，而是将它标记为[墓碑](http://docs.datastax.com/en/glossary/doc/glossary/gloss_tombstone.html)。

随着时间的推移，Cassandra可能会在不同的SSTables中写入一行的多个版本的数据。每个版本都可能有独立的不同的时间戳的列集合。随着SSTables的增加，数据的分布需要收集越来越多的SSTables来返回一个完整的行数据。

为了保证数据库的健康性，Cassandra周期性的合并SSTables,并将老数据废弃掉。这个过程称之为合并压缩。

## 合并压缩 ##
Cassandra 支持不同类型的压缩策略，这个决定了哪些SSTables被选中做compaction，以及压缩的行在新的SSTables中如何排序。每一种策略都有自己的优势，下面的文字解释了每一种Cassandra's compaction 策略。

尽管下面片段的开始都介绍了一个常用的推荐，但是有很多的影响因子是的compaction策略的选择变得很复杂。

### SizeTieredCompactionStrategy(STCS) ###

建议用在写占比高的情况。
当Cassandra 相同大小的SSTables数目达到一个固定的数目(默认是4),STCS 开始压缩。STCS将这些SSTables合并成一个大的SSTable。当这些大的SSTable数量增加，STCS将它们合并成更大的SSTables。在给定的时间范围内，SSTables大小变化如下图所示
![](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/dml-how-maintained-STCS-1.png)

STCS 在写占比高的情况下压缩效果比较好，它将读变得慢了，因为根据大小来合并的过程不会将数据按行进行分组，这样使得某个特定行的多个版本更有可能分布在多个SSTables中。而且，STCS不会预期的回收删除的数据，因为触发压缩的是SSTable的大小，SSTables可能增长的足够快去合并和回收老数据。随着最大的SSTables 大小在增加，disk需要空间同时去存储老的SSTables和新的SSTables。在STCS压缩的过程中可能回超过一个节点上典型大小的磁盘大小。

- **优势:** 写占比高的情况压缩很好
- **劣势:** 可能将过期的数据保存的很久，随着时间推移，需要的内存大小随之增加。

### LeveledCompactionStrategy(LCS) ###
建议用在读占比高的情况。

LCS减少了STCS多操作的一些问题。这种策略是通过一系列层级来工作的。首先，memtables中数据被flush到SSTables是第一层(L0)。LCS 压缩将这些第一层的SSTables合并成更大的SSTables L1。

Leveled compaction —— 添加SSTables
![](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/dml-how-maintained-leveled-1.png)

高于L1层的SSTables会被合并到一个大小大于等于sstable_size_in_md（默认值:160MB）的SSTables中。如果一个L1层的SSTable存储的一部分数据大于L2，LCS会将L2层的SSTable移动到一个更高的等级。

许多插入操作之后的Leveled compaction
![](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/dml-how-maintained-leveled-2.png)

在每个高于L0层的等级中，LCS创建相同大小的SSTables。每一层大小是上一层的10倍，因此L1层的SSTable是L0层的10倍，L2层是L0层100倍。如果L1层的压缩结果超过了10倍，超出的SSTables就会被移到L2层。

LCS压缩过程确保了从L1层开始的SSTables不会有重复的数据。对于许多读，这个过程是的Cassandra能够从一个或者二个SSTables中获取到全部的数据。实际上，90%的都能够满足从一个SSTable中获取。因为LCS不去compact L0 tables。资源敏感型的读涉及到多个L0 SSTables的情况还是会发生。

高于L0层，LCS需要更少的磁盘空间去做压缩——一般是SSTable大小的10倍。过时的数据回收的更频繁，因此删除的数据占用磁盘空间更少的比例。然而，LCS 压缩操作使用了更多的I/O操作，增加了节点的I/O负担。对于写占比高的情况，使用这种策略的获取的报酬不值得付出的I/O操作对性能造成损失的代价。在大多数情况下，配置成LCS的表的测试表明写和压缩I/O饱和了。

**注：**Cassandra绕过了compaction操作，当使用LCS策略bootstrapping一个新节点到集群中。初始的数据被直接搬到正确的层级因为这儿没有已有的数据，因此每一层没有分片重复。获取更多的信息，查看[](http://www.datastax.com/dev/blog/bootstrapping-performance-improvements-for-leveled-compaction)

**优势：** 磁盘空间的要求容易预测。读操作的延迟更容易预测。过时的数据回收的更及时。

**劣势:** 更高的I/O使用影响操作延迟。

### TimeWindowCompactionStrategy(TWCS) ###
建议用在时间序列且设置了TTL的情况。

TWCS有点类似于简单设置的DTCS。TWCS通过使用一系列的时间窗口将SSTables进行分组。在compaction阶段，TWCS在最新的时间窗口内使用STCS去压缩SSTables。在一个时间窗口的结束，TWCS将掉落在这个时间窗口的所有的SSTables压缩层一个单独的SSTable，在SSTable maximum timestamp基础上。一旦一个时间窗口的主要压缩完成了，这部分数据就不会再有进一步的压缩了。这个过程结束之后SSTable开始写入下一个时间窗口。
![](http://docs.datastax.com/en/cassandra/3.0/cassandra//images/dml-how-maintained-TWCS-1.png)

如上图所示，从上午10点到上午11点，memtables flush到100MB的SSTables中。使用STCS策略将这些SSTables压缩到一个更大的SSTables中。在上午11点的时候，这些SSTables被合并到一个单独的SSTable,而且不会被TWCS再进行压缩了。在中午12点，上午11点到中午12点创建的新的SSTables被STCS进行压缩，在这个时间窗口结束的时候，TWCS压缩开始。注意在每个TWCS时间窗口包含不同大小的数据。

**注:** 可以在[这里](https://academy.datastax.com/courses/ds210-datastax-enterprise-operations-apache-cassandra/time-windowed-compaction)看动画解释。

TWCS配置有两个主要的属性设置

- **compaction_window_unit:** 时间单位，用来定义窗口大小(milliseconds,seconds,hours等等)
- **compaction_window_size:** 每个窗口有多少单元(1,2,3等等)

上面配置的一个例子：compaction_window_unit = 'minutes',compaction_window_size = 60

**优势:**用作时间序列数据，为表中所有数据使用默认的TTL。比DTCS配置更简单。

**劣势:** 不适用于时间乱序的数据，因为SSTables不会继续做压缩，存储会没有边界的增长，所以也不适用于没有设置TTL的数据。相比较DTCS，需要更少的调优配置。

### DateTieredCompactionStrategy(DTCS) ###
Cassandra 3.0.8/3.8 中弃用了。

DTCS类似于STCS。但是STCS压缩事基于SSTable 大小，而DTCS是基于SSTable年纪(在一个SSTable中，每一列都标记着一个写入的时间戳。)对于一个SSTable的年纪，DTCS使用SSTable中任意列中的oldest(最小的)时间戳。

配置DTCS时间戳

### 哪一种压缩策略最好 ###

为了实现最好的压缩策略:
1. Review 你应用的需求
2. 配置表使用最合适的策略
3. 测试压缩策略

下面的问题基于有Cassandra 开发者和使用者的使用经验以及上述描述的策略。

**你的表处理的是时间序列的数据吗**

如果是的，你最好的选择是TWCS 或者DTCS。想要看更多的细节，参考上面的描述。

如果你的表不是局限于时间序列的数据，选择就变得更加复杂了。下面的问题可能会给你其他的考虑去做选择。

**你的表处理读比写多，或者写多于读吗**
LCS 是一个好的选择，如果表处理的读是写的两倍或者更多——尤其是随机读。如果读的比重和写的比重类似。LCS对性能的影响可能不值得获取的好处。要意识到LCS可能会被一大批写很快覆盖掉。

**表中的数据是否经常改变**
LCS的一个优势在于它将相关的数据都保持在小范围的SSTables中，如果你的数据是不可更改的或者不是经常做upserts.STCS可以实现相同类型的分组而不用付出LCS在性能上影响。

**是否需要预测读和写等级**
LCS 保证SSTables在一个可预测的大小和数量中。例如，如果你的表读/写比例比较小，对于读，期望获得一致的服务层面的一致性，或许值得付出写性能的牺牲去确保读速率和延迟在一个可预测的等级。而且可以通过水平扩展（添加更多的节点）来克服掉写入牺牲。

**表会通过一个batch操作插入数据吗**
batch 写或batch读，STCS表现的都比LCS好。batch过程造成很少或没有碎片，因此LCS的好处实现不了，batch操作可以通过LCS配置来覆盖。

**系统有受限的磁盘空间吗**
LCS处理磁盘空间比STCS更高高效：除了数据本身占用的空间，需要大约10%的预留空间。STCS和DTCS需要更多，在某些情况下，差不多需要50%。

**系统是否到达I/O限制**
LCS相比较DTCS 或者STCS，对I/O更加的敏感。换成LCS，可能需要引入更多的I/O来实现这种策略的优势。






