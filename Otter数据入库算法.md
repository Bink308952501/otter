<h1>核心算法介绍</h1>
实际测试中，otter的同步速度相比于mysql的复制，约有5倍左右的性能提升，这取决于其同步算法的实现. 抛弃了强一致性，得到了性能提升

<h3>数据合并</h3>
<pre>
1. insert + insert -> insert (数据迁移+数据增量场景)
2. insert + update -> insert  (update字段合并到insert)
3. insert + delete -> delete 
4. update + insert -> insert (数据迁移+数据增量场景)
5. update + update -> update
6. update + delete -> delete
7. delete + insert -> insert 
8. delete + update -> update (数据迁移+数据增量场景)
9. delete + delete -> delete
</pre>
说明. 
<p>1.  insert/行记录update 执行merge sql，解决重复数据执行</p>
<p>2.  合并算法执行后，单pk主键只有一条记录，减少并行load算法的复杂性(比如batch合并，并行/串行等处理)</p>

<h3>数据入库算法</h3>
<p>   &nbsp;&nbsp;&nbsp;入库算法采取了按pk hash并行载入+batch合并的优化</p>
<p>   &nbsp;&nbsp;&nbsp;a.  打散原始数据库事务，预处理数据，合并insert/update/delete数据(参见合并算法)，然后按照table + pk进行并行(相同table的数据，先执行delete,后执行insert/update，串行保证，解决唯一性约束数据变更问题)，相同table的sql会进行batch合并处理</p>
<p>   &nbsp;&nbsp;&nbsp;b.  提供table权重定义，根据权重定义不同支持"业务上类事务功能"，并行中同时有串行权重控制. </p>
<p>  </p>
<p>  &nbsp;&nbsp;&nbsp;业务类事务描述：比如用户的一次交易付款的流程，先产生一笔交易记录，然后修改订单状态为已付款.  用户对这事件的感知，是通过订单状态的已付款，然后进行查询交易记录。</p>
<p>  &nbsp;&nbsp;&nbsp;所以，可以对同步进行一次编排： 先同步完交易记录，再同步订单状态。 (给同步表定义权重，权重越高的表相对重要，放在后面同步，最后达到的效果可以保证业务事务可见性的功能，快的等慢的. )</p>

----
<h1>初步性能指标：</h1>
<p>1.  单机房同步</p>
<p>   &nbsp;&nbsp;&nbsp;a.  100tps ， 延迟100ms</p>
<p>   &nbsp;&nbsp;&nbsp;b.  5000tps,  延迟1s</p>
<p>2.  中美异地机房同步</p>
<p>   &nbsp;&nbsp;&nbsp;a.  100tps ， 延迟2s</p>
<p>   &nbsp;&nbsp;&nbsp;b.  5000tps ，延迟10s</p>
ps. 性能指标取决于目标数据库性能，数据大小等多个因素，单机房100b大小，极限tps可以1w+
<p> </p>