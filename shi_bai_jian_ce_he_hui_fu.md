# 失败检测和恢复

失败检测是一种为本地决策提供信息的方法，从gossip的状态和历史获取信息，判断系统中的一个节点是否down了或者已经恢复了。Cassandra 利用这个信息避免将客户端的请求路由到任何时候有可能不可到达的节点。\(cassandra 同样能够通过[Dynamic Snitch](http://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archSnitchesAbout.html "dynamic snitch")\)避免将客户端请求路由到那些存活的但是性能比较差的节点上。

gossip过程能够跟踪其他节点的状态，通过直接\(直接与某个节点gossip\)或非直接\(通过二手，三手等\)方式。相比于一个固定的阈值来标记一个节点为fail，Cassandra 采用一个自然增长的检测机制来计算每个节点的阈值，考虑到了网络、负载、历史状况等因素。当进行gossip交换时，每个节点维护了一个其他节点gossip信息到达的滑动窗口时间。可以通过配置[phi\_convict\_threshold](http://docs.datastax.com/en/cassandra/3.0/cassandra/configuration/configCassandra_yaml.html#configCassandra_yaml__phi_convict_threshold "phi\_convict\_threshold")属性来调节失败检测的敏感性。值越低，一个没有应答的节点更有可能被标记为down,值越高，短暂的失败更低可能的被标记为失败。大部分情况下，默认值就可以了。但是在Amazon EC2上需要增加到10或者12.\(因为常常会遇到网络拥堵\)，在不稳定的网络环境中\(比如EC2\)，提高值到10或者12可以帮助避免错误的失败检测。不建议使用高于12，或者低于5的值。

节点失败可能有各种各样的原因造成的，比如硬件失败，网络电力供应中断。节点中断经常是短暂的但是有可能持续很长时间的。因为一个节点中断很少意味着永久离开集群，不会自动从集群ring中移除。其他的节点会周期性的尝试和失败的节点重新建立联系，看它们是否已经回归。想要永久的改变集群节点的成员关系，需要管理员通过notetool明确的将节点添加进来或者移除出集群。

当一个节点经过down到重新回归的，可能会丢失掉它需要维护的副本数据。[repair](http://docs.datastax.com/en/cassandra/3.0/cassandra/operations/opsRepairNodesTOC.html "Repair mechanisms")可以帮助恢复这些数据，比如hinted handoffs以及手动repair.节点down掉的时间决定了通过哪种机制来保持数据的一致性。

**注：**

> hintedhandoff有时间限制，默认三小时，超过此时间前面的数据会不断的被覆盖掉。必须要手动repair



