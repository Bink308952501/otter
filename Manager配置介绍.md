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

<h1>具体参数详解</h1>
<h3>channel参数</h3>
待补充
<h3>pipeline参数</h3>
待补充
<h3>canal参数</h3>
待补充
<h3>Node参数</h3>
待补充
<h3>Zookeeper集群参数</h3>
待补充
<h3>主备配置参数</h3>
待补充
<h3>数据源参数</h3>
待补充
<h3>数据表参数</h3>
待补充