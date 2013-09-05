 <div class="blog_content">
    <div style="font-size: 14px;" class="iteye-blog-content-contain">
<h1>环境准备</h1>
<p>1.  otter node会受otter manager进行管理，所以需要预先安装otter manager，参见：Otter Manager Quickstart. </p>
<p>2.  完成manager安装后，需要在manager页面为node定义配置信息，并生一个唯一id. </p>
<p>&nbsp;&nbsp;&nbsp;   a.   首先访问manager页面的机器管理页面，点击添加机器按钮</p>
<p><img width="460" height="393" alt="" src="http://dl2.iteye.com/upload/attachment/0088/1859/f9420edd-2cd9-33a3-ae16-24139c4004af.png"><br>    几点说明： </p>
<ul>
<li>机器名称：可以随意定义，方便自己记忆即可</li>
<li>机器ip：对应node节点将要部署的机器ip，如果有多ip时，可选择其中一个ip进行暴露.  (此ip是整个集群通讯的入口，实际情况千万别使用127.0.0.1，否则多个机器的node节点会无法识别)</li>
<li>机器端口：对应node节点将要部署时启动的数据通讯端口，建议值：2088  </li>
<li>下载端口：对应node节点将要部署时启动的数据下载端口，建议值：9090</li>
<li>外部ip ：对应node节点将要部署的机器ip，存在的一个外部ip，允许通讯的时候走公网处理。</li>
<li>zookeeper集群：为提升通讯效率，不同机房的机器可选择就近的zookeeper集群.  </li>
</ul>
<p>   node这种设计，是为解决单机部署多实例而设计的，允许单机多node指定不同的端口</p>
<p> </p>
<p>&nbsp;&nbsp;&nbsp;   b.  机器添加完成后，跳转到机器列表页面，获取对应的机器序号nid</p>
<p><img width="695" height="120" alt="" src="http://dl2.iteye.com/upload/attachment/0088/1861/06e5d134-4576-3a52-bbc8-8423173603ec.png"><br> </p>
<p> </p>
<p>    <strong>通过这两部操作，获取到了node节点对应的唯一标示，称之为node id，简称：nid.  记录该nid，后续启动nid时会使用</strong></p>
<p> </p>
<p><strong>   </strong><span style="line-height: 1.5;">3.  node节点进行跨机房传输时，会使用到HTTP多线程传输技术，目前主要依赖了aria2c做为其下载客户端，后续会推出java版本.   </span></p>
<p><span style="line-height: 1.5;">&nbsp;&nbsp;&nbsp;       a.  aria2 官方首页： </span><a style="line-height: 1.5;" href="http://aria2.sourceforge.net/">http://aria2.sourceforge.net/</a></p>
<p>&nbsp;&nbsp;&nbsp;       b.  下载页面： <a style="line-height: 1.5;" href="http://sourceforge.net/projects/aria2/files/stable/">http://sourceforge.net/projects/aria2/files/stable/</a></p>
<p>      当前测试过多个HTTP多线程下载客户端，比如wget,curl,axel,oget,proz,aria2c，测试结果<span style="line-height: 1.5;">aria2c下载效率最快，基本可以压满网卡.  </span></p>
<p><span style="line-height: 1.5;">      注意：下载完成或者编译完成后，将对应的aria2c包加入到PATH路径即可.  </span></p>
<p> </p>
<h1>启动步骤</h1>
<p>1.  下载otter node</p>
<p>    &nbsp;&nbsp;&nbsp;直接下载 ，可访问：https://github.com/alibaba/otter/releases ，会列出所有历史的发布版本包下载方式，比如以x.y.z版本为例子：</p>
<pre class="java" name="code">wget https://github.com/alibaba/otter/releases/download/otter-x.y.z/node.deployer-x.y.z.tar.gz</pre>
<p> or </p>
<p>   &nbsp;&nbsp;&nbsp;自己编译</p>
<pre class="java" name="code">git clone git@github.com:alibaba/otter.git  
cd otter;   
mvn clean install -Dmaven.test.skip -Denv=release  </pre>
<p> 编译完成后，会在根目录下产生target/node.deployer-$version.tar.gz</p>
<p> </p>
<p>2.  解压缩</p>
<pre class="java" name="code">mkdir /tmp/node
tar zxvf node.deployer-$version.tar.gz  -C /tmp/node  </pre>
<p> </p>
<p>3.  配置修改</p>
<p> &nbsp;&nbsp;&nbsp;   a.  nid配置  (将环境准备中添加机器后获取到的序号，保存到conf目录下的nid文件，比如我添加的机器对应序号为1)</p>
<pre class="java" name="code">echo 1 &gt; conf/nid</pre>
<p> &nbsp;&nbsp;&nbsp;   b.  otter.properties配置修改</p>
<p>    </p>
<pre class="java" name="code"># otter node root dir
otter.nodeHome = ${user.dir}/../node 
## otter node dir
otter.htdocs.dir = ${otter.nodeHome}/htdocs
otter.download.dir = ${otter.nodeHome}/download
otter.extend.dir= ${otter.nodeHome}/extend

## default zookeeper sesstion timeout = 90s
otter.zookeeper.sessionTimeout = 90000

## otter communication pool size
otter.communication.pool.size = 10

## otter arbitrate &amp; node connect manager config ， 修改为正确的manager服务地址
otter.manager.address = 127.0.0.1:1099
</pre>
<p> </p>
<p>4.  准备启动</p>
<pre class="java" name="code">sh startup.sh</pre>
<p> </p>
<p>5.  查看日志  </p>
<p>    如果manager页面的ip配置不正确，会出现类似错误：</p>
<p>    打开日志： vi logs/node/node.log </p>
<pre class="java" name="code">Exception in thread "main" java.lang.IllegalArgumentException: node[1] ip[127.0.0.1] port[2088] , but your host ip[10.12.48.215] is not matched!
        at com.alibaba.otter.node.etl.OtterController.checkNidVaild(OtterController.java:245)
        at com.alibaba.otter.node.etl.OtterController.initNid(OtterController.java:230)
        at com.alibaba.otter.node.etl.OtterController.start(OtterController.java:73)
        at com.alibaba.otter.node.deployer.OtterLauncher.main(OtterLauncher.java:25)</pre>
<p>    此时修改ip为对应的host ip后，再次启动即可。 </p>
<pre class="java" name="code">vi logs/node/node.log</pre>
<pre class="java" name="code">2013-08-14 15:42:16.886 [main] INFO  com.alibaba.otter.node.deployer.OtterLauncher - INFO ## the otter server is running now ......</pre>
<p>    看到如下日志，代表node启动完成.  </p>
<p> </p>
<p>6.  验证</p>
<p>  访问： http://127.0.0.1:8080/node_list.htm，查看对应的节点状态，如果变为了已启动，代表已经正常启动。(ps，如果是未启动，会是一个红色高亮)</p>
<p><img width="693" height="242" alt="" src="http://dl2.iteye.com/upload/attachment/0088/1904/fae149d6-8790-3a3b-af58-2981c8784c4e.png"></p>
<p> </p>
<p>7.  关闭 </p>
<pre class="java" name="code">sh stop.sh</pre>
<p>    关闭后，可查看下manager页面，检查下节点状态. </p>
<p> </p>
<p>it's  over. <br> </p>
</div>