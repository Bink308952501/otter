<div class="blog_content">
    <div class="iteye-blog-content-contain">
<h1>扩展性定义</h1>
<p style="font-size: 14px;">按照实现不同，可分为两类：</p>
<ol style="font-size: 14px;">
<li>数据处理自定义，比如Extract , Transform的数据处理.  目前Select/Load不支持数据自定义处理</li>
<li>组件功能性扩展，比如支持oracle日志获取，支持hbase数据输出等. </li>
</ol>
<h3>数据处理自定义</h3>
<p style="font-size: 14px;">Extract模块：</p>
<ul style="font-size: 14px;">
<li>EventProcessor  :  自定义数据处理，可以改变一条变更数据的任意内容</li>
<li>FileResolver  :  解决数据和文件的关联关系</li>
</ul>
<p style="font-size: 14px;">目前两者都只支持java语言编写，但都支持运行时动态编译&amp;lib包载入的功能。</p>
<ol style="font-size: 14px;">
<li>通过Otter Manager直接发布source文件代码，然后推送到node节点上即时生效，不需要重启任何java进程，有点动态语言的味道</li>
<li>可以将class文件放置到extend目录或者打成jar包，放置在node启动classpath中，也可以通过Otter Manager指定类名的方式进行加载，这样允许业务完全自定义。(但有个缺点，如果使用了一些外部包加入到node classpath中，比如远程接口调用，目前EventProcessor的调用是串行处理，针对串行进行远程调用执行，效率会比较差. )</li>
</ol>
<p style="font-size: 14px;"><img height="359" alt="" width="551" src="http://dl2.iteye.com/upload/attachment/0088/3087/6697009d-a6b4-369f-a324-a6c850ce0a5d.png"><br> </p>
<p style="font-size: 14px;">EventProcessor接口示例：</p>
<pre name="code" class="java">/**
 * 业务自定义处理过程
 * 
 * @author jianghang 2012-6-25 下午02:26:36
 * @version 4.1.0
 */
public interface EventProcessor {

    /**
     * 自定义处理单条EventData对象
     * 
     * @return {@link EventData} 返回值=null，需要忽略该条数据
     */
    public EventData process(EventData eventData);
}</pre>
<h3>扩展代码开发</h3>
<h4>1.  自定义维护</h4>
<p>依赖配置：</p>
<pre name="code" class="java">&lt;dependency&gt;
	&lt;groupId&gt;com.alibaba.otter&lt;/groupId&gt;
	&lt;artifactId&gt;shared.etl&lt;/artifactId&gt;
	&lt;version&gt;x.y.z&lt;/version&gt;
&lt;/dependency&gt;
&lt;dependency&gt;
	&lt;groupId&gt;com.alibaba.otter&lt;/groupId&gt;
	&lt;artifactId&gt;node.extend&lt;/artifactId&gt;
	&lt;version&gt;x.y.z&lt;/version&gt;
&lt;/dependency&gt;</pre>
a.  创建mvn标准工程
<pre name="code" class="java">mvn archetype:create -DgroupId=com.alibaba.otter -DartifactId=node.code</pre>
b.  修改pom.xml，添加依赖
<p>c.  mvn eclipse:eclipse导入工程</p>
<p>d.  代码开发完成后，使用的有两种选择：1.  manager上选择source，直接粘帖源码 .   2. 将该代码打成jar包，放到node工程的lib目录下，在manager上选择clazz后，写上对应的类名即可</p>
<p> </p>
<h4>2.  基于otter.extend工程维护</h4>
<p>a. 下载otter源码包， <a href="https://github.com/alibaba/otter" style="font-size: 12px; line-height: 1.5;">https://github.com/alibaba/otter</a></p>
<p>b. 导入工程.  </p>
<pre name="code" class="java">mvn clean eclipse:eclipse install -Dmaven.test.skip</pre>
c.  代码开发完成后，使用的有两种选择：1.  manager上选择source，直接粘帖源码 .   2. 整个otter编译打包，发布后，在manager上选择clazz后，写上对应的类名即可
<p> </p>
<p> </p>
<h3>扩展示例代码：</h3>
<ul>
<li>EventProcessor扩展：<a href="https://github.com/alibaba/otter/blob/master/node/extend/src/main/java/com/alibaba/otter/node/extend/processor/TestEventProcessor.java" style="color: #4183c4; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">TestEventProcessor</a>
</li>
<li>FileResolver扩展：<a href="https://github.com/alibaba/otter/blob/master/node/extend/src/main/java/com/alibaba/otter/node/extend/fileresolver/TestFileResolver.java" style="color: #4183c4; font-family: Helvetica, arial, freesans, clean, sans-serif; font-size: 15px; line-height: 25px;">TestFileResolver</a>
</li>
</ul>
<h3>组件功能性扩展 </h3>
<p style="font-size: 14px;">  目前这块扩展性机制不够，设计时只预留了接口，但新增一个功能实现，需要通过硬编码的方式去进行，下载otter的源码，增加功能支持，修改spring配置，同时修改web页面，方便使用。</p>
<p style="font-size: 14px;">  基于manager的灵活扩展性的实现，暂没有想到很好的办法，如果你有好的思路和实现方式，也可以告知我们，谢谢。</p>
<p style="font-size: 14px;">   </p>
<p style="font-size: 14px;">  比如举增加hbase load实现为例，需要扩展的内容： </p>
<ul style="font-size: 14px;">
<li>增加hbase数据源的抽象</li>
<li>增加hbase表的抽象，比如column，columnFamily</li>
<li>增加hbase transform的实现</li>
<li>增加hbase loader的实现</li>
</ul>
</div>