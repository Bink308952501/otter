<div class="blog_content">
    <div class="iteye-blog-content-contain">
<h1><strong>环境准备</strong></h1>
<p>1.  otter manager依赖于mysql进行配置信息的存储，所以需要预先安装mysql，并初始化otter manager的系统表结构</p>
<p>&nbsp;&nbsp;&nbsp;     a.  安装mysql，这里不展开，网上一搜一大把</p>
<p>&nbsp;&nbsp;&nbsp;     b.  初始化otter manager系统表： </p>
下载：<pre name="code" class="java">wget https://raw.github.com/alibaba/otter/master/manager/deployer/src/main/resources/sql/otter-manager-schema.sql </pre>
载入：<pre name="code" class="java">source otter-manager-schema.sql</pre>
<p style="font-size: 14px;"> </p>
<p style="font-size: 14px;">2.  整个otter架构依赖了zookeeper进行多节点调度，所以需要预先安装zookeeper，不需要初始化节点，otter程序启动后会自检. </p>
<p style="font-size: 14px;">&nbsp;&nbsp;&nbsp;     a.  manager需要在otter.properties中指定一个就近的zookeeper集群机器</p>
<p style="font-size: 14px;"> </p>
<h1><strong>启动步骤</strong></h1>
<p style="font-size: 14px;"><strong>    </strong>1.  下载otter manager</p>
<p style="font-size: 14px;">     直接下载 ，可访问：https://github.com/agapple/otter_download/ ，会列出所有历史的发布版本包下载方式，比如以x.y.z版本为例子：</p>
<pre name="code" class="shello">wget https://raw.github.com/agapple/otter_download/master/manager.deployer-x.y.z.tar.gz</pre>
    or
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">    自己编译</p>
<pre name="code" class="shell">git clone git@github.com:alibaba/otter.git
cd otter; 
mvn clean install -Dmaven.test.skip -Denv=release</pre>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">    编译完成后，会在根目录下产生target/manager.deployer-$version.tar.gz</p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;"> </p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">   2. 解压缩</p>
<pre name="code" class="java">mkdir /tmp/manager
tar zxvf manager.deployer-$version.tar.gz  -C /tmp/manager</pre>
<p style="font-size: 14px;"> </p>
<p style="font-size: 14px;">   3. 配置修改</p>
<p style="font-size: 14px;"> </p>
<pre name="code" class="html">## otter manager domain name #修改为正确访问ip，生成URL使用
otter.domainName = 127.0.0.1    
## otter manager http port
otter.port = 8080
## jetty web config xml
otter.jetty = jetty.xml

## otter manager database config   #修改为正确数据库信息
otter.database.driver.class.name = com.mysql.jdbc.Driver
otter.database.driver.url = jdbc:mysql://127.0.01:3306/ottermanager
otter.database.driver.username = root
otter.database.driver.password = hello

## otter communication port
otter.communication.manager.port = 1099

## otter communication pool size
otter.communication.pool.size = 10

## default zookeeper address
otter.zookeeper.cluster.default = 127.0.0.1:2181 #修改为正确的地址，手动选择一个地域就近的zookeeper集群列表
## default zookeeper sesstion timeout = 90s
otter.zookeeper.sessionTimeout = 90000

## otter arbitrate connect manager config
otter.manager.address = ${otter.domainName}:${otter.communication.manager.port}
</pre>
   
<p> </p>
<p style="font-size: 14px;">   4.  准备启动</p>
<p style="font-size: 14px;"> </p>
<pre name="code" class="java">sh startup.sh</pre>
  
<p> </p>
<p style="font-size: 14px;">   5. 查看日志</p>
<p style="font-size: 14px;"> </p>
<pre name="code" class="java">vi logs/webx.log</pre>
<pre name="code" class="java">2013-08-14 13:19:45.911 [] WARN  com.alibaba.otter.manager.deployer.JettyEmbedServer - ##Jetty Embed Server is startup!
2013-08-14 13:19:45.911 [] WARN  com.alibaba.otter.manager.deployer.OtterManagerLauncher - ## the manager server is running now ......</pre>
    出现类似日志，代表启动成功
<p> </p>
<p style="font-size: 14px;">   </p>
<p style="font-size: 14px;">   6.   验证</p>
<p style="font-size: 14px;">         访问： http://127.0.0.1:8080/，出现otter的页面，即代表启动成功     </p>
<p style="font-size: 14px;"><br><img src="http://dl2.iteye.com/upload/attachment/0088/1833/d81cd060-546c-312e-9a64-82ebd35f4f33.png" alt=""><br>    7.   关闭</p>
<pre name="code" class="java">sh stop.sh</pre>
</div>
<div class="iteye-blog-content-contain">    it's over.   </div>
  </div>
</div>