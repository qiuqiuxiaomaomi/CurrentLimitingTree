# CurrentLimitingTree
高并发系统限流技术研究


<pre>
在开发高并发系统时有三把利器来保护系统：
      1）缓存
      2）降级
      3）限流

   缓存：
       缓存的目的是提升系统访问速度和增大系统处理能力，可谓是抗高并发流量的银弹。
 
   降级：
       是当服务出问题或者影响到核心流程的性能则需要暂时屏蔽掉，待高峰或者问题解决后再打开。

   限流：
       而有些场景并不能用缓存和降级来解决，比如稀缺资源（秒杀，抢购），写服务（评论，下单）
   ，频繁的复杂查询，因此需要有一种手段来限制这些场景的并发、请求量，即限流。
</pre>

<pre>
限流的目的：
      限流的目的是通过对并发访问、请求进行限速或者一个时间窗口内的请求进行限速来保护系统，
   一旦达到限制速率则可以拒绝服务（定向到错误页面或者告知资源没有了），排队或等待（比如秒
   杀，评论，下单），降级（返回兜底数据或默认数据，如商品详情页库存默认有货）
</pre>

<pre>
限流分类
      一般来说高并发系统常见的限流有：
           1）限制总并发数（比如数据库连接池，线程池）
           2）限制瞬时并发数（如nginx的limit_conn，用来限制瞬时并发连接数）
           3）限制时间窗口内的平均速率（如Guava的Ratelimiter, Nginx的limit_req，限制
              每秒的平均速率）
           5）限制远程接口调用速率
           6）限制MQ的消费速率
           7）其他根据网络连接数，网络流量，CPU或内存负载等来限流。
</pre>

<pre>
常见的限流算法：
       1）令牌桶
       2）漏桶
       3）计数器
</pre>

![](https://i.imgur.com/S2MMxuI.png)

<pre>
令牌桶算法：
      令牌桶算法是一个存放固定容量的桶，按照固定速率往桶里添加令牌，令牌桶算法的描述如下：
          1）假设限制1r/s，则按照500ms的固定速率往桶里添加令牌；
          2）桶中最多存放b个令牌，当桶满时，新添加的令牌被丢弃或者拒绝
          3）当一个n字节大小的数据包到达，晶从桶中删除出n个令牌，接着数据包被发送到网络
             上。
          5）如果桶中的令牌不足n个，则不会删除令牌，且该数据包将被限流（要么丢弃，要么
             缓冲区等待）
</pre>

<pre>
漏桶算法：
       漏桶作为计量工具时，可以用于流量整形和流量控制，漏桶算法描述如下：
           1）一个固定容量的漏桶，按照常量固定速率流入水滴；
           2）如果桶是空的，则不需要流出水滴；
           3）如果以任意速率流入睡袋到漏桶；
           5）如果流入水滴超出了桶的容量，则流入的水滴溢出了（被丢弃），而漏桶容量是不变的。
</pre>

<pre>
令牌桶与漏桶的对比
      1）令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看同种令牌是否足够，当令牌
         数目减为0时，则拒绝新的服务。
      2）漏桶则是按照常量固定速率流出请求，流入请求速率任意，当流入的请求数量积到漏桶容量
         时，则新流入的请求被拒绝。
      3）令牌桶限制的平均速率（允许突发请求，只要有令牌就可以处理，支持一次拿3个令牌，4个
         令牌），并允许一定程度突发流量。
      5）漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如都是1的速率流出，而不
         能使一次1，下次是2），从而平滑突发流入速率。
      6）令牌桶允许一定程度的突发，而漏桶主要目的的是平滑流入速率。
      7）两个算法实现可以一样，但是方向是相反的，对于相同的参数得到的限流效果是一样的。

      另外，有时候，我们还使用计数器来进行限流，主要用来限制并发总数，比如数据库连接池，
      线程池，秒杀的总数；只要全局总请求求数或者一定时间段的总请求数设定的阈值则进行限流，
      是简单粗暴的总数量限流，而不是平均速率限流。
</pre>

<pre>
应用级限流
</pre>