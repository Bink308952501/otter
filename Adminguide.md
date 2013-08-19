 <div class="blog_content">
    <div class="iteye-blog-content-contain" style="font-size: 14px;">
<h1>几点说明</h1>
<p>otter系统自带了manager，所以简化了一些admin管理上的操作成本，比如可以通过manager发布同步任务配置，接收同步任务反馈的状态信息等。</p>
<p>目前manager的操作可分为两部分：</p>
<ul>
<li>同步配置管理<br>1.  添加数据源<br>2.  canal解析配置<br>3.  添加数据表<br>4.  同步任务</li>
<li>同步状态查询<br>1.  查询延迟<br>2.  查询吞吐量<br>3.  查询同步进度<br>4.  查询报警&amp;异常日志</li>
</ul>
<p>manager的用户权限在设计的时候，主要分为三类：</p>
<ul>
<li>ADMIN :  超级管理员</li>
<li>OPERATOR : 普通用户，管理某个同步任务下的同步配置，添加数据表，修改canal配置等</li>
<li>ANONYMOUS :  匿名用户，只能进行同步状态查询的操作. </li>
</ul>
<p> </p>
<h1>同步配置管理</h1>
<p>see the page for 配置： [[Manager配置介绍]]</p>
<p> </p>
<h1>同步状态查询</h1>
<p>see the page for 查询： [[Manager使用介绍]]</p>
</div>
    

  </div>
</div>