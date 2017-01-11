RackInferringSnitch 通过机架和数据中心来决定节点的距离，分别和节点ip地址的第三位、第二位对应。这个探测器时用来写自定义探测器最好的例子。(除非这正好匹配你的部署协议)

![](http://docs.datastax.com/en/cassandra/3.0/cassandra/images/arc_rack_inferring_snitch_ips.png)


**注:**
> 这个探测器实现起来非常粗暴，就是取ip
> `public String getRack(InetAddress endpoint)
    {
        return Integer.toString(endpoint.getAddress()[2] & 0xFF, 10);
    }`