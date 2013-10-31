 <div class="blog_content">
    <div class="iteye-blog-content-contain" style="font-size: 14px;">
<h1>常见问题</h1>
<h3> 1.  canal和otter的关系？</h3>
<p> 答： 在回答这问题之前，首先来看一张canal&amp;otter和mysql复制的类比图.  </p>
<p><img alt="" src="http://dl2.iteye.com/upload/attachment/0089/0118/f230f237-d6f4-309f-b9ac-bd454da9b69a.jpg" width="655" height="474"></p>
<p>mysql的自带复制技术可分成三步：</p>
<ol>
<li>master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events，可以通过show binlog events进行查看）；</li>
<li>slave将master的binary log events拷贝到它的中继日志(relay log)，这里是I/O thread线程.  </li>
<li>slave重做中继日志中的事件，将改变反映它自己的数据，这里是SQL thread线程.  </li>
</ol>
<p>基于canal&amp;otter的复制技术和mysql复制类似，具有类比性.  </p>
<ol>
<li>Canal对应于I/O thread，接收Master Binary Log. </li>
<li>Otter对应于SQL thread，通过Canal获取Binary Log数据，执行同步插入数据库. </li>
</ol>
<p>两者的区别在于：</p>
<ol>
<li>otter目前嵌入式依赖canal，部署为同一个jvm，目前设计为不产生Relay Log，数据不落地. </li>
<li>otter目前允许自定义同步逻辑，解决各类需求.  <br>a.  ETL转化.  比如Slave上目标表的表名，字段名，字段类型不同，字段个数不同等. <br>b.  异构数据库.  比如Slave可以是oracle或者其他类型的存储,nosql等. <br>c.  M-M部署，解决数据一致性问题<br>d.  基于manager部署，方便监控同步状态和管理同步任务.  </li>
</ol>
<h3>2.  canal目前支持的数据库版本？</h3>
<p>答： 支持mysql系列的5.1 ~ 5.6版本，目前maridb经测试暂不支持.   (全面支持ROW/STATEMENT/MIXED几种binlog格式的解析)</p>
<p> </p>
<h3>3.  otter目前支持的数据库情况？</h3>
<p>答：这里总结了一下</p>
<ol>
<li>从问题1中的图中看到，otter依赖canal解决数据库增量日志，所以会收到canal的版本支持限制，仅支持mysql系列，不支持oracle做为master库进行解析. </li>
<li>mysql做为master，otter只支持ROW模式的数据同步，其他两种模式不支持.  (只有ROW模式可保证数据的最终一致性)</li>
<li>目标库，也就是slave，可支持mysql/oracle，也就是说可以将mysql的数据同步到oracle库中，反过来不行. </li>
</ol>
<h3>4.  otter目前存在的同步限制？</h3>
<p>答：这里总结了一下</p>
<ol>
<li><span style="line-height: 21px;">暂不支持无主键表同步.  (同步的表必须要有主键，无主键表update会是一个全表扫描，效率比较差)</span></li>
<li><span style="line-height: 21px;">支持部分ddl同步  (支持create table / drop table / alter table / truncate table / rename table / create index / drop index，其他类型的暂不支持，比如grant,create user,trigger等等)，同时ddl语句不支持幂等性操作，所以出现重复同步时，会导致同步挂起，可通过配置高级参数:跳过ddl异常，来解决这个问题.  </span></li>
<li><span style="line-height: 21px;">不支持带外键的记录同步.  (数据载入算法会打算事务，进行并行处理，会导致外键约束无法满足)</span></li>
<li><span style="line-height: 21px;">数据库上trigger配置慎重.  (比如源库，有一张A表配置了trigger，将A表上的变化记录到B表中，而B表也需要同步。如果目标库也有这trigger，在同步时会插入一次A表，2次B表，因为A表的同步插入也会触发trigger插入一次B表，所以有2次B表同步.)</span></li>
</ol>
<h3>5.  otter同步相比于mysql的优势？</h3>
<p>答：</p>
<ol>
<li>管理&amp;运维方便.  otter为纯java开发的系统，提供web管理界面，一站式管理整个公司的数据库同步任务.  </li>
<li>同步效率提升.  在保证数据一致性的前提下，拆散原先Master的事务内容，基于pk hash的并发同步，可以有效提升5倍以上的同步效率. </li>
<li>自定义同步功能.   支持基于增量复制的前提下，定义ETL转化逻辑，完成特殊功能. </li>
<li>异地机房同步.   相比于mysql异地长距离机房的复制效率，比如阿里巴巴杭州和美国机房，复制可以提升20倍以上. 长距离传输时，master向slave传输binary log可能会是一个瓶颈.</li>
<li>双A机房同步.   目前mysql的M-M部署结构，不支持解决数据的一致性问题，基于otter的双向复制+一致性算法，可完美解决这个问题，真正实现双A机房. </li>
<li>特殊功能.  <br>a.  支持图片同步.  数据库中的一条记录，比如产品记录，会在数据库里存在一张图片的path路径，可定义规则，在同步数据的同时，将图片同步到目标.  </li>
</ol>
<h3>6.  node jvm内存不够用，如何解决？</h3>
<p style="font-size: 14px;">node出现java.lang.OutOfMemoryError : Gc overhead limit exceeded. </p>
<p style="font-size: 14px;">答：</p>
<p>单node建议的同步任务，建议控制下1~2wtps以下，不然内存不够用.  出现不够用时，具体的解决方案:</p>
<ol>
<li>调大node的-Xms,-Xmx内存设置，默认为3G,heap区大概是2GB</li>
<li>减少每个同步的任务内存配置.<br>a. canal配置里有个内存存储buffer记录数参数，默认是32768，代表32MB的binlog，解析后在内存中会占用100MB的样子. <br>b. pipeline配置里有个批次大小设置，默认是6000，代表每次获取6MB左右，解析后在内存占用=并行度*6MB*3，大概也是100MB的样子. </li>
</ol>
<p> </p>
<p>   所以默认参数，全速跑时，单个通道占用200MB的样子，2GB能跑几个大概能估算出来了</p>
<h3>7.  源库binlog不正确，如何重置同步使用新的位点开始同步？</h3>
<p>场景：</p>
<ul>
<li>源库binlog被删除，比如出现：Could not find first log file name in binary log index file</li>
<li>源库binlog出现致命解析错误，比如运行过程使用了删除性质的ddl，drop table操作，导致后续binlog在解析时无法获取表结构. </li>
</ul>
<p>答：</p>
<p>首先需要理解一下canal的位置管理，主要有两个位点信息：起始位置 和 运行位置(记录最后一次正常消费的位置).   优先加载运行位置，第一次启动无运行位置，就会使用起始位置进行初始化，第一次客户端反馈了ack信号后，就会记录运行位置.   </p>
<p>所以重置位置的几步操作：</p>
<ol>
<li><span style="line-height: 21px;">删除运行位置.    (pipeline同步进度页面配置)<br><img alt="" height="260" src="http://dl2.iteye.com/upload/attachment/0090/1630/6b9b59ef-4175-3e9b-ad35-d73a0d7088fb.png" width="508"><br><br></span></li>
<li><span style="line-height: 21px;">配置起始位置.   (canal配置页面)<br><img alt="" height="183" src="http://dl2.iteye.com/upload/attachment/0090/1636/9f8ec0b3-51e9-321f-a5b7-725792d06f31.png" width="516"></span></li>
<li><span style="line-height: 21px;">检查是否生效.  (pipeline对应的日志)<span style="line-height: 21px;"><br><img alt="" height="97" src="http://dl2.iteye.com/upload/attachment/0090/1626/a1ee7cc6-775d-37e3-9ef4-edcea30c6460.png" width="739"><br></span></span></li>
</ol>
<p>注意点：</p>
<ul>
<li>如果日志中出现prepare to find start position just last position. 即代表是基于运行位置启动，新设置的位点并没有生效。 (此时需要检查下位点删除是否成功 或者 canal是否被多个pipeline引用，导致位点删除后，被另一个pipeline重新写入，导致新设置的位点没有生效.)</li>
<li><strong><span style="color: red;">otter中使用canal，不允许pipeline共享一个canal.  otter中配置的canal即为一个instance，而otter就是为其一个client，目前canal不支持一个instance多个client的模式，会导致数据丢失，慎重. </span></strong></li>
</ul>
<h3>8.  日志列表中出现POSITIONTIMOUT后，数据进行重复同步？<br><img height="190" alt="" src="http://dl2.iteye.com/upload/attachment/0090/7324/a972d43f-700c-3a44-95e0-191bc9090645.jpg" width="741"><br> </h3>
<p> </p>
<p>答：</p>
<p>首先需要理解下Position超时监控，该监控主要是监控当前同步进度中的位点的最后更新时间和当前时间的差值，简单点说就是看你同步进度的位点多少时间没有更新了，超过阀值后触发该报警规则. </p>
<p>还有一点需要说明，同步进度中的位点只会记录一个事务的BEGIN/COMMIT位置，保证事务处理的完整性，不会记录事务中的中间位置. </p>
<p>几种情况下会出现同步进度位点长时间无更新：</p>
<ol>
<li>源库出现大事务，比如进行load data/ delete * from xxx，同时操作几百万/千万的数据，同步该事务数据时，位点信息一直不会被更新，比如默认超过10分钟后，就会触发Position超时监控，此时就是一个误判，触发自动恢复，又进入重新同步，然后进入死循环。</li>
<li>otter系统未知bug，导致系统的调度算法出现死锁，无法正常的同步数据，导致同步进度无法更新，触发该Position超时监控。此时：自动恢复的一次停用+启用同步任务，就可以恢复正常. <br>ps.  该Position超时监控，可以说是主要做为一种otter的系统保险机制，可以平衡一下，如果误判的影响&gt;系统bug触发的概率，可以考虑关闭Position超时监控，一般超时监控也会发出报警. </li>
</ol>
<p> </p>
<h3>9.  日志列表中出现miss data with keys异常，同步出现挂起后又自动恢复？</h3>
<p>异常信息：</p>
<p> </p>
<pre class="java" name="code">pid:2 nid:2 exception:setl:load miss data with keys:[MemoryPipeKey[identity=Identity[channelId=1,pipelineId=2,processId=4991953],time=1383190001987,dataType=DB_BATCH]]</pre>
</div>
<div class="iteye-blog-content-contain">答：</div>
<div class="iteye-blog-content-contain">   要理解该异常，需要先了解一下[[otter调度模型]]，里面SEDA中多个stage之间通过pipe进行数据交互，比如T模块完成后会将数据存到pipe中，然后通知SEDA中心，中心会通知L模块起来工作，L模块就会根据T传给中心的pipeKey去提取数据，而该异常就是当L模块根据pipeKey去提取数据时，发现数据没了。 主要原因：pipe在设计时，如果是单机传输时，会使用softReference来存储，所以当jvm内存不足时就会被GC掉，所以就会出现无数据的情况.   </div>
<div class="iteye-blog-content-contain">  ps. 如果miss data with keys异常非常多的时候，你就得考虑是否当前node已经超负载运行，内存不够，需要将上面的部分同步任务迁移出去。如果是偶尔的异常，那可以忽略，该异常会有自动恢复RESTART同步任务的处理。</div>
<div class="iteye-blog-content-contain">
<h3>10.  日志列表中出现manager异常？</h3>
<p>异常信息：</p>
<pre class="java" name="code">pid:2 nid:null exception:channel:can't restart by no select live node</pre>
<p>该异常代表pipelineId = 2，select模块的node没有可用的节点. </p>
<p> </p>
<p>异常信息：</p>
<pre class="java" name="code">pid:-1 nid:null exception:cid:2 restart recovery successful for rid:-1</pre>
<p>该异常代表channelId = 2，成功发起了一次restart同步任务的操作. </p>
<p> </p>
<p>异常信息：</p>
<pre class="java" name="code">pid:-1 nid:null exception:nid:2 is dead and restart cids:[1,2]</pre>
<p>该异常代表node id = 2，因为该node挂了，触发了channelId = 1 / 2的两个同步任务发起restart同步任务的操作.  (一种failover的机制)</p>
</div>