**使用最新的JVM**  
使用最新的64位[Oracle Java Platfom标准的JDK8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)，或者[OpenJDK8](http://openjdk.java.net/)

**同步时钟**  
同步所有节点上的时钟，使用NTP\(网络时间协议\)或者其他方法。  
之所以需要是因为当机器在不同的地理位置时，Cassandra 会覆盖掉某一列当这列有个更新版本的时间戳

Cassandra 时间戳是按照微妙编码的，因为UNIX日期不带时区信息。Cassandra 所有写入的时间戳是形式是UTC，DataStax建议只有当需要输出给人看的时候，才转化为本地的一个时间。

**TCP配置**

为了处理Cassandra的成千上万的并发连接，以下Linux网络按以下配置。将这些配置加到/etc/sysctl.conf

```
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.rmem_default = 16777216
net.core.wmem_default = 16777216
net.core.optmem_max = 40960
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
```

让配置立即生效\(取决于你的linux发行版本\)

```
sudo sysctl -p /etc/sysctl.conf
```

```
sudo sysctl -p /etc/sysctl.d/filename.conf
```

**禁掉CPU调频**

最近的linux系统都会包含一个模块叫做CPU调频，或者CPU速度调节。它允许一个机器时钟速度能够动态的调节，这样机器就可以在负载低的时候以低速度工作。这样可以降低机器电量的消耗，和散热（这会极大的影响制冷花费）。不幸的是，这种行为对于运行了Cassandra的机器有不利的影响，因为并发量会固定在一个低的速值。

在大部分的linux系统，CPUfreq基于定义的规则管理频率的调节，默认的ondemand调频器会在负载高的时候将时钟频率切换到最高，当系统空闲的时候讲时钟频率切换到最低。

不要使用默认的调频器来降低CPU频率。为了确保获得一个好的性能，使用performance 调频器来重新配置所有的CPUs。performance 调频器会将频率锁定在最大值。这种调频器不会切换频率，这就意味着不能节省电力但是机器可以一直在最大的并发量上运行。对于大部分的系统，按以下配置调频器

```
for CPUFREQ in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
do
    [ -f $CPUFREQ ] || continue
    echo -n performance > $CPUFREQ
done
```

\[了解更多信息\]\(https://support.datastax.com/hc/en-us/articles/115003018063\)

**确保重启后新的配置生效**

注意：

取决于你的环境，下面的一些配置可能在重启机器后不生效，和你系统管理员确认，确保这些配置在你的环境生效







