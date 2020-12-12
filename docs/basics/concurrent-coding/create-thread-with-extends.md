

    /**
     * @author Hollis
     */
    public class MultiThreads {
    
        public static void main(String[] args) throws InterruptedException {
            System.out.println(Thread.currentThread().getName());
    
            System.out.println("继承Thread类创建线程");
            SubClassThread subClassThread = new SubClassThread();
            subClassThread.start();  
        }
    }
    
    class SubClassThread extends Thread {
    
        @Override
        public void run() {
            System.out.println(getName());
        }
    }
    

输出结果：

    main
    继承Thread类创建线程
    Thread-0
    

SubClassThread是一个继承了Thread类的子类，继承Thread类，并重写其中的run方法。然后new 一个SubClassThread的对象，并调用其start方法，即可启动一个线程。之后就会运行run中的代码。

每个线程都是通过某个特定Thread对象所对应的方法`run()`来完成其操作的，方法`run()`称为线程体。通过调用Thread类的`start()`方法来启动一个线程。

在主线程中，调用了子线程的`start()`方法后，主线程无需等待子线程的执行，即可执行后续的代码。而子线程便会开始执行其`run()`方法。

当然，`run()`方法也是一个公有方法，在main函数中也可以直接调用这个方法，但是直接调用`run()`的话，主线程就需要等待其执行完，这种情况下，`run()`就是一个普通方法。

如果读者感兴趣的话，查看一下前面介绍的Thread的源码，就可以发现，他继承了一个接口，那就是`java.lang.Runnable`，其实，开发者在代码中也可以直接通过这个接口创建一个新的线程。