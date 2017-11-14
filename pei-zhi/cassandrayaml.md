# 配置文件 cassandra.yaml

cassandra.yaml 是Cassandra 的主要配置文件，核心的配置项都在该文件下管理。

**注：**

> 对cassandra.yaml 中的配置项进行修改之后，必须要重启节点以使其生效。cassandra.yaml 文件的存储目录为：
>
> * Cassandra package 安装: /etc/cassandra
>
> * Cassandra tar 包安装: install\_location（安装目录）/conf
>
> * DataStax 企业版 package 安装: /etc/dse/cassandra
>
> * DataStax 企业版 tar 包安装: install\_location（安装目录）/resources/cassandra/conf



cassandra.yaml文件中的配置项分为以下几个部分：

* ##### **快速启动**

        Cassandra集群最基础的配置项。

* ##### 常规项

        配置Cassandra常用的一些配置项。

* ##### 性能优化

        优化性能以及系统资源利用率，具体包括commit log, compaction, 内存, 磁盘 I/O, CPU, 读写操作等。

* ##### 高级选项

        高级选项在常规情况下使用的比较少。

* ##### 安全

        服务端和客户端的安全配置。

**注：**

> 在cassandra.yaml配置文件中，当出现不配置，注释掉，或者其他程序的实现依赖于cassandra.yaml的配置项时，都会使用配置项在程序内部的默认值。另外，在配置文件中所有注释掉的配置项中的值不一定完全与程序内部的默认值相等，配置文件中仅仅是建议值。

#### 快速启动

Cassandra集群最基础的配置项。

关联知识点：初始化一个多节点Cassandra集群（单一数据中心部署）和 初始化一个多节点Cassandra集群（多数据中心部署）。

##### cluster\_name

      \(默认值：Test Cluster\)集群名称。这个设置能预防节点在加入集群时加到同一个逻辑集群中，而不是其他集群。同一集群中的所有节点的cluster\_name必须相同。

**listen\_address**

     （默认值：localhost）Cassandra绑定的IP地址或主机名，用来对外提供访问地址和连接集群中的其他节点。该配置和listen\_interface只需要配置一个。以下是其几种正确的配置方式：

* 单一节点部署时：下列方法选其一：
* 多节点部署时：配置该机器的机器名或者IP地址，或者使用listen\_interface配置。
* 多节点部署，节点部署在多网络环境或多个数据中心中，比如部署在AWS的EC2环境中并且支持共有和私有网络环境自动切换时：配置该机器的机器名或者IP地址，或者使用listen\_interface配置。
* 多节点部署，节点部署在两个不同的物理网络环境中，并且分属不同的数据中心，或者Cassandra集群部署在亚马逊EC2的多个区域中，并且使用**Ec2MultiRegionSnitch策略时：**

                  





#### 



