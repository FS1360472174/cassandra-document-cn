可以使用GoogleCloudSnitch为[Google Cloud Platform](https://cloud.google.com/)提供单个地区或多地区的Cassandra 部署。地区作为数据中心，区域(zones)被当做数据中心中的机架。在相同逻辑网络中所有的通信都通过私有IP地址进行通信。

region名字作为数据中心名字，zones作为数据中心里面的机架。例如，如果一个节点在us-central1-a region，us-central1是数据中心的名字，a是机架的位置(机架对于分发副本很重要，而不是为了数据中心的命名)这种snitch可以跨region而不需要额外的配置。

如果你只使用单数据中心，不需要指定任何的属性。

如果使用多数据中心，需要在cassandra-rackdc.properties配置文件中设置dc_suffix选项。其他行会被忽略。

例如，us-central1地区的每个节点，在cassandra-rackdc.properties文件中指定数据中心。

**Note:** 数据中心名字大小写敏感

- node0
	
	dc_suffix=_a_cassandra

- node1

	dc_suffix=_a_cassandra

- node2

	dc_suffix=_a_cassandra

- node3

	dc_suffix=_a_cassandra

- node4

	dc_suffix=_a_analytics

- node5
	
	dc_suffix=_a_search

**Note:** 数据中心名字和机架名字大小写敏感