Murmur3Partitioner 是默认的分区器，提供了更快的hashing.相比较其他的分区器，极大的提高了性能。Murmur3Partitioner 可以在虚拟节点情况下使用，如果你不使用虚拟节点，你必须要计算tokens。像[Generating tokens](http://docs.datastax.com/en/cassandra/3.0/cassandra/configuration/configGenTokens.html)中描述的一样。

在新集群中使用Murmur3Paritioner;你不能在一个现有的集群中更换分区器，去使用一个不同的分区方式。Murmur3Partitioner 使用MurmurHash方法，这个hashing方法为partition key创建一个64位的hash值。可能的范围值是-2^63 到(2^63)-1.

使用Murmur3Partitioner，可以在一个CQL 查询中使用[token function](http://docs.datastax.com/en/cql/3.3/cql/cql_reference/paging.html) 对结果分页
