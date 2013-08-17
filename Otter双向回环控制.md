  <div class="blog_content">
    <div class="iteye-blog-content-contain">
<h1>基本需求</h1>
<ol>
<li>支持mysql/oracle的异构数据库的双向回环，早期有变态需求：杭州是mysql，美国是oracle，需要做双向同步。</li>
<li>需要支持级联同步，比如A&lt;-&gt;B-&gt;C，A同步到B的数据，不能从B回到A，但需要同步到C</li>
</ol>
<h1>实现思路</h1>
<ul>
<li>利用事务机制，在事务头和尾中插入otter同步标识</li>
<li>解析时识别同步标识，判断是否需要屏蔽同步</li>
</ul>
<p>几点注意：</p>
<ul>
<li>基于标准SQL实现<br>可以支持mysql/oracle等异构数据库的双向同步</li>
<li>事务完整解析&amp;完整可见性<br>事务被拆开同步，会出现部分回环同步，数据不一致.  比如一个事务被拆分为了3截，中间一截因为没有事务头和尾的标识，如果发生同步了，就会导致数据不一致. </li>
</ul>
<h1>实现示意图</h1>
<p><img alt="" height="344" width="705" src="http://dl2.iteye.com/upload/attachment/0088/3083/40aabd49-c517-34c1-ac13-84d7f6be90e4.png"></p>
<p><br> </p>
</div>
    