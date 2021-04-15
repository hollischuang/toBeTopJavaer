在Java中有两类线程：User Thread(用户线程)、Daemon Thread(守护线程) 。用户线程一般用户执行用户级任务，而守护线程也就是“后台线程”，一般用来执行后台任务，守护线程最典型的应用就是GC(垃圾回收器)。

这两种线程其实是没有什么区别的，唯一的区别就是Java虚拟机在所有“用户线程”都结束后就会退出。

我们可以通过使用`setDaemon()`方法通过传递true作为参数，使线程成为一个守护线程。我们必须在启动线程之前调用一个线程的`setDaemon()`方法。否则，就会抛出一个`java.lang.IllegalThreadStateException`。

可以使用`isDaemon()`方法来检查线程是否是守护线程。

    /**
     * @author Hollis
     */
    public class Main {
        public static void main(String[] args) {
    
            Thread t1 = new Thread();
            System.out.println(t1.isDaemon());
            t1.setDaemon(true);
            System.out.println(t1.isDaemon());
            t1.start();
            t1.setDaemon(false);
        }
    }
    

以上代码输出结果：

    false
    true
    Exception in thread "main" java.lang.IllegalThreadStateException
        at java.lang.Thread.setDaemon(Thread.java:1359)
        at com.hollis.Main.main(Main.java:16)
    

我们提到，当JVM中只剩下守护线程的时候，JVM就会退出，那么写一段代码测试下：

    /**
     * @author Hollis
     */
    public class Main {
        public static void main(String[] args) {
    
            Thread childThread = new Thread(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        System.out.println("I'm child thread..");
                        try {
                            TimeUnit.MILLISECONDS.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
            childThread.start();
            System.out.println("I'm main thread...");
        }
    }
    

以上代码中，我们在Main线程中开启了一个子线程，在并没有显示将其设置为守护线程的情况下，他是一个用户线程，代码比较好理解，就是子线程处于一个while(true)循环中，每隔一秒打印一次`I'm child thread..`

输出结果为：

    I'm main thread...
    I'm child thread..
    I'm child thread..
    .....
    I'm child thread..
    I'm child thread..
    

我们再把子线程设置成守护线程，重新运行以上代码。

    /**
     * @author Hollis
     */
    public class Main {
        public static void main(String[] args) {
    
            Thread childThread = new Thread(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        System.out.println("I'm child thread..");
                        try {
                            TimeUnit.MILLISECONDS.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
            childThread.setDaemon(true);
            childThread.start();
            System.out.println("I'm main thread...");
        }
    }
    

以上代码，我们通过`childThread.setDaemon(true);`把子线程设置成守护线程，然后运行，得到以下结果：

    I'm main thread...
    I'm child thread..
    

子线程只打印了一次，也就是，在main线程执行结束后，由于子线程是一个守护线程，JVM就会直接退出了。

**值得注意的是，在Daemon线程中产生的新线程也是Daemon的。**

提到线程，有一个很重要的东西我们需要介绍一下，那就是ThreadLocal。