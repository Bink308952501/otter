 <div class="blog_content">
    <div style="font-size: 14px;" class="iteye-blog-content-contain">
<h1> 背景</h1>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">otter4.0发布至今也差不多有近一年的时间，中间过程有着比较曲折经历，拥抱了许多变化，目前otter4已经在逐步替换otter3，继续服务icbu的相关中美同步业务，otter3即将成为过去式。</p>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"> </p>
<h1 style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">otter4管理系统(manager)</h1>
<p style="background-image: none; margin-top: 10px; margin-bottom: 10px;">比如我们内部系统使用了otter.alibaba-inc.com的域名，后续文档描述的时候会基于此域名链接</p>
<p> </p>
<h3>系统登录 </h3>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">匿名访问时只拥有同步状态的查询权限，可考虑与自己用户授权管理.</p>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"> </p>
<h3 style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">同步管理</h3>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">  <img width="718" src="http://dl2.iteye.com/upload/attachment/0088/2579/ae92dd57-891a-3734-a879-246d2bf385d7.png" height="413" alt=""><br> </p>
<h3 style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">同步模式配置 (点击同步列表右边的查看/编辑链接)</h3>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><img src="http://dl2.iteye.com/upload/attachment/0088/2603/35d8d83c-d322-345b-943f-ec6b564a3c47.png" alt=""><br>说明：</p>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">   a. 同步一致性</p>
<ol>
<li style="font-size: 14px; color: #000000; font-family: Helvetica, Tahoma, Arial, sans-serif;">基于数据库反查 (简单点说，就是强制反查数据库，从binlog中拿到pk，直接反查对应数据库记录进行同步，回退到几天前binlog进行消费时避免同步老版本的数据时可采用)</li>
<li style="font-size: 14px; color: #000000; font-family: Helvetica, Tahoma, Arial, sans-serif;">基于当前日志变更 (基于binlog/redolog解析出来的字段变更值进行同步，不做数据库反查，推荐使用)</li>
</ol>
<p> </p>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">   b. 同步模式</p>
<ol>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif;"><span style="line-height: 21px;">行模式 (兼容otter3的处理方案，改变记录中的任何一个字段，触发整行记录的数据同步，在目标库执行merge sql)</span></span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif;"><span style="line-height: 21px;">列模式 (基于log中的具体变更字段，按需同步)</span></span></li>
</ol>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">   c.  特殊组合： (同样支持)</p>
<ol>
<li>基于数据库反查+列模式</li>
<li>基于当前日志变更+行模式</li>
</ol>
<h3 style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">同步简要信息 (点击同步列表上的通道Channel名字的链接)</h3>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><img width="770" src="http://dl2.iteye.com/upload/attachment/0088/2581/269bea05-beb3-3567-be23-2312afe47b01.png" height="233" alt=""><br>说明：</p>
<ul>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">延迟时间 = 数据库同步到目标库成功时间 - 数据库源库产生变更时间， 单位秒. (由对应node节点定时推送配置)</span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">最后同步时间 = 数据库同步到目标库最近一次的成功时间 (当前同步关注的相关表，同步到目标库的最后一次成功时间)</span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">最后位点时间 = 数据binlog消费最后一次更新位点的时间 (和同步时间区别：一个数据库可能存在别的表的变更，不会触发同步时间变更，但会触发位点时间变更)</span></li>
</ul>
<ol style="font-size: 14px; line-height: 21px; color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<li style="font-size: 14px; line-height: 1.5; margin-bottom: 0px; margin-left: 0px;">
<li style="font-size: 14px; line-height: 1.5; margin-bottom: 0px; margin-left: 0px;">
<h3>同步详细信息 (点击同步简要信息上的Pipeline名字的链接)</h3>
</li>
</ol>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">目前主要分为几类:</p>
<ol>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">映射关系列表</span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">延迟时间</span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">同步进度</span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">监控管理</span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">日志记录</span></li>
</ol>
<h5 style="line-height: normal; margin-top: 1em; margin-bottom: 0.1em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E6%98%A0%E5%B0%84%E5%85%B3%E7%B3%BB%E5%88%97%E8%A1%A8"></a>映射关系列表</h5>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"> <br><img width="730" src="http://dl2.iteye.com/upload/attachment/0088/2583/54f0a764-dfa8-3d60-bab1-df2037b6acdc.png" height="153" alt=""><br> </p>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"> </p>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">点击查看打开映射关系信息页面：<br><br><img width="644" src="http://dl2.iteye.com/upload/attachment/0088/2605/db1f9ca4-2ad3-385f-acc5-ede11f8462b9.png" height="404" alt=""><br> <br>说明：</p>
<ul>
<li><span style="font-size: 1em; line-height: 1.5;">定义同步的源和目标的表信息 (注意:表明可以不同，可以定义数据库分库/分表)</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">Push权重 (对应的数字越大，同步会越后面得到同步，优先同步权重小的数据)</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">FileResolver (数据关联文件的解析类，目前支持动态源码推送,在目标jvm里动态编译生效，不再需要起停同步任务)</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">EventProcessor (业务自定义的数据处理类，比如可以定义不需要同步status='ENABLE'的记录或者根据业务改变同步的字段信息 简单业务扩展，otter4新特性)</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">字段同步 (定义源和目标的字段映射，字段名和字段类型均可进行映射定义，类似于数据库视图定义功能 otter4新特性)</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">组合同步 (字段组的概念，字段组中的一个字段发生变更，会确保字段组中的3个字段一起同步到目标库 otter4新特性)</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">多个字段决定一个图片地址，变更文件字段中的任何一个字段，就会触发FileResolver类解析，从而可以确保基于字段同步模式，也可以保证FileResolver能够正常解析出文件 otter4重要的优化)</span></li>
</ul>
<h5 style="line-height: normal; margin-top: 1em; margin-bottom: 0.1em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E5%90%9E%E5%90%90%E9%87%8F"></a>吞吐量</h5>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"> <img width="654" src="http://dl2.iteye.com/upload/attachment/0088/2585/73f4b409-328e-3856-885f-25dc744d3cfe.png" height="373" alt=""><br>说明：</p>
<ul>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">数据记录统计 (insert/update/delete的变更总和，不区分具体的表，按表纬度的数据统计，可查看映射关系列表-&gt;每个映射关系右边的行为曲线链接)</span></li>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">文件记录统计</span></li>
</ul>
<h5 style="line-height: normal; margin-top: 1em; margin-bottom: 0.1em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E5%BB%B6%E8%BF%9F%E6%97%B6%E9%97%B4"></a>延迟时间</h5>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><img width="610" src="http://dl2.iteye.com/upload/attachment/0088/2587/1a9ef455-caee-3870-93c6-3a9ef5fe0371.png" height="378" alt=""><br>说明：</p>
<ul>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">延迟时间的统计 = 数据库同步到目标库成功时间 - 数据库源库产生变更时间， 单位秒. (由对应node节点定时推送配置)</span></li>
</ul>
<h5 style="line-height: normal; margin-top: 1em; margin-bottom: 0.1em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E5%90%8C%E6%AD%A5%E8%BF%9B%E5%BA%A6"></a>同步进度</h5>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><img width="620" src="http://dl2.iteye.com/upload/attachment/0088/2589/3f87427e-d030-3394-a267-b7dde54cb55e.png" height="420" alt=""><br>说明：</p>
<ul>
<li><span style="line-height: 1.5; font-size: 12px;">mainstem状态： 代表canal模块当前的运行节点(也即是binlog解析的运行节点，解析会相对耗jvm内存)</span></li>
<li><span style="line-height: 1.5; font-size: 12px;">position状态： 当前同步成功的最后binlog位点信息 (包含链接的是数据库ip/port，对应binlog的位置，对应binlog的变更时间此时间即是计算延迟时间的源库变更时间)</span></li>
<li><span style="line-height: 1.5; font-size: 12px;">同步进度： 每个同步批次会有一个唯一标识，可根据该唯一标示进行数据定位，可以查看每个批次的运行时间，找出性能瓶颈点</span></li>
</ul>
<h5 style="line-height: normal; margin-top: 1em; margin-bottom: 0.1em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E7%9B%91%E6%8E%A7%E7%AE%A1%E7%90%86"></a>监控管理</h5>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><br><img width="831" src="http://dl2.iteye.com/upload/attachment/0088/2591/06d37075-2cdf-3623-b01f-47076b3fe4ee.png" height="272" alt=""><br> <br>说明：</p>
<ol style="font-size: 14px; line-height: 21px; color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<li style="font-size: 14px; line-height: 1.5; margin-bottom: 0px; margin-left: 0px;">监控项目</li>
</ol>
<p> </p>
<ul>
<li><span style="font-size: 1em; line-height: 1.5;">同步延迟，position超时(位点超过多少时间没有更新) ， 一般业务方关心这些即可</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">异常 (同步运行过程中出现的异常，比如oracle DBA关心oracle系统ORA-的异常信息，mysql DBA关心mysql数据库相关异常)</span></li>
<li><span style="font-size: 1em; line-height: 1.5;">process超时(一个批次数据执行超过多少时间)，同步时间超时(数据超过多少时间没有同步成功过)</span></li>
</ul>
<p> </p>
<ol style="font-size: 14px; line-height: 21px; color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<li style="font-size: 14px; line-height: 1.5; margin-bottom: 0px; margin-left: 0px;">阀值设置</li>
</ol>
<p> </p>
<ul>
<li><span style="font-size: 1em; line-height: 1.5;">1800@09:00-18:00 , 这例子是指定了早上9点到下午6点，报警阀值为1800.</span></li>
</ul>
<p> </p>
<ol style="font-size: 14px; line-height: 21px; color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<li style="font-size: 14px; line-height: 1.5; margin-bottom: 0px; margin-left: 0px;">发送对象</li>
</ol>
<ul>
<li><span style="color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif; font-size: 1em; line-height: 1.5;">otterteam为otter团队的标识，阿里内部使用了dragoon系统监控报警通知，如果外部系统可实现自己的报警通知机制</span></li>
</ul>
<h5 style="line-height: normal; margin-top: 1em; margin-bottom: 0.1em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E6%97%A5%E5%BF%97%E8%AE%B0%E5%BD%95"></a>日志记录</h5>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><br><img width="678" src="http://dl2.iteye.com/upload/attachment/0088/2593/9b94d8eb-37d2-3f95-b900-16b903802df9.png" height="360" alt=""><br> <br>说明：</p>
<ol style="font-size: 14px; line-height: 21px; color: #333333; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<li style="font-size: 14px; line-height: 1.5; margin-bottom: 0px; margin-left: 0px;">日志标题即为对应的监控规则定义的名字，可根据监控规则检索对应的日志记录</li>
<li style="font-size: 14px; line-height: 1.5; margin-bottom: 0px; margin-left: 0px;">日志内容即为发送报警的信息</li>
</ol>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">注意： otter4采用主动推送报警的模式，可以保证报警的及时性以及日志完整性(相比于日志文件扫描机制来说)</p>
<h3 style="line-height: normal; font-size: 1.4em; margin-top: 1.5em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86"></a>配置管理</h3>
<h4 style="line-height: normal; font-size: 1.2em; margin-top: 1.2em; margin-bottom: 0.3em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E6%95%B0%E6%8D%AE%E6%BA%90"></a>数据源</h4>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><br><img width="694" src="http://dl2.iteye.com/upload/attachment/0088/2595/21d92ead-950d-3b9a-bfd1-df35e3efd7f2.png" height="236" alt=""><br> <br>说明：</p>
<ul>
<li><span style="line-height: 1.5; font-size: 12px;">主要是数据库连接信息：定义字符编码，ip地址等 (类似napoli等存储也可以抽象为数据源进行配置)</span></li>
<li><span style="line-height: 1.5; font-size: 12px;">切换数据库时，可根据ip检索同步数据库，找到需要切换的同步任务</span></li>
</ul>
<h4 style="line-height: normal; font-size: 1.2em; margin-top: 1.2em; margin-bottom: 0.3em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E6%95%B0%E6%8D%AE%E8%A1%A8%26nbsp%3B"></a>数据表</h4>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><br><img width="731" src="http://dl2.iteye.com/upload/attachment/0088/2601/2f0f62b5-1447-31cc-b67e-9920bc12c2e6.png" height="334" alt=""><br> </p>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">说明：</p>
<ul>
<li><span style="line-height: 1.5; font-size: 12px;">数据表是一种抽象概念，(针对数据库类型即为一个数据库表的定义)</span></li>
</ul>
<h4 style="line-height: normal; font-size: 1.2em; margin-top: 1.2em; margin-bottom: 0.3em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-canal%E9%85%8D%E7%BD%AE%26nbsp%3B"></a>canal配置</h4>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;">说明：主要是管理canal链接到mysql/oracle获取日志的相关参数等，业务方可不重点关注</p>
<h4 style="line-height: normal; font-size: 1.2em; margin-top: 1.2em; margin-bottom: 0.3em; font-family: Arial, Helvetica, FreeSans, sans-serif;">
<a name="otter4%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E4%BB%8B%E7%BB%8D-%E6%9C%BA%E5%99%A8%E7%AE%A1%E7%90%86"></a>机器管理</h4>
<p style="color: #333333; background-image: none; margin-top: 10px; margin-bottom: 10px; font-family: Arial, Helvetica, FreeSans, sans-serif;"><img width="705" src="http://dl2.iteye.com/upload/attachment/0088/2599/2f885561-45bc-30b7-aef9-5c8f3f0783e2.png" height="344" alt=""><br> </p>
</div>