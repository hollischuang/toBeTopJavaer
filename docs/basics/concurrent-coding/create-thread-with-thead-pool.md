Java中提供了对线程池的支持，有很多种方式。Jdk提供给外部的接口也很简单。直接调用ThreadPoolExecutor构造一个就可以了：

    public class MultiThreads {
        public static void main(String[] args) throws InterruptedException, ExecutionException {
            System.out.println(Thread.currentThread().getName());
            System.out.println("通过线程池创建线程");
            ExecutorService executorService = new ThreadPoolExecutor(1, 1, 60L, TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(10));
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName());
                }
            });
        }
    }
    

输出结果：

    main
    通过线程池创建线程
    pool-1-thread-1
    

所谓线程池本质是一个hashSet。多余的任务会放在阻塞队列中。

线程池的创建方式其实也有很多，也可以通过Executors静态工厂构建，但一般不建议。建议使用线程池来创建线程，并且建议使用带有ThreadFactory参数的ThreadPoolExecutor（需要依赖guava）构造方法设置线程名字，具体原因我们在后面的章节中在详细介绍。