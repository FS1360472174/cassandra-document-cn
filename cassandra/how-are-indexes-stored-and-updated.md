# indexes是如何存储和更新的 #

Secondary indexes是被用来过滤非primary key列的表查询。例如，一个表存储cyclist names 和ages。使用cyclist的last name作为主键，可能会有一个age字段的secondary index，使得能够允许根据年龄来查询。查询匹配非主键的列是反范式的，因为这样的查询经常会导致表连续数据切片产生。

![](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/indexing_diagram.png)
如果一个表根据last names时存储数据，表将会分成多个parittions存储在不同的节点上。基于last names的某个特定范围的查询，比如所有的last name 为Matthews名字的cyclists，会导致表的顺序查询，但是基于age的查询，比如哪些cyclists 28岁，会导致所有节点都会去查一个值。不是primary keys在数据存储时是乱序的。这种根据非主键的查询会导致全partitions的扫描。扫描所有的partitions会导致非常高昂的读延迟，是不被允许的。

Secondary indexes 可以为表的某一个列构建。这些indexes通过一个后台进程都存储在每个节点的本地的一个隐藏表中。如果一个secondary index被用在一个查询中而且没有限制一个特定的partition key,这样的query同样会有高昂的查询延迟，因为所有的节点都得查询。带这些参数的查询只允许查询选项为 ALLOW FILTERING。这个选项不适用生产环境。如果一个查询包括一个partiton key的条件和secondary index列条件，这样的查询才会成功，因为会被转换为单个节点的分区。

然而这种技术并不能保证零麻烦的索引，因此需要知道[什么时候用/不用index](http://docs.datastax.com/en/cql/3.3/cql/cql_using/useWhenIndex.html)。像上面描述的例子，age列上的index可以使用，但是更好的解决方案是创建一个物化视图或者额外的一张根据age排序的表。

和关系型数据一样，维护index需要处理时间和资源，因此不必要的indexes应该要避免。当一个列被更新的时候，它的index同样需要更新。如果当一个老的列值还存在于memtable中，这个经常发生在重复的更新某些行，Cassandra会将对应的过时的index删除;否则老的index entry仍然会被compaction清除掉。如果读操作在compaction清理之前看到一个过时的index entry，reader线程会设置它无效。

**注:**
> 在[Casssandra Secondary Index介绍](http://blog.csdn.net/FS1360472174/article/details/52733434)一文中有详细介绍过Secondary Indx