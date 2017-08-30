**使用最新的JVM**
使用最新的64位[Oracle Java Platfom标准的JDK8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)，或者[OpenJDK8](http://openjdk.java.net/)

**同步时钟**
同步所有节点上的时钟，使用NTP(网络时间协议)或者其他方法。
之所以需要是因为当机器在不同的地理位置时，Cassandra 会覆盖掉某一列当这列有个更新版本的时间戳

Cassandra 时间戳是按照微妙编码的，因为UNIX日期不带时区信息。Cassandra 所有写入的时间戳是形式是UTC，DataStax建议只有当需要输出给人看的时候，才转化为本地的一个时间。

