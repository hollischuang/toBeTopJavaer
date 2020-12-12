

    public class MultiThreads {
        public static void main(String[] args) throws InterruptedException {
            System.out.println(Thread.currentThread().getName());
    
    
            System.out.println("实现Runnable接口创建线程");
            RunnableThread runnableThread = new RunnableThread();
            new Thread(runnableThread).start();
    
          }
    }
    
    class RunnableThread implements Runnable {
    
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
        }
    }
    

输出结果：

    main
    实现Runnable接口创建线程
    Thread-1
    

通过实现接口，同样覆盖`run()`就可以创建一个新的线程了。

我们都知道，Java是不支持多继承的，所以，使用Runnbale接口的形式，就可以避免要多继承 。比如有一个类A，已经继承了类B，就无法再继承Thread类了，这时候要想实现多线程，就需要使用Runnable接口了。

除此之外，两者之间几乎无差别。

但是，这两种创建线程的方式，其实是有一个缺点的，那就是：在执行完任务之后无法获取执行结果。

如果我们希望再主线程中得到子线程的执行结果的话，就需要用到Callable和FutureTask
