# 数据是如何维护  #
Cassandra 写入过程中将数据存入到的文件叫做SSTables.SSTables 是不可更改的。Cassandra在写入或者更新时不是去覆盖已有的行，而是写入一个带有新的时间戳版本的数据到新的SSTables红。Cassandra删除操作不是去移除数据，而是将它标记为[墓碑](http://docs.datastax.com/en/glossary/doc/glossary/gloss_tombstone.html)。

随着时间的推移，Cassandra可能会在不同的SSTables中写入一行的多个版本的数据。每个版本都可能有独立的不同的时间戳的列集合。随着SSTables的增加，数据的分布需要收集越来越多的SSTables来返回一个完整的行数据。

为了保证数据库的健康性，Cassandra周期性的合并SSTables,并将老数据废弃掉。这个过程称之为合并压缩。

## 合并压缩 ##
Cassandra 支持不同类型的压缩策略，这个决定了哪些SSTables被选中做compaction，以及压缩的行在新的SSTables中如何排序。每一种策略都有自己的优势，下面的文字解释了每一种Cassandra's compaction 策略。

尽管下面片段的开始都介绍了一个常用的推荐，但是有很多的影响因子是的compaction策略的选择变得很复杂。

### SizeTieredCompactionStrategy(STCS) ###

建议用在写占比多的情况。
当Cassandra 相同大小的SSTables数目达到一个固定的数目(默认是4),STCS 开始压缩。STCS将这些SSTables合并成一个大的SSTable。档