# CQL初识 #

#Cassandra查询语言概况#

Cassandra 查询语言（CQL）是Cassandra数据库的查询语言。这个版本的CQL能够工作在Cassandra3.0 linux平台。

Cassandra 查询语言（CQL）是和cassandra数据库通信的主要语言。和Cassandra交互的最基本方式就是通过CQL shell，即cqlsh，你可以创建keyspaces,tables,插入和查询表，等等。如果你喜欢图形化工具，可以使用[DataStax DevCenter](http://docs.datastax.com/en/developer/devcenter/doc/devcenter/features.html)。生产开发时，DataStax提供了一系列的[drivers](http://docs.datastax.com/en/developer/driver-matrix/doc/common/driverMatrix.html),这样CQL语句就能通过客户端发送给集群。其他的一些管理员的工作可以交给[OpsCenter](http://docs.datastax.com/en/latest-opsc/)

**重要：**这篇文档基于你很熟悉[Cassandra3.0文档](http://docs.datastax.com/en/cassandra/3.0/cassandra/cassandraAbout.html)

# CQL for Cassandra3.0功能 #

## 新增CQL功能 ##

- CQL3支持[JSON](http://docs.datastax.com/en/cql/3.3/cql/cql_using/useInsertJSON.html)
- 用户定义函数(UDFs)
- 用户定义聚合(UDAs)
- 基于角色的访问控制（RBAC）
- Native Protocol v.4
- 物化视图
- 新增的cqlsh CLEAR命令

## 改善的功能 ##

- 新增的COPY命令选项
- CREATE TABLE 命令新增WITH ID 选项
- 支持 IN 语法在任意的partition key列或者clustering 列
- 接收$引用字符串
- 允许混合Token和Partition Key限制
- 支持对Map类型数据的Key/Value建立索引
- 新增Tinyint 和smallint数据类型
- 更改CREATE TABLE compression 选项语法

## 移除掉的功能 ##

- 移除CQL2
- 移除cassandra-cli

## 本地协议 ##

Native 协议更新到v4,在DataStax drivers已经实现