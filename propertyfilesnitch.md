这个探测器通过机架和数据中心来决定节点的距离。使用cassandra-topology.properties 文件中定义的网络拓扑细节。当使用这个探测器可以将数据中心的名字定义为任何你想要的。确保定义keyspace时指定的名字和这边定义的一样。集群中的每个节点都应该在cassandra-topology.properties文件中定义。而且集群中的每个节点这个文件都应该一样。

**过程**

如果你的节点有有不一样ip，集群中有两个物理数据中心每个数据中心都有两个机架。第三个逻辑数据中心用来复制分析数据。配置文件可能看起来像下面这样：


**Note:** 数据中心和机架的名字是大小写敏感的.

   
    # datacenter One
    
    175.56.12.105=DC1:RAC1
    175.50.13.200=DC1:RAC1
    175.54.35.197=DC1:RAC1
    
    120.53.24.101=DC1:RAC2
    120.55.16.200=DC1:RAC2
    120.57.102.103=DC1:RAC2
    
    # datacenter Two
    
    110.56.12.120=DC2:RAC1
    110.50.13.201=DC2:RAC1
    110.54.35.184=DC2:RAC1
    
    50.33.23.120=DC2:RAC2
    50.45.14.220=DC2:RAC2
    50.17.10.203=DC2:RAC2
    
    # Analytics Replication Group
    
    172.106.12.120=DC3:RAC1
    172.106.12.121=DC3:RAC1
    172.106.12.122=DC3:RAC1
    
    # default for unknown nodes 
    default =DC3:RAC1


**注：**
> 这种配置方式应该是比较常见的方式，笔者也常常这么干。清晰明了，简单易懂。唯一的问题在于进行节点扩展时，需要更新所有的节点上此配置文件。不过不用重启节点令配置生效。默认刷新的时间是5s
>     
>      org.apache.cassandra.locator.PropertyFileSnitch
>      private static final int DEFAULT_REFRESH_PERIOD_IN_SECONDS = 5;
