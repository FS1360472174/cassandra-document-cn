这一节包含如何部署一个单数据中心的Cassandra集群。如果你刚刚接触Cassandra，而且从来没有搭建过一个集群，
看下[准备和测试集群部署](http://docs.datastax.com/en/dse-planning/doc/)

##准备##
在开始一个集群前必须保证灭一个节点都正确配置了。必须理解或者操作完下面这些说明

- 理解Cassandra是如何工作的，至少要确保读完了[理解架构](http://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archTOC.html),尤其是数据复制、
Cassandra机架特性章节。
- 在每个节点上安装Cassandra
- 为集群选择一个名字
- 获取每个节点的IP
- 决定哪些节点时seed 节点。**不要将所有节点都设置为种子节点。**请阅读[内部节点通信](http://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archGossipAbout.html)
- 设置`snitch`和`replication strategy`。`GossipingPropertyFileSnitch` 和`NetworkTopologyStrategy`是推荐的线上配置
- 为每个机架设置一个名字转换。例如，一个好的名字类似于RAC1，RAC2或者RAC101，RAC102。
- cassandra.yaml配置文件，和其他属性文件如cassandra-rackdc.properties给你更多的配置选项。[配置](http://docs.datastax.com/en/cassandra/3.0/cassandra/configuration/configTOC.html)可以找到更多的信息。


这个例子描述了如何安装一个单数据中心，2个机架6个节点的集群。每个节点都已经使用`GossipingPropertyFileSnitch`和256个虚拟节点配置好了。

在Cassandra中，"数据中心"是等价于"复制组"。这两个名词指的都是为了复制的目的，将节点配置在一起。

##过程##

1.假设你在这些节点上安装了Cassandra

    
    ```
    node0 110.82.155.0 (seed1)
    node1 110.82.155.1
    node2 110.82.155.2
    node3 110.82.156.3 (seed2)
    node4 110.82.156.4
    node5 110.82.156.5     
    ```
    
   **注：** 推荐在每个数据中心有一个以上的种子节点   
  

2.如果你在集群中开启了防火墙，必须确定开启了节点之间通信的特定的端口

3.如果Cassandra已经在运行了，必须要停掉机器并清除到数据，从System表中移除掉默认的集群名字(Test Cluster)，所有的节点必须使用相同的集群名字。

 包安装

a.停掉Cassandra
    
    ```
    $ sudo service cassandra stop
    ```
    
b. 清除数据
    ``
    $ sudo rm -rf /var/lib/cassandra/data/system/*
    ``
Tarball 安装：
   a. 停掉Cassandra
   ``
   $ps auwx|grep cassandra
   sudo kill pid
   
   ``
   b. 清除掉数据
   ``
   $sudo rm -rf /var/lib/cassandra/data/data/system/*
   
   
4.为每个节点设置cassandra.yaml文件的属性
  **注：**更新了cassandra.yaml文件的配置，必须要重启节点才能生效
  
  设置属性：
  - cluster_name：
  - num_tokens: 推荐值:256
  - -seeds: 每个seed node的内部IP地址，在新的集群中，seed node不会进行bootstrap(一个新节点计入到一个已有集群的过程)
  - listen_address: 如果一个节点时种子节点，这个地址必须要和种子节点列表中的IP地址匹配。否则，gossip通信会失败因为它不知道这是个种子。
  
  如果没有设置listen_address值，Cassandra会向系统询问local地址，和hostname关联的。在有些情况下，Cassandra不会生成正确的地址，所以你必须明确指定listen_address
  - rpc_address: 客户端连接的监听地址
  - endpoint_snitch: snitch的名字(看下[endpoint_snitch](http://docs.datastax.com/en/cassandra/3.0/cassandra/configuration/configCassandra_yaml.html#configCassandra_yaml__endpoint_snitch))。如果你改变了snitches，看下[Switching snitches](http://docs.datastax.com/en/cassandra/3.0/cassandra/operations/opsSwitchSnitch.html)
   - auto_bootstrap:false (添加这个配置**只有**当初始化一个没有数据的节点)
   
   **注：**如果集群中的集群中的节点在磁盘，共享库，等方面都一样。可以在所有节点上使用一样的cassandra.yaml文件。
   
   
   例子：
   ```
     cluster_name: 'MyCassandraCluster'
     num_tokens: 256
     seed_provider:
     - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
     - seeds: "110.82.155.0,110.82.155.3"
    listen_address:
    rpc_address: 0.0.0.0
    endpoint_snitch: GossipingPropertyFileSnitch
   ```
   
   如果rpc_address设置成了通配符地址(0.0.0.0),那么broadcast_rpc_address必须要设置，否则这个服务不会启动。
   
 5.在cassandra-rackdc.properties文件中，设置你在[准备]中决定的数据中心和机架名字。例如
 
 ```
   dc=DC1
   rack=RAC1
 ```
 6.GossipingPropertyFileSnitch经常加载cassandra-topology.properties当这个文件存在的时候，在任何新的集群上或者从PropertyFileSnitch迁移过来的集群，都要在每个节点上将这个文件移除。
 7.当你安装和配置完所有的节点。DataStax推荐同时启动seed node。然后启动其他的节点
 
 **注：**如果一个节点因为自动重启而重启了，你必须首先停止这个节点，然后清除掉文件目录。
 
 8.检查ring是否启动工作
 
 包安装
 `nodetool status`
 Tarball安装
 ```
   cd install_location
   bin/nodetool status
 ```
 输出必须列出每一个节点，并且显示状态为UN(Up Normal)

