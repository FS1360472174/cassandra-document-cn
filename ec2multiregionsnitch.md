当在Amazon EC2中的cassandra集群需要跨多地区的时候，使用Ec2MultiRegionSnitch。

当使用Ec2MultiRegionSnitch时,必须要在cassandra.yaml文件和属性文件cassandra-rackdc.properties中配置设置。

**cassandra.yaml文件配置跨区域通信**

Ec2MultiRegionSnitch 指定**broadcast_address**值为public IP,以此来允许跨地区的连接。将每个节点配置如下:

1. 在cassandra.yaml文件，设置listen_address 为节点的私有IP地址，broadcast_address设置为节点的public IP.

这样可以使得在EC2 某个region的Cassandra 节点可以绑定到另外的region,从而支持了多数据中心。对于region内部的流量，Cassandra会切换到private IP建立连接。

2. 在cassandra.yaml文件中设置seed nodes为public IP.私有IP不会再网络间被路由到。如:

	`seeds: 50.34.16.33, 60.247.70.52`

对于EC2中每一个seed nodes,可以通过下面指令找到public IP 地址

	`curl http://instance-data/latest/meta-data/public-ipv4`

**Note:**不要讲所有的节点都作为seeds,具体查看[gossip](http://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archGossipAbout.html)

3. 确保 storage_port(7000)或者ssl_storage_port(7001)没有被防火墙屏蔽

**配置snitch跨地区通信**

在EC2部署，地区(region)的名字作为数据中心的名字。区域(zones)被当做数据中心中的机架。例如，如果一个节点在us-east-1区域，us-east是数据中心的名字，1是机架的位置。(机架对于分发副本很重要，而不是为了数据中心的命名)

对于每个节点，需要在cassandra-rackdc.properties文件中指定它的数据中心。dc_suffix 选项定力了snitch将用到的数据中心。其他行会被忽略。

在下面的例子中，这儿有两个cassandra数据中心，每个数据中心根据负载命名。在这个例子中，数据中心命名习惯是根据负载性质来定的。你可以使用其他的规范，如DC1，DC2,100，200.(数据中心的名字大小写敏感)


<table cellpadding="4" cellspacing="0" summary="" id="archSnitchEC2MultiRegion__example-table" class="table" frame="border" border="1" rules="all"><colgroup><col style="width:50%"><col style="width:50%"></colgroup><thead class="thead" style="text-align:left;">
            <tr>
              <th class="entry cellrowborder" style="vertical-align:top;" id="d4154e149">Region: us-east</th>

              <th class="entry cellrowborder" style="vertical-align:top;" id="d4154e152">Region: us-west</th>

            </tr>

          </thead>
<tbody class="tbody">
            <tr>
              <td class="entry cellrowborder" style="vertical-align:top;" headers="d4154e149 ">Node and datacenter:<ul class="ul">
                  <li class="li"><strong class="ph b">node0</strong>
                    <p class="p"><code class="ph codeph">dc_suffix=_1_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node1</strong><p class="p"><code class="ph codeph">dc_suffix=_1_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node2</strong><p class="p"><code class="ph codeph">dc_suffix=_2_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node3</strong>
                    <p class="p"><code class="ph codeph">dc_suffix=_2_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node4</strong>
                    <p class="p"><code class="ph codeph">dc_suffix=_1_analytics</code></p>
</li>

                  <li class="li"><strong class="ph b">node5</strong>
                    <p class="p"><code class="ph codeph">dc_suffix=_1_search</code></p>
</li>

                </ul>
<div class="p">This results in four <span class="keyword parmname">us-east</span> datacenters:<pre class="pre codeblock no-highlight"><code>us-east_1_cassandra
us-east_2_cassandra
us-east_1_analytics
us-east_1_search</code></pre></div>
</td>

              <td class="entry cellrowborder" style="vertical-align:top;" headers="d4154e152 ">Node and datacenter:<ul class="ul">
                  <li class="li"><strong class="ph b">node0</strong><p class="p"><code class="ph codeph">dc_suffix=_1_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node1</strong><p class="p"><code class="ph codeph">dc_suffix=_1_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node2</strong><p class="p"><code class="ph codeph">dc_suffix=_2_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node3</strong>
                    <p class="p"><code class="ph codeph">dc_suffix=_2_cassandra</code></p>
</li>

                  <li class="li"><strong class="ph b">node4</strong><p class="p"><code class="ph codeph">dc_suffix=_1_analytics</code></p>
</li>

                  <li class="li"><strong class="ph b">node5</strong>
                    <p class="p"><code class="ph codeph">dc_suffix=_1_search</code></p>
</li>

                </ul>
<div class="p">This results in four <span class="keyword parmname">us-west</span> datacenters:<pre class="pre codeblock no-highlight"><code>us-west_1_cassandra
us-west_2_cassandra
us-west_1_analytics
us-west_1_search</code></pre></div>
</td>

            </tr>

          </tbody>
</table>

**Keyspace Strategy 选项**

当定义keyspace strategy 选项，使用EC 地区名字，如'us-east'作为数据中心的名字。

    相关的信息
    [install locationl](http://docs.datastax.com/en/cassandra/3.0/cassandra/install/referenceInstallLocationsTOC.html)