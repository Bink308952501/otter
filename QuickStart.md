 <div class="blog_content">
    <div style="font-size: 14px;" class="iteye-blog-content-contain">
<h1>几点说明</h1>
<p>     otter依赖于<a href="https://github.com/alibaba/canal">canal</a>提供数据库日志，针对mysql数据有一些要求，具体请查看： <a style="font-size: 12px; line-height: 1.5;" href="https://github.com/alibaba/canal/wiki/QuickStart">https://github.com/alibaba/canal/wiki/QuickStart</a> </p>
<p>     有一点特别注意：目前canal支持mixed,row,statement多种日志协议的解析，<strong>但配合otter进行数据库同步，目前仅支持row协议的同步，使用时需要注意. </strong></p>
<p> </p>
<p><strong style="font-size: 2em; line-height: 1.5em;">环境准备</strong></p>
<p>1.  操作系统</p>
<p>&nbsp;&nbsp;&nbsp;     a.  otter为纯java编写，windows/linux均可支持</p>
<p>&nbsp;&nbsp;&nbsp;     b. jdk建议使用1.6.25以上的版本，稳定可靠，目前阿里巴巴使用基本为此版本</p>
<p> </p>
<p>2.  整个otter同步由几部分组成，需要预先进行安装，后续会有专门的篇幅展开介绍</p>
<ul>
<li>manager </li>
<li>node</li>
</ul>
<p>3.  otter node依赖于zookeeper进行分布式调度，需要安装一个zookeeper节点或者集群.  </p>
<p> &nbsp;&nbsp;&nbsp;    重要：考虑异地机房的地域性，node机器会优先选择就近的zookeeper节点进行访问，比如国际站会在杭州和美国各部署node，针对美国的node会选择美国的zookeeper进行访问，提升读效率. ps.  不同机房的zookeeper集群组成一个物理大集群，只不过是根据地域不同划分为不同逻辑集群，所有地域的node机器对zookeeper进行写操作都会发到一个地域的zookeeper进行paoxs算法仲裁. </p>
<p> &nbsp;&nbsp;&nbsp;    所以，manager启动完成后，需要首先定义不同机房的zookeeper机器集群。</p>
<p> </p>
<p> &nbsp;&nbsp;&nbsp;    比如：目前otter使用的zookeeper集群，在杭州会有3个机房，分别部署3+2+2台机器组成一个leader/follower集群，在美国一个机房部署2台机器，做为杭州机房zookeeper的observer(读镜像)，(observer模式特点：读请求在自己本地，写请求代理到leader/follower上投票处理，然后异步接收leader的写结果反馈. )</p>
<h1><img src="http://dl2.iteye.com/upload/attachment/0088/3018/78d22c08-f1e5-3f2a-862e-ce8ebf3db6d3.png" height="360" width="599" alt=""></h1>
<p>说明：添加了zookeeper集群后，会在添加node和canal时自动出现在可选列表里。 这也是添加node和canal的一个前提</p>
<p> </p>
<h1>环境安装</h1>
<h3>1.  manager安装</h3>
<p>      Otter Manager QuickStart： [[Manager_Quickstart]]
<p> </p>
<h3>2.  node安装</h3>
<p>      Otter Node QuickStart : [[Node_Quickstart]]
<p> </p>
<h1>操作演示</h1>
<h3> 演示视频（5分钟教你配置一个同步任务）：请点击图片或者<font color="red"><a href="http://www.tudou.com/programs/view/Q-qnCg7d-ew">这里</a></font> </p></h3>
[![ScreenShot](http://dl2.iteye.com/upload/attachment/0088/3012/4409999b-486f-36d7-a425-962b941b3b15.jpg)](http://www.tudou.com/programs/view/Q-qnCg7d-ew)
----
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
</div>
  </div>
</div>