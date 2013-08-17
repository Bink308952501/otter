<div class="blog_content">
    <div class="iteye-blog-content-contain" style="font-size: 14px;">
<h1>基本需求</h1>
<ol>
<li>网络不可靠，异地机房尤为明显.  </li>
<li>manager/node的jvm不可靠，需要考虑异常crash情况</li>
<li>node的jvm不可靠，需要考虑异常crash的情况</li>
<li>数据库不可靠，需要考虑数据库切换，比如binlog获取和数据载入时，都需要考虑数据库HA机制</li>
<li>系统发布时，排除正常的jvm关闭和启动</li>
</ol>
<h1>实现思路</h1>
<h3>1.  考虑node和manager独立部署</h3>
<p>     manager对于node来说可以是一个optional的环境，只有在第一次启动任务时需要，node一旦启动了同步任务后，无论manager是否可用，不能影响正常同步。</p>
<p>需要考虑的点：</p>
<ul>
<li>node对于配置需要有本地cache</li>
<li>node推送统计信息到manager需要有容错处理，需要考虑manager failover(一台manager挂了，需要链接到另一台).  </li>
</ul>
<p>目前otter内部，manager部署2台，manager主要集中在杭州机房，node部署70+，node分布在各个机房。</p>
<p> </p>
<h3>2.  建议异常流程处理机制</h3>
<p><img width="681" height="462" alt="" src="http://dl2.iteye.com/upload/attachment/0088/3105/0af1957d-1bfd-3473-9e11-19b3da7e7e54.png"></p>
<p>otter调度系统在设计的时候，会有个假定，认为90%的情况都是正常工作的，所以一旦出现异常，处理的代价相对比较高，会使用分布式锁机制。</p>
<p>仲裁器设计了三种异常机制指令：</p>
<ul>
<li>WARNING  :  只发送报警信息，不做任何S/E/T/L调度干预</li>
<li>ROLLBACK :   尝试获取分布式锁，避免并发修改，其次修改分布式Permit为false，停止后续的所有S/E/T/L调度，然后删除所有当前process调度信息，通过zookeeper watcher通知所有相关node，清理对应process的上下文，pipe的数据存储会通过TTL来进行清理，不需要ROLLBACK干预。完成后，释放锁操作</li>
<li>RESTART ： 前面几个步骤和ROLLBACK基本类似，唯一不同点在于，在释放锁之前会尝试修改分布式Permit为true，重新开启同步，然后释放锁.  </li>
</ul>
<p>罗列了一下不同异常对应的处理机制：</p>
<ul>
<li>两个节点通讯时网络异常，节点发起ROLLBACK</li>
<li>节点执行S/E/T/L模块，比如写数据库出现网络异常，节点发起ROLLBACK</li>
<li>节点发生了CRASH，由manager进行监听，manager发现后发起RESTART</li>
</ul>
<h3>3.  node节点监控原理：(和hadoop/hbase原理基本一致，利用zookeeper)</h3>
<ul>
<li>每个node在启动完成后，都会在zookeeper中创建一个Ephemerals节点(此节点特点，当node节点发生crash之后，与zookeeper建立的sesstion因为没有心跳，超过一定时间后就会出现SesstionExpired，然后zookeeper会删除该节点)</li>
<li>manager监听整个node节点列表的变化，任何一个node节点的消失，都会收到zookeeper watcher通知，与内存中上一个版本进行比较，判断出当前消失的node节点</li>
<li><span style="font-family: arial; font-size: small;">针对该消失的node节点，会有一段保护期(因为可能正常的发布，会关闭node，同样会触发该watcher)，如果该node在保护期内重新启动了，则不做任何处理。默认保护期为90秒</span></li>
<li><span style="font-family: arial; font-size: small;">如果保护期内node节点未正常启动，说明node是异常crash，通过查询配置，找到使用了该node的所有同步任务，对每个同步任务发起一个RESTART指令，让所有同步任务重新做一次负载均衡选择，避免挂死在老的node上，一直死等其结果返回。</span></li>
</ul>
<h3>4.  数据库切换</h3>
<p>数据库异常问题多种多样，比如数据库hang住，数据库不可用，数据库不可写等等。 在阿里巴巴内部一般会有DBA控制数据库的切换问题。</p>
<p>比如会有一套管理系统，配置当前mysql主备的关系，发现主机不可用时，他们会通过该系统切备机变为主机，然后推送该配置到所有节点，然后各个客户端收到主备切换消息，更改自己的数据库链接，完成数据库切换。</p>
<p> </p>
<p>因为内部系统无法直接开源，在otter开源版本中，也自带了一个简单版的数据库主备推送的机制，通过页面上的切换按钮，就可以通知到所有的otter节点，切换数据库链接，包括canal的binlog解析和otter数据库loader等。</p>
<p> </p>
<p>otter内部配置中称之为主备配置(media配置)，为一对主备IP，定义一个groupKey。然后在各个地方使用该groupKey。</p>
<ul>
<li>比如jdbc url使用group后为：jdbc:mysql://groupKey=xxxx</li>
<li>canal中可以选择HA机制为media，然后填入对应的groupKey即可</li>
</ul>
<p>发现需要做数据库切换了，可以直接点击切换按钮，目前otter的实现为定时轮询非推送，一般需要1分钟左右才会正式生效，或者发生一次RESTART指令。同样，主备切换可以暴露为服务，方便大家接入各自的数据库管理平台，这也是otter抽象这么一层主备切换配置的原因</p>
<p> </p>
</div>