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
<p>3.  otter manager依赖于zookeeper进行管理多个node节点之间的协作，需要安装一个zookeeper节点或者集群. </p>
<p> </p>
<h1>环境安装</h1>
<h3>1.  manager安装</h3>
<p>      Otter Manager QuickStart： [[Manager_Quickstart]]
<p> </p>
<h3>2.  node安装</h3>
<p>      Otter Node QuickStart : [[Node_Quickstart]]
<p> </p>
<h1>操作演示</h1>
<p>     操作步骤：</p>
<p>&nbsp;&nbsp;&nbsp;        1.  添加canal</p>
<p> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            a.  提供数据库ip信息</p>
<p>&nbsp;&nbsp;&nbsp;        2.  添加数据库 </p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;             a.  源库</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;             b.  目标库</p>
<p>&nbsp;&nbsp;&nbsp;        2.  添加channel</p>
<p>&nbsp;&nbsp;&nbsp;        3.  添加pipeline </p>
<p> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            a.  选择node节点</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;             b.  选择canal</p>
<p>&nbsp;&nbsp;&nbsp;        4.  添加同步映射规则 </p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;             a.  定义源表和目标表的同步关系<span style="font-size: 12px; line-height: 1.5;"><br></span></p>
<p>&nbsp;&nbsp;&nbsp;        5.  添加监控</p>
<p>&nbsp;&nbsp;&nbsp;        6.  启动</p>
<p>     演示视频： 待完善.  </p>
</div>
    

  </div>
</div>