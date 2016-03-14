<h1>项目介绍</h1>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">名称：otter ['ɒtə(r)]</p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">译意： 水獭，数据搬运工</p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">语言： 纯java开发</p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">定位： 基于数据库增量日志解析，准实时同步到本机房或异地机房的mysql/oracle数据库. </p>
<p> </p>
<h1>工作原理</h1>
<p><img width="848" src="http://dl2.iteye.com/upload/attachment/0088/1189/d420ca14-2d80-3d55-8081-b9083606a801.jpg" height="303" alt=""></p>
<p>原理描述：</p>
<p>1.   基于Canal开源产品，获取数据库增量日志数据。 什么是Canal,  请<a href="https://github.com/alibaba/canal">点击</a></p>
<p>2.   典型管理系统架构，manager(web管理)+node(工作节点)</p>
<p>&nbsp;&nbsp;&nbsp;     a.  manager运行时推送同步配置到node节点</p>
<p>&nbsp;&nbsp;&nbsp;     b.  node节点将同步状态反馈到manager上</p>
<p>3.  基于zookeeper，解决分布式状态调度的，允许多node节点之间协同工作. </p>
<p> </p>
<h1>otter能解决什么？</h1>
<p>1.  异构库同步</p>
<p>&nbsp;&nbsp;&nbsp;   a.  mysql -&gt;  mysql/oracle.  (目前开源版本只支持mysql增量，目标库可以是mysql或者oracle，取决于canal的功能)</p>
<p>2.  单机房同步 (数据库之间RTT &lt; 1ms)</p>
<p>&nbsp;&nbsp;&nbsp;   a. 数据库版本升级</p>
<p>&nbsp;&nbsp;&nbsp;   b. 数据表迁移</p>
<p>&nbsp;&nbsp;&nbsp;   c. 异步二级索引</p>
<p>3.  异地机房同步 (比如阿里巴巴国际站就是杭州和美国机房的数据库同步，RTT &gt; 200ms，<strong>亮点</strong>)</p>
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
<h1>相关名词解释</h1>
<p style="font-size: 14px;"> </p>
<h3 style="font-size: 14px;">otter核心model关系图</h3>
<p style="font-size: 14px;"><img alt="" src="http://dl2.iteye.com/upload/attachment/0088/3048/a5583dc2-a337-3583-8b92-04298ef6fb74.jpg" height="409" width="545"></p>
<p style="font-size: 14px;"> </p>
<h3 style="font-size: 14px;">名词解释</h3>
<ul>
<li>Pipeline：从源端到目标端的整个过程描述，主要由一些同步映射过程组成</li>
<li>Channel：同步通道，单向同步中一个Pipeline组成，在双向同步中有两个Pipeline组成</li>
<li>DateMediaPair：根据业务表定义映射关系，比如源表和目标表，字段映射，字段组等</li>
<li>DateMedia : 抽象的数据介质概念，可以理解为数据表/mq队列定义</li>
<li>DateMediaSource : 抽象的数据介质源信息，补充描述DateMedia</li>
<li>ColumnPair : 定义字段映射关系</li>
<li>ColumnGroup : 定义字段映射组</li>
<li>Node : 处理同步过程的工作节点，对应一个jvm</li>
</ul>
----
<h3 style="font-size: 14px;">otter的S/E/T/L stage阶段模型</h3>
<p><img alt="" src="http://dl2.iteye.com/upload/attachment/0088/3073/6d6e4f03-23df-3759-b7db-370e08c7b34c.png"></p>
说明：为了更好的支持系统的扩展性和灵活性，将整个同步流程抽象为Select/Extract/Transform/Load，这么4个阶段. 
<p>Select阶段: 为解决数据来源的差异性，比如接入canal获取增量数据，也可以接入其他系统获取其他数据等。</p>
<p>Extract/Transform/Load 阶段：类似于数据仓库的ETL模型，具体可为数据join，数据转化，数据Load的</p>
<p> </p>
<h1>相关实现介绍</h1>
* &nbsp;&nbsp;&nbsp;[[Otter调度模型]]
* &nbsp;&nbsp;&nbsp;[[Otter数据入库算法]]
* &nbsp;&nbsp;&nbsp;[[Otter双向回环控制]]
* &nbsp;&nbsp;&nbsp;[[Otter数据一致性]]
* &nbsp;&nbsp;&nbsp;[[Otter高可用性]]
* &nbsp;&nbsp;&nbsp;[[Otter扩展性]]