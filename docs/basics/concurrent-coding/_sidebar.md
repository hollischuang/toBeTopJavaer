* 并发与并行
    
    * [什么是并发](/basics/concurrent-coding/concurrent.md)
    
    * [什么是并行](/basics/concurrent-coding/parallel.md)
    
    * [并发与并行的区别](/basics/concurrent-coding/concurrent-vs-parallel.md)
    
* 线程
    
    * 线程的实现
    * 线程的状态
    * 线程优先级
    * 线程调度
    * 创建线程的多种方式
    * 守护线程
    * 线程与进程的区别
    
* 线程池
    
    * 自己设计线程池
    * submit() 和 execute()
    * 线程池原理
    * 为什么不允许使用Executors创建线程池
    
* 线程安全
    
    * [死锁？](/basics/concurrent-coding/deadlock-java-level.md)
    * 死锁如何排查
    * 线程安全和内存模型的关系
    
* 锁
    
    * CAS
    * 乐观锁与悲观锁
    * 数据库相关锁机制
    * 分布式锁
    * 偏向锁
    * 轻量级锁
    * 重量级锁
    * monitor
    * 锁优化
    * 锁消除
    * 锁粗化
    * 自旋锁
    * 可重入锁
    * 阻塞锁
    
* 死锁
    
    * 死锁的原因
    
    * 死锁的解决办法
    
* synchronized
    
    * [synchronized是如何实现的？](/basics/concurrent-coding/synchronized.md)
    
    * synchronized和lock之间关系
    * 不使用synchronized如何实现一个线程安全的单例
    * synchronized和原子性、可见性和有序性之间的关系
    
* volatile
    
    * happens-before
    * 内存屏障
    * 编译器指令重排和CPU指令重排
    
    * volatile的实现原理
    
    * volatile和原子性
    * 可见性和有序性之间的关系
    
    * 有了symchronized为什么还需要volatile
    
* sleep 和 wait
    
* wait 和 notify
    
* notify 和 notifyAll
    
* ThreadLocal
    
* 写一个死锁的程序
    
* 写代码来解决生产者消费者问题
    
* 并发包
    
* 阅读源代码，并学会使用
    
    * Thread
    * Runnable
    * Callable
    * ReentrantLock
    * ReentrantReadWriteLock
    * Atomic*
    * Semaphore
    * CountDownLatch
    * ConcurrentHashMap
    * Executors
    