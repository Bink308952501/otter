<div class="blog_content">
    <div class="iteye-blog-content-contain" style="font-size: 14px;">
<h1>项目介绍</h1>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">名称：otter ['ɒtə(r)]</p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">译意： 水獭，数据搬运工</p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">语言： 纯java开发</p>
<p style="margin-top: 15px; margin-bottom: 15px; color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">定位： 基于数据库增量日志解析，准实时同步到本机房或跨机房的mysql/oracle数据库. </p>
<p> </p>
<h1>工作原理</h1>
<p><img width="848" src="http://dl2.iteye.com/upload/attachment/0088/1189/d420ca14-2d80-3d55-8081-b9083606a801.jpg" height="303" alt=""></p>
<p>原理描述：</p>
<p>1.   基于Canal开源产品，获取数据库增量日志数据。 什么是Canal,  请<a href="https://github.com/alibaba/canal">点击</a></p>
<p>2.   典型管理系统架构，manager(web管理)+node(工作节点)</p>
<p>&nbsp;&nbsp;&nbsp;     a.  manager运行时推送同步配置到node节点</p>
<p>&nbsp;&nbsp;&nbsp;     b.  node节点将同步状态反馈到manager上</p>
<p>3.  基于zookeeper，解决分布式状态调度的，允许多node节点之间协同工作. </p>
<p> </p>
<h1>QuickStart</h1>
<p>See the page for quick start: QuickStart.</p>
<p> </p>
<h1>AdminGuide</h1>
<p>See the page for admin deploy guide : AdminGuide</p>
<p> </p>
<h1>问题反馈</h1>
<p>1.  <span style="color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">qq交流群： 161559791</span></p>
<p><span style="color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">2.  </span><span style="color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">邮件交流： jianghang115@gmail.com</span></p>
<p><span style="color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">3.  </span><span style="color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">新浪微博： agapple0002</span></p>
<p><span style="color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">4.  </span><span style="color: #333333; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">报告issue：</span><a href="https://github.com/agapple/otter/issues" style="color: #4183c4; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">issues</a></p>
<p> </p>
</div>