集群中的所有节点都在一个地区，这种简单的集群部署在Amazon EC2可以使用Ec2Snitch方法。

在EC2上的部署，地区(region)的名字作为数据中心的名字。区域(zones)被当做数据中心中的机架。例如，如果一个节点在us-east-1区域，us-east是数据中心的名字，1是机架的位置。(机架对于分发副本很重要，而不是为了数据中心的命名)因为使用的是私有IPs,所以探测器无法跨地区。

如果你只使用单数据中心，不需要指定任何的属性。

如果使用多数据中心，需要在cassandra-rackdc.properties配置文件中设置dc_suffix选项。其他行会被忽略。

例如，us-east地区的每个节点，在cassandra-rackdc.properties文件中指定数据中心。

**Note：** 数据中心名字是大小写敏感的

- node0
	
	dc_suffix=_1_cassandra

- node1

	dc_suffix=_1_cassandra

- node2

	dc_suffix=_1_cassandra

- node3

	dc_suffix=_1_cassandra

- node4

	dc_suffix=_1_analytics

- node5

	dc_suffix=_1_search

这样会为该地区生成三个数据中心

    us-east_1_cassandra
	us-east_1_analytics
	us-east_1_search

**Note:** 在这个例子中，数据中心命名习惯是根据负载性质来定的。你可以使用其他的规范，如DC1，DC2,100，200.

**Keyspace Strategy 选项**

当定义keyspace strategy 选项，使用EC 地区名字，如'us-east'作为数据中心的名字

**注:**
> 亚马逊是云主机，提供给用户的只有region 和zone的概念。分别对应着cassandra的数据中心和机架，确保数据不会被放在一起。可以在[AWS regions](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)`查看regions信息。
