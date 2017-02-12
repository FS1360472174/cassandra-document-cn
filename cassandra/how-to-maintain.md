# 数据是如何维护  #
Cassandra 写入过程中将数据存入到的文件叫做SSTables.SSTables 是不可更改的。Cassandra在写入或者更新时不是去覆盖已有的行，而是写入一个带有新的时间戳版本的数据到新的SSTables红。Cassandra删除操作不是去移除数据，而是将它标记为[墓碑](http://docs.datastax.com/en/glossary/doc/glossary/gloss_tombstone.html)。

随着时间的推移，Cassandra可能会在不同的SSTables中写入一行的多个版本的数据。每个版本都可能有独立的不同的时间戳的列集合。随着SSTables的增加，数据的分布需要收集越来越多的SSTables来返回一个完整的行数据。

为了保证数据库的健康性，Cassandra周期性的合并SSTables,并将老数据废弃掉。这个过程称之为压缩。