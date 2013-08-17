<div class="blog_content">
    <div style="font-size: 14px;" class="iteye-blog-content-contain">
<h1>背景</h1>
<p> 在介绍调度模型之前，首先了解一下otter系统要解决的异地机房的网络环境.  </p>
<p> </p>
<ol>
<li>中美网络延迟 (平均200ms)</li>
<li>中美传输速度 (2~6MB/s)</li>
</ol>
<p>网络因素是一个很重要的问题，</p>
<p>&nbsp;&nbsp;&nbsp;   a.  比如中美网络延迟RTT，平均为200ms，这直接会影响整个系统的架构设计。</p>
<p>试想一下，发送一条binlog一次RTT 200ms，那是否意味着单线程1秒钟只能发送5条。不过tcp在解决这类问题时，有自己的一套优化算法，叫做滑动窗口，它发送数据可能一次性发了10条，然后一起等返回结果，这样可以提升传输效率，但始终无法满足1秒传输1w+记录的需求。这也就决定了，需要对发送的binlog做批处理，一次性发送尽可能多的数据，然后一起等结果. </p>
<p> </p>
<p>&nbsp;&nbsp;&nbsp;   b . 比如中美单socket的带宽只有2～6MB，这直接会影响整个系统的架构设计。</p>
<p>试想一下，假定一binlog平均1kb，那6MB最多只有6000条数据，也就意味着最大的同步tps只有6000? 而且很多业务的mysql都是共享，也就是6000个binlog对象中，可能只有三分之一或者四分之一是某个业务的，这肯定是无法满足要求的。所以，基于带宽的问题，决定otter架构必须是双节点部署，在杭州一个节点，美国一个节点，杭州这边对数据做加速同步处理，然后快速传递到美国. </p>
<p> </p>
<p>基于这两个因素，决定了： batch处理 +  双节点部署的架构.  </p>
<p> </p>
<h1>调度模型</h1>
<p><img style="font-size: 12px; line-height: 1.5;" width="306" alt="" height="232" src="http://dl2.iteye.com/upload/attachment/0088/3053/050fca62-3302-3380-8c0c-d8afae648a35.png"></p>
<p>在正式介绍otter调度模型之前，我们首先得了解TCP/IP协议在解决此类"差网络"环境的一些处理方案，从中借鉴相应的方案.  </p>
<p> </p>
<p><img alt="" src="http://dl2.iteye.com/upload/attachment/0088/3073/6d6e4f03-23df-3759-b7db-370e08c7b34c.png"></p>
<p>在介绍之前，首先要了解一下otter的概念模型，S/E/T/L模块，不同的模块复制各自的业务，如果要扩展也只需要扩展其中的一个模块即可。</p>
<p>这其中主要是引入了数据仓库中的ETL模型，支持系统的扩展性，增加了Select模块，解决数据来源的差异性问题. </p>
<p> </p>
<h3>Nagle算法</h3>
<p>通过<a href="https://github.com/alibaba/canal">Canal</a>解决Nagle算法，Canal之前是做为otter的一个子项目，为解决otter的数据增量获取的机制，并为otter项目的特点而量身打造了几个feature. </p>
<p> </p>
<p>Canal的处理：</p>
<p>&nbsp;&nbsp;&nbsp;a.  构建RingBuffer (可以基于内存控制模式/数量控制模式)</p>
<p>&nbsp;&nbsp;&nbsp;b.  允许客户端指定batchSize获取</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  i.  内存大小 </p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  ii.  记录数</p>
<p>&nbsp;&nbsp;&nbsp;c.   指定定batchSize + timeout获取</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   i.  timeout = -1 ,即时获取，有多少取多少</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   ii. timeout = 0，阻塞至满足batchSize条件</p>
<p> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  iii. timeout &gt; 0，阻塞指定的时间或者满足batchSize.</p>
<p>建议值：batchSize=4000(约4M) , timeout=500，内存控制模式</p>
<p> </p>
<h3>滑动窗口</h3>
<p> <br><img alt="" src="http://dl2.iteye.com/upload/attachment/0088/3055/8849cb1e-44e0-35b9-95d8-0beda30c5929.png"></p>
<p> </p>
<p>说明：</p>
<ol>
<li>otter通过select模块串行获取canal的批数据，<strong>注意是串行获取</strong>，每批次获取到的数据，就会有一个全局标识，otter里称之为processId.  </li>
<li>select模块获取到数据后，将其传递给后续的ETL模型.  这里E和T模块会是一个<strong>并行处理</strong>
</li>
<li>将数据最后传递到Load时，会根据每批数据对应的processId，按照顺序进行<strong>串行加载</strong>。 ( 比如有一个processId=2的数据先到了Load模块，但会阻塞等processId=1的数据Load完成后才会被执行)</li>
</ol>
<p>简单一点说，Select/Load模块会是一个串行机制来保证binlog处理的顺序性，Extract/Transform会是一个并行，加速传输效率. </p>
<p> </p>
<h4><strong>并行度</strong></h4>
<p>  类似于tcp滑动窗口大小，比如整个滑动窗口设置了并行度为5时，只有等第一个processId Load完成后，第6个Select才会去获取数据。</p>
<p> </p>
<h4>数据可靠性</h4>
<ul>
<li>如何保证数据不丢：2pc.  (get/ack)</li>
<li>如何处理重传协议：get/ack/rollback</li>
<li>如何支持并行化：多get cursor+ack curosr (可以参看Canal的异步ACK模型)</li>
</ul>
<p><img alt="" src="http://dl2.iteye.com/upload/attachment/0088/3059/1c163143-39ab-3725-af98-087659976291.png"><br> </p>
<h4>编程模型抽象(SEDA模型)</h4>
<p><img alt="" src="http://dl2.iteye.com/upload/attachment/0088/3061/d02a79a7-fd5b-3059-9078-e04dce11c63f.png"></p>
<p>说明： 将并行化调度的串行/并行处理，进行隐藏，抽象了await/single的接口，整个调度称之为仲裁器。(有了这层抽象，不同的仲裁器实现可以解决同机房，异地机房的同步需求)</p>
<p>模型接口：</p>
<ul>
<li>await模拟object获取锁操作</li>
<li>notify被唤醒后提交任务到thread pools</li>
<li>single模拟object释放锁操作，触发下一个stage</li>
</ul>
<p>这里使用了SEDA模型的优势：</p>
<p> </p>
<ul>
<li>共享thread pool，解决流控机制</li>
<li>划分多stage，提升资源利用率</li>
<li>统一编程模型，支持同机房，跨机房不同的调度算法</li>
</ul>
<h4>仲裁器算法</h4>
<p>主要包括： 令牌生成(processId)  +  事件通知. </p>
<p>令牌生成：</p>
<ul>
<li>基于AtomicLong.inc()机制，(纯内存机制，解决同机房，单节点同步需求，不需要多节点交互)</li>
<li>基于zookeeper的自增id机制，(解决异地机房，多节点协作同步需求)</li>
</ul>
<p>事件通知： (简单原理： 每个stage都会有个block queue，接收上一个stage的single信号通知，当前stage会阻塞在该block queue上，直到有信号通知)</p>
<ul>
<li>block queue + put/take方法，(纯内存机制)</li>
<li>block queue + rpc + put/take方法  (两个stage对应的node不同，需要rpc调用，需要依赖负载均衡算法解决node节点的选择问题)</li>
<li>block queue  +  zookeeper watcher ()</li>
</ul>
<p>负载均衡算法：</p>
<ul>
<li>Stick :  类似于session stick技术，一旦第一次选择了node，下一次选择会继续使用该node.  (有一个好处，资源上下文缓存命中率高)</li>
<li>Random :  随机算法</li>
<li>RoundRbin ： 轮询算法 </li>
</ul>

<span style="font-size: 14px; font-weight: normal; line-height: 21px;">注意点：每个node节点，都会在zookeeper中生成Ephemeral节点，每个node都会缓存住当前存活的node列表，node节点消失，通过zookeeper watcher机制刷新每个node机器的内存。然后针对每次负载均衡选择时只针对当前存活的节点，保证调度的可靠性。</span>

<p> </p>
<h4>调度算法成本估算</h4>
<p> </p>
<p>中美网络RTT = 200ms , zookeeper一次写入=10ms</p>
<p>调度成本估算：</p>
<p>&nbsp;&nbsp;&nbsp;   a.  zookeeper + zookeeper watch (完全分布式)</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      10 * 4 + 200 * 2 + 200 = 640ms</p>
<p>&nbsp;&nbsp;&nbsp;   b.  zookeeper + rpc (sticky分布式，尽可能选择同节点)</p>
<p>&nbsp;&nbsp;&nbsp;      10 + 100 + 200  =  310ms</p>
<p>&nbsp;&nbsp;&nbsp;c.  memory + memory (内存调度，单机房)</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      0ms</p>
<p>&nbsp;&nbsp;&nbsp;d.  memory + rpc (跨机房调度，最优实现，待完成??)</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;     0 + 100 + 100 = 200ms</p>
<p> </p>
<h4>数据传输</h4>
<p>  有了一层SEDA调度模型的抽象，S/E/T/L模块之间互不感知，那几个模块之间的数据传递，需要有一个机制来处理，这里抽象了一个pipe(管道)的概念.  </p>
<p>原理： </p>
<p>   stage |  pipe | stage </p>
<p>基于pipe实现：</p>
<ul>
<li>in memory  (两个stage经过仲裁器调度算法选择为同一个node时，直接使用内存传输)</li>
<li>rpc call (&lt;1MB)</li>
<li>file(gzip) + http多线程下载</li>
</ul>
<p>在pipe中，通过对数据进行TTL控制，解决TCP协议中的丢包问题控制. </p>
<p> </p>
</div>