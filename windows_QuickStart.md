### # 网友:逆寻思念，提供windows安装过程

<div class="blog_content">
    <div style="font-size: 14px;" class="iteye-blog-content-contain">
<p>局域网中使用数据库同步</p>
<p>将ip:192.168.0.174中的数据单项同步到ip:192.168.0.25</p>
<p>服务器地址为192.168.0.25</p>
<p>最低运行环境</p>
<p>&nbsp;&nbsp;&nbsp;    Java  1.6.0 +     低于此版本导致无法开启-server命令</p>
<p>&nbsp;&nbsp;&nbsp;    MySQL 5.1.5 +     低于此版本导致无法使用binlog_format</p>
<p> </p>
<p>1. 安装 zookeeper-3.4.5.tar.gz</p>
<p>&nbsp;&nbsp;&nbsp;    1 &gt; C:\syn\zookeeper-3.4.5\conf\zoo_sample.cfg 复制并重命名为zoo.cfg</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 dataDir=C:/syn/zookeeper-3.4.5/data</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        添加 server.1=127.0.0.1:2887:3887</p>
<p>&nbsp;&nbsp;&nbsp;    2 &gt; C:\syn\zookeeper-3.4.5\data</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        创建本目录并创建新文件myid内容为1</p>
<p>&nbsp;&nbsp;&nbsp;    3 &gt; 命令行执行</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        C:\syn\zookeeper-3.4.5\bin\zkServer.cmd</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        C:\syn\zookeeper-3.4.5\bin\zkCli.cmd -server 127.0.0.1:2181</p>
<p>2. 安装 manager.deployer-4.2.2.tar.gz</p>
<p>&nbsp;&nbsp;&nbsp;    1 &gt; 导入数据库文件</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        C:\syn\otter-manager-schema.sql</p>
<p>&nbsp;&nbsp;&nbsp;    2 &gt; C:\syn\manager.deployer-4.2.2\conf\otter.properties</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 otter.domainName = 192.168.0.25</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 otter.database.driver.url = jdbc:mysql://127.0.0.1:3306/otter?useUnicode=true&amp;characterEncoding=utf8</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 otter.zookeeper.sessionTimeout = 90000</p>
<p>&nbsp;&nbsp;&nbsp;    3 &gt; 命令行执行</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        C:\syn\manager.deployer-4.2.2\bin\startup.bat</p>
<p>&nbsp;&nbsp;&nbsp;    4 &gt; 访问192.168.0.25:8080</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        添加Zookeeper集群 192.168.0.25:2181;</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        添加Node管理 192.168.0.25 2088 9090</p>
<p>3. 安装 aria2-1.17.1-win-32bit-build1.zip</p>
<p>&nbsp;&nbsp;&nbsp;    1 &gt; 配置系统运行环境</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        Path=C:\syn\aria2-1.17.1-win-32bit-build1</p>
<p>4. 安装 node.deployer-4.2.2.tar.gz</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    1 &gt; C:\syn\node.deployer-4.2.2\conf</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        创建新文件nid内容为1</p>
<p>&nbsp;&nbsp;&nbsp;    2 &gt; C:\syn\node.deployer-4.2.2\conf\otter.properties</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 otter.zookeeper.sessionTimeout = 90000</p>
<p>&nbsp;&nbsp;&nbsp;    3 &gt; 命令行执行</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        C:\syn\node.deployer-4.2.2\bin\startup.bat</p>
<p>5. 启用 数据库里row level的复制</p>
<p>&nbsp;&nbsp;&nbsp;    1 &gt; C:\syn\mysql\my.ini</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 log-bin=mysql-bin</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 binlog_format=ROW</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        修改 default-character-set=utf8</p>
<p> &nbsp;&nbsp;&nbsp;   2 &gt; 测试</p>
<p>&nbsp;&nbsp;&nbsp;        show variables like '%binlog_format%';</p>
<p>6. 配置同步数据库任务</p>
<p>&nbsp;&nbsp;&nbsp;    1 &gt; 导入测试sql表</p>
<p>        CREATE TABLE `test`.`example` (</p>
<p>          `id` INT(11)  NOT NULL AUTO_INCREMENT,</p>
<p>          `name` VARCHAR(32) COLLATE utf8_bin DEFAULT NULL ,</p>
<p>           PRIMARY KEY (`ID`)</p>
<p>        ) ENGINE=INNODB DEFAULT CHARSET=utf8;</p>
<p>&nbsp;&nbsp;&nbsp;    2 &gt; 访问192.168.0.25:8080</p>
<p>&nbsp;&nbsp;&nbsp;        数据源配置</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            src_174 jdbc:mysql://192.168.0.174:3306</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            dest_25 jdbc:mysql://192.168.0.25:3306</p>
<p>&nbsp;&nbsp;&nbsp;        Canal配置</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            test_canal 192.168.0.25:3306;</p>
<p>&nbsp;&nbsp;&nbsp;        数据表管理</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            src_174 test example</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            dest_25 test example</p>
<p>&nbsp;&nbsp;&nbsp;        Channel管理</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            example</p>
<p>&nbsp;&nbsp;&nbsp;        Pipeline管理</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            略</p>
<p>&nbsp;&nbsp;&nbsp;        映射关系列表</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            略</p>
<p>&nbsp;&nbsp;&nbsp;        启动</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            insert into test.example(id,name) values(null,'hello');</p>
</div>