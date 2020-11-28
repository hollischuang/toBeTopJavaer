线程是有状态的，并且这些状态之间也是可以互相流转的。Java中线程的状态分为6种：

*   1\.初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
*   1\.运行(RUNNABLE)：Java线程中将就绪（READY）和运行中（RUNNING）两种状态笼统的称为“运行”。 
    *   就绪（READY）:线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中并分配cpu使用权 。
    *   运行中（RUNNING）：就绪(READY)的线程获得了cpu 时间片，开始执行程序代码。
*   3\.阻塞(BLOCKED)：表示线程阻塞于锁（关于锁，在后面章节会介绍）。
*   4\.等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
*   5\.超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。
*   6\. 终止(TERMINATED)：表示该线程已经执行完毕。

下图是一张线程状态的流转图：

<img src="https://www.hollischuang.com/wp-content/uploads/2018/12/167019dc85aaaf5a.jpg" alt="" width="1155" height="771" class="aligncenter size-full wp-image-5859" />


可以看到，图中的各个状态之间的流转路径上都有标注对应的Java中的方法。这些就是Java中进行线程调度的一些api。

