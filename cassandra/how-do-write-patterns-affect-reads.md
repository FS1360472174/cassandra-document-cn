# 读是如何影响写的 #

思考集群中写操作会对读操作有什么影响是非常重要的。[compaction 策略](http://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlHowDataMaintain.html)的类型使得数据的处理是可配置的，同时会影响读的性能。使用SizeTieredCompactionStrategy或者DateTieredCompactionStrategy可能会在行被频繁更新的时候造成数据碎片化。LeveledCompactionStrategy(LCS)在这种情况下是被设计用来阻止数据碎片化的。