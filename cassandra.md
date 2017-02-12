#cassandra 如何读写#



为了管理和访问cassandra中的数据,非常有必要理解cassandra 如何存储数据的。

hinted handoff 特征为Cassandra的ACID\(原子性，一致性，隔离性，持久性\)增加了一致和【non-conformance】

数据库属性是理解读和写的主要概念。在Cassandra,一致性指的是如何保证数据最新，以及每一行的数据的各个副本都是同步的

的。



开发应用程序的数据存储和查询的客户端的工具和APIs在\[这边\]\(http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html\)



#数据是如何写入的#



