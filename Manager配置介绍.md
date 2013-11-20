<h1>操作演示</h1>
<h3> 演示视频（5分钟教你配置一个同步任务）：请点击图片或者<font color="red"><a href="http://www.tudou.com/programs/view/Q-qnCg7d-ew">这里</a></font> </p></h3>
[![ScreenShot](http://dl2.iteye.com/upload/attachment/0088/3012/4409999b-486f-36d7-a425-962b941b3b15.jpg)](http://www.tudou.com/programs/view/Q-qnCg7d-ew)

<p>    演示说明：</p>
<p>&nbsp;&nbsp;&nbsp;1.  搭建一个数据库同步任务，源数据库ip为：10.20.144.25，目标数据库ip为：10.20.144.29.  源数据库已开启binlog，并且binlog_format为ROW. </p>
<pre class="java" name="code">mysql&gt; show variables like '%binlog_format%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+</pre>
<p>&nbsp;&nbsp;&nbsp;2.  数据同步精确到一张表进行测试，测试的表名为test.example，简单包含两个子段，测试过程中才创建. </p>
<p>&nbsp;&nbsp;&nbsp;3.  配置完成后，手动在源库插入数据，然后快速在目标库进行查看数据，验证数据是否同步成功. </p>
<p></p>
-------

视频中的演示文本：
<pre>
CREATE TABLE  `test`.`example` (
  `id` int(11)  NOT NULL AUTO_INCREMENT,
  `name` varchar(32) COLLATE utf8_bin DEFAULT NULL ,
   PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into test.example(id,name) values(null,'hello');

-----
Otter QuickStart 如何配置一个任务
-----
操作步骤：
1.  添加数据库
    a.  源库 jdbc:mysql://10.20.144.25:3306
    b.  目标库 jdbc:mysql://10.20.144.29:3306
2.  添加canal
    a.  提供数据库ip信息 
3.  添加同步表信息
    a.  源数据表 test.example
    b.  目标数据表 test.example
4.  添加channel
5.  添加pipeline
    a.  选择node节点
    b.  选择canal
6.  添加同步映射规则
    a.  定义源表和目标表的同步关系
7.  启动
8.  测试数据 
</pre>

<h1>通道配置说明</h1>
<h2>多种同步方式配置</h2>
<h3>a. 单向同步</h3>
<p style="font-size: 14px;">   单向同步为最基本的同步方式，目前支持mysql -&gt; mysql/oracle的同步.</p>
<p style="font-size: 14px;">   基本配置方式就如操作视频中所演示的，操作步骤：</p>
<ol style="font-size: 14px;">
<li>配置一个channel</li>
<li>配置一个pipeline<br>对应node机器选择时，建议遵循：S/E节点选择node需尽可能离源数据库近，T/L节点选择node则离目标数据库近.  如果无法提供双节点，则选择离目标数据库近的node节点相对合适.</li>
<li>配置一个canal</li>
<li>定义映射关系. <br>canal中配置解析的数据库ip需要和映射关系中源表对应的数据库ip一致.  ps. 映射关系进行匹配的时候是基于表名，虽然数据库ip不匹配也会是有效.  </li>
</ol>
<h3>b.  双向同步</h3>
<p style="font-size: 14px;">   双向同步可以理解为两个单向同步的组合，但需要额外处理避免回环同步.   回环同步算法： <a style="color: #4183c4; font-weight: bold; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 12px; line-height: 19px; background-color: #f7f7f7;" class="internal present" href="https://github.com/alibaba/otter/wiki/Otter%E5%8F%8C%E5%90%91%E5%9B%9E%E7%8E%AF%E6%8E%A7%E5%88%B6">Otter双向回环控制</a> . </p>
<p style="font-size: 14px;">   同时，因为双向回环控制算法会依赖一些系统表，需要在需要做双向同步的数据库上初始化所需的系统表.  </p>
<p style="font-size: 14px;">   获取初始sql: </p>
<pre name="code" class="java">wget https://raw.github.com/alibaba/otter/master/node/deployer/src/main/resources/sql/otter-system-ddl-mysql.sql</pre>
<p style="font-size: 14px;"> </p>
<p style="font-size: 14px;">  配置上相比于单向同步有一些不同，操作步骤：</p>
<ol style="font-size: 14px;">
<li>配置一个channel</li>
<li>配置两个pipeline <br>注意：两个单向的canal和映射配置，在一个channel下配置为两个pipeline.   如果是两个channel，每个channel一个pipeline，将不会使用双向回环控制算法，也就是会有重复回环同步. </li>
<li>每个pipeline各自配置canal，定义映射关系</li>
</ol>
<h3>c.  双A同步</h3>
<p style="font-size: 14px;">    双A同步相比于双向同步，主要区别是双A机房会在两地修改同一条记录，而双向同步只是两地的数据做互相同步，两地修改的数据内容无交集.  </p>
<p style="font-size: 14px;">    所以双A同步需要额外处理数据同步一致性问题.   同步一致性算法：<a style="color: #4183c4; font-weight: bold; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 12px; line-height: 19px; background-color: #f7f7f7;" class="internal present" href="https://github.com/alibaba/otter/wiki/Otter%E6%95%B0%E6%8D%AE%E4%B8%80%E8%87%B4%E6%80%A7">Otter数据一致性</a> ，目前开源版本主要是提供了单向回环补救的一致性方案.  </p>
<p style="font-size: 14px;">    双A同步相比于双向同步，整个配置主要是一些参数上有变化，具体步骤：</p>
<ol style="font-size: 14px;">
<li>配置一个channel <br><img height="305" src="http://dl2.iteye.com/upload/attachment/0089/3555/fcf1872f-2e1c-370a-aae5-b08bac7d557a.png" width="431" alt=""><br> </li>
<li>配置两个pipeline<br><img height="132" src="http://dl2.iteye.com/upload/attachment/0089/3557/07866a7c-ba20-3989-aa4b-b0d8a76e4b44.png" width="704" alt="">
</li>
<li>每个pipeline各自配置canal，定义映射关系 </li>
</ol>
<h3>d.  级联同步</h3>
<p style="font-size: 14px;">   单向同步/双向同步，都是针对一个channel下的多pipeline配置进行控制，是否可以使用多个channel完成类似级联同步的功能.  </p>
<p style="font-size: 14px;">   几种级联同步. </p>
<ul style="font-size: 14px;">
<li>A-&gt;B-&gt;C ，A单向同步到B，B再单向同步到C</li>
<li>A&lt;-&gt;B-&gt;C，A和B组成一个双向，B再单向同步到C</li>
<li>A&lt;-&gt;B&lt;-C，A和B组成一个双向，C将数据单向同步B，也就是B是一个接受多M同步写入的的节点，目前mysql不支持</li>
<li>A&lt;-&gt;B-&gt;C，B-/-&gt;D，A和B组成一个双向，B再单向同步到C，但A同步到B的数据不同步到D，但B地写入的数据同步到D，目前mysql不支持.  </li>
</ul>
<p style="font-size: 14px;">   对应操作步骤：    </p>
<ol style="font-size: 14px;">
<li>目前channel之间的级联同步，不需要设置任何参数，只要通过canal进行binlog解析即可. </li>
<li>针对级联屏蔽同步，需要利用到自定义同步标记的功能，比如A-&gt;B，B同步到C但不同步到D。需要在A-&gt;B的同步高级参数里定义NOT_DDD，然后在B同步到D的高级参数里也定义NOT_DDD. <br>原理：这样在B解析到A-&gt;B写入的同步标记为NOT_DDD，与当前同步定义的NOT_DDD进行匹配，就会忽略此同步. <br><img height="190" src="http://dl2.iteye.com/upload/attachment/0089/3565/e0582df5-6041-3693-b654-94870a088e50.png" width="366" alt=""><br><br>
</li>
</ol>
<h3>e.  多A同步</h3>
<p style="font-size: 14px;">  基于以上的单向/双向/双A/级联同步，可以随意搭建出多A同步，不过目前受限于同步数据的一致性算法，只能通过星形辐射，通过级联同步的方式保证全局多A机房的数据一致性.  <br>  比如图中B和C之前的一致性同步，需要通过主站点A来保证. </p>
<p style="font-size: 14px;"><img height="364" src="http://dl2.iteye.com/upload/attachment/0089/3572/57f45537-bb81-3388-8991-595ef7710fd5.jpg" width="470" alt=""><br>      </p>
<h2>自定义数据同步(自 由 门)</h2>
<p style="font-size: 14px;">    主要功能是在不修改原始表数据的前提下，触发一下数据表中的数据同步。 </p>
<p style="font-size: 14px;">    可用于： </p>
<ul style="font-size: 14px;">
<li>同步数据订正</li>
<li>全量数据同步.   (自 由 门触发全量，同时otter增量同步，需要配置为行记录模式，避免update时因目标库不存在记录而丢失update操作) </li>
</ul>
<p style="font-size: 14px;">    主要原理：</p>
<p style="font-size: 14px;">    a.   基于otter系统表retl_buffer，插入特定的数据，包含需要同步的表名，pk信息。</p>
<p style="font-size: 14px;">    b.   otter系统感知后会根据表名和pk提取对应的数据(整行记录)，和正常的增量同步一起同步到目标库。</p>
<p style="font-size: 14px;"> </p>
<p style="font-size: 14px;">    目前otter系统感知的自 由 门数据方式为：</p>
<ul style="font-size: 14px;">
<li>日志记录.  (插入表数据的每次变更，需要开启binlog，otter获取binlog数据，提取同步的表名，pk信息，然后回表查询整行记录)</li>
</ul>
<p style="font-size: 14px;">   retl_buffer表结构：</p>
<pre name="code" class="sql">  CREATE TABLE retl_buffer 
   (    
    ID BIGINT AUTO_INCREMENT,   ## 无意义，自增即可
    TABLE_ID INT(11) NOT NULL,   ## tableId, 可通过该链接查询：http://otter.alibaba-inc.com/data_media_list.htm，即序号这一列，如果配置的是正则，需要指定full_name，当前table_id设置为0. 
    FULL_NAME varchar(512),  ## schemaName + '.' +  tableName  (如果明确指定了table_id，可以不用指定full_name)
    TYPE CHAR(1) NOT NULL,   ## I/U/D ，分别对应于insert/update/delete
    PK_DATA VARCHAR(256) NOT NULL, ## 多个pk之间使用char(1)进行分隔
    GMT_CREATE TIMESTAMP NOT NULL, ## 无意义，系统时间即可
    GMT_MODIFIED TIMESTAMP NOT NULL,  ## 无意义，系统时间即可
    CONSTRAINT RETL_BUFFER_ID PRIMARY KEY (ID) 
   )  ENGINE=InnoDB DEFAULT CHARSET=utf8;</pre>
<p style="font-size: 14px;"> </p>
<p style="font-size: 14px;">    全量同步操作示例： </p>
<pre name="code" class="java">insert into retl.retl_buffer(ID,TABLE_ID, FULL_NAME,TYPE,PK_DATA,GMT_CREATE,GMT_MODIFIED) (select null,0,'$schema.table$','I',id,now(),now() from $schema.table$); </pre>
<p style="font-size: 14px;"> </p>
<h1>具体参数详解</h1>
<h2>channel参数</h2>
<ol>
<li style="font-size: 14px;">同步一致性.  ==&gt; 基于数据库反查(根据binlog反查数据库)，基于当前变更(binlog数据)。针对数据库反查，在延迟比较大时比较有效，可将最新的版本快速同步到目标，但会对源库有压力. </li>
<li style="font-size: 14px;">同步模式. ==&gt; 行模式，列模式。行模式特点：如果目标库不存在记录时，执行插入。列模式主要是变更哪个字段，只会单独修改该字段，在双Ａ同步时，为减少数据冲突，建议选择列模式。</li>
<li>是否开启数据一致性. ==&gt;　请查看数据一致性文档：<a style="color: #4183c4; font-weight: bold; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 12px; line-height: 19px; background-color: #f7f7f7;" class="internal present" href="https://github.com/alibaba/otter/wiki/Otter%E6%95%B0%E6%8D%AE%E4%B8%80%E8%87%B4%E6%80%A7">Otter数据一致性</a><br>a.  数据一致性算法<br>b.  一致性反查数据库延迟阀值</li>
</ol>
<h2>pipeline参数</h2>
<ol>
<li>并行度.  ==&gt;  查看文档：<a style="color: #4183c4; font-weight: bold; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 12px; line-height: 19px; background-color: #f7f7f7;" class="internal present" href="https://github.com/alibaba/otter/wiki/Otter%E8%B0%83%E5%BA%A6%E6%A8%A1%E5%9E%8B">Otter调度模型</a>，主要是并行化调度参数.(滑动窗口大小)</li>
<li><span style="line-height: 21px;">数据反查线程数. ==&gt;   如果选择了同步一致性为反查数据库，在反查数据库时的并发线程数大小</span></li>
<li>数据载入线程数.  ==&gt;  在目标库执行并行载入算法时并发线程数大小</li>
<li>文件载入线程数.  ==&gt;  数据带文件同步时处理的并发线程数大小</li>
<li>主站点.  ==&gt; 双Ａ同步中的主站点设置　</li>
<li>消费批次大小. ==&gt;  获取canal数据的batchSize参数</li>
<li>获取批次超时时间. ==&gt;  获取canal数据的timeout参数 </li>
</ol>pipeline 高级设置<br style="font-size: 1em;"><ul>
<li>使用batch. ==&gt;  是否使用jdbc batch提升效率，部分分布式数据库系统不一定支持batch协议</li>
<li>跳过load异常. ==&gt;  比如同步时出现目标库主键冲突，开启该参数后，可跳过数据库执行异常</li>
<li>仲裁器调度模式. ==&gt;  查看文档：<a style="color: #4183c4; font-weight: bold; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 12px; line-height: 19px; background-color: #f7f7f7;" class="internal present" href="https://github.com/alibaba/otter/wiki/Otter%E8%B0%83%E5%BA%A6%E6%A8%A1%E5%9E%8B">Otter调度模型</a>
</li>
<li>负载均衡算法. ==&gt;   查看文档：<a style="color: #4183c4; font-weight: bold; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 12px; line-height: 19px; background-color: #f7f7f7;" class="internal present" href="https://github.com/alibaba/otter/wiki/Otter%E8%B0%83%E5%BA%A6%E6%A8%A1%E5%9E%8B">Otter调度模型</a>
</li>
<li>传输模式.  ==&gt;  多个node节点之间的传输方式，RPC或HTTP.  HTTP主要就是使用aria2c，如果测试环境不装aria2c，可强制选择为RPC</li>
<li>记录selector日志. ==&gt;   是否记录简单的canal抓取binlog的情况</li>
<li>记录selector详细日志. ==&gt; 是否记录canal抓取binlog的数据详细内容</li>
<li>记录load日志. ==&gt;  是否记录otter同步数据详细内容</li>
<li>dryRun模式.  ==&gt; 只记录load日志，不执行真实同步到数据库的操作</li>
<li>支持ddl同步.  ==&gt;  是否同步ddl语句</li>
<li>是否跳过ddl异常.  ==&gt;  同步ddl出错时，是否自动跳过</li>
<li>文件重复同步对比  ==&gt;  数据带文件同步时，是否需要对比源和目标库的文件信息，如果文件无变化，则不同步，减少网络传输量. </li>
<li>文件传输加密 ==&gt;  基于HTTP协议传输时，对应文件数据是否需要做加密处理</li>
<li>启用公网同步 ==&gt;  每个node节点都会定义一个外部ip信息，如果启用公网同步，同步时数据传递会依赖外部ip.  </li>
<li>跳过自 由 门数据  ==&gt;  自定义数据同步的内容</li>
<li>跳过反查无记录数据  ==&gt;   反查记录不存在时，是否需要进行忽略处理，不建议开启. </li>
<li>启用数据表类型转化  ==&gt;  源库和目标库的字段类型不匹配时，开启改功能，可自动进行字段类型转化</li>
<li>兼容字段新增同步   ==&gt;  同步过程中，源库新增了一个字段(必须无默认值)，而目标库还未增加，是否需要兼容处理</li>
<li>自定义同步标记  ==&gt;  级联同步中屏蔽同步的功能. </li>
</ul>
<p> </p>
<h2>Canal参数</h2>
<ol>
<li>数据源信息<br>单库配置： 10.20.144.34:3306;<br>多库合并配置： 10.20.144.34:3306,10.20.144.35:3306;  (逗号分隔)<br>主备库配置：10.20.144.34:3306;10.20.144.34:3307;  (分号分隔)</li>
<li>数据库帐号</li>
<li>数据库密码</li>
<li>connectionCharset  ==&gt;  获取binlog时指定的编码</li>
<li>位点自定义设置  ==&gt;   格式：{"journalName":"","position":0,"timestamp":0};  <br>指定位置：{"journalName":"","position":0};  <br>指定时间：{"timestamp":0};  </li>
<li>内存存储batch获取模式　==&gt;  MEMSIZE/ITEMSIZE，前者为内存控制，后者为数量控制. 　针对MEMSIZE模式的内存大小计算 = 记录数 * 记录单元大小 </li>
<li><span style="line-height: 21px;">内存存储buffer记录数</span></li>
<li>内存存储buffer记录单元大小</li>
<li>HA机制 </li>
<li>心跳SQL配置 ==&gt; 可配置对应心跳SQL，如果配置 是否启用心跳HA，当心跳ＳＱＬ检测失败后，canal就会自动进行主备切换. </li>
</ol>
<h2>Node参数</h2>
<ol>
<li>机器名称 ==&gt; 自定义名称，方便记忆</li>
<li>机器ip ==&gt;  机器外部可访问的ip，不能选择127.0.0.1</li>
<li>机器端口 ==&gt;  和manager/node之间RPC通讯的端口</li>
<li>下载端口 ==&gt; 和node之间HTTP通讯的端口</li>
<li>外部Ip ==&gt; node机器可以指定多IP，通过pipeline配置决定是否启用</li>
<li>zookeeper集群 ==&gt; 就近选择zookeeper集群</li>
</ol>
<h2>Zookeeper集群参数</h2>
<ol>
<li>集群名字  ==&gt;   自定义名称，方便记忆</li>
<li>zookeeper集群 ==&gt;  zookeeper集群机器列表，逗号分隔，最后以分号结束</li>
</ol>
<h2>主备配置参数</h2>
<ol>
<li>group Key ==&gt; 自定义名称，otter其他地方基于该名称进行引用</li>
<li>master / slave ==&gt;  主备库ip信息</li>
</ol>
<p>生成了groupKey，1.  可以在数据库配置时，设置url：jdbc:mysql://groupKey=key (更改 key).    2. 在canal配置时，选择HA机制为media，可填入该groupKey进行引用</p>
<p> </p>