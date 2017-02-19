# 数据是如何读的 #

为了满足读，Cassandra必须要从存活的memtable和潜在的多个SSTables中联合查询结果。

Cassandra在读的过程中