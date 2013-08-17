<h1>otter能解决什么？</h1>
<p>1.  异构库同步</p>
<p>&nbsp;&nbsp;&nbsp;   a.  mysql -&gt;  mysql/oracle.  (目前开源版本只支持mysql增量，目标库可以是mysql或者oracle，取决于canal的功能)</p>
<p>2.  单机房同步 (数据库之间RTT &lt; 1ms)</p>
<p>&nbsp;&nbsp;&nbsp;   a. 数据库版本升级</p>
<p>&nbsp;&nbsp;&nbsp;   b. 数据表迁移</p>
<p>&nbsp;&nbsp;&nbsp;   c. 异步二级索引</p>
<p>3.  异地机房同步 (比如阿里巴巴国际站就是杭州和美国机房的数据库同不，RTT &gt; 200ms，<strong>亮点</strong>)</p>
<p>&nbsp;&nbsp;&nbsp;   a. 机房容灾</p>
<p>4.  双向同步</p>
<p>&nbsp;&nbsp;&nbsp;    a.  避免回环算法  (通用的解决方案，支持大部分关系型数据库)</p>
<p>&nbsp;&nbsp;&nbsp;    b.  数据一致性算法   (保证双A机房模式下，数据保证最终一致性，<strong>亮点</strong>)</p>
<p>5.  文件同步</p>
<p>&nbsp;&nbsp;&nbsp;    a.  站点镜像  (进行数据复制的同时，复制关联的图片，比如复制产品数据，同时复制产品图片).</p>
<p> </p>
<h3>单机房复制示意图：</h3>
<p><img height="249" width="563" src="http://dl2.iteye.com/upload/attachment/0088/1975/dede22c8-59ca-378a-90d5-4f45b289ab30.jpg" alt=""></p>
<p>说明： </p>
<p>   &nbsp;&nbsp;&nbsp;a.  数据on-Fly，尽可能不落地，更快的进行数据同步.  (开启node <span style="line-height: 1.5;">loadBalancer算法，如果Node节点S+ETL落在不同的Node上，数据会有个网络传输过程</span><span style="line-height: 1.5;">)</span></p>
<p>   &nbsp;&nbsp;&nbsp;b.  node节点可以有failover /  loadBalancer.  </p>
<p> </p>
<h3>异地机房复制示意图：</h3>
<p><img height="332" width="667" src="http://dl2.iteye.com/upload/attachment/0088/1981/5369b533-5b9a-32e6-bbc0-14c407188e93.jpg" alt=""></p>
<p>说明： </p>
<p>   &nbsp;&nbsp;&nbsp;a.  数据涉及网络传输，S/E/T/L几个阶段会分散在2个或者更多Node节点上，多个Node之间通过zookeeper进行协同工作  (一般是Select和Extract在一个机房的Node，Transform/Load落在另一个机房的Node)</p>
<p>   &nbsp;&nbsp;&nbsp;b.  node节点可以有failover /  loadBalancer.  (每个机房的Node节点，都可以是集群，一台或者多台机器)</p>
<p> </p>
<h3>初步性能指标：</h3>
<p>1.  单机房同步</p>
<p>   &nbsp;&nbsp;&nbsp;a.  100tps ， 延迟100ms</p>
<p>   &nbsp;&nbsp;&nbsp;b.  5000tps,  延迟1s</p>
<p>2.  中美异地机房同步</p>
<p>   &nbsp;&nbsp;&nbsp;a.  100tps ， 延迟2s</p>
<p>   &nbsp;&nbsp;&nbsp;b.  5000tps ，延迟10s</p>
ps. 性能指标取决于目标数据库性能，数据大小等多个因素，单机房100b大小，极限tps可以1w+
<p> </p>