我们学习过，Java虚拟机采用抢占式调度模型。也就是说他会给优先级更高的线程优先分配CPU。

虽然Java线程调度是系统自动完成的，但是我们还是可以“建议”系统给某些线程多分配一点执行时间，另外的一些线程则可以少分配一点——这项操作可以通过设置线程优先级来完成。

Java语言一共设置了10个级别的线程优先级（Thread.MIN_PRIORITY至Thread.MAX_PRIORITY），在两个线程同时处于Ready状态时，优先级越高的线程越容易被系统选择执行。

Java 线程优先级使用 1 ~ 10 的整数表示。默认的优先级是5。

    最低优先级 1：Thread.MIN_PRIORITY
    
    最高优先级 10：Thread.MAX_PRIORITY
    
    普通优先级 5：Thread.NORM_PRIORITY
    

在Java中，可以使用Thread类的`setPriority()`方法为线程设置了新的优先级。`getPriority()`方法返回线程的当前优先级。当创建一个线程时，其默认优先级是创建该线程的线程的优先级。

以下代码演示如何设置和获取线程的优先：

    /**
     * @author Hollis
     */
    public class Main {
    
        public static void main(String[] args) {
            Thread t = Thread.currentThread();
            System.out.println("Main Thread  Priority:" + t.getPriority());
    
            Thread t1 = new Thread();
            System.out.println("Thread(t1) Priority:" + t1.getPriority());
            t1.setPriority(Thread.MAX_PRIORITY - 1);
            System.out.println("Thread(t1) Priority:" + t1.getPriority());
    
            t.setPriority(Thread.NORM_PRIORITY);
            System.out.println("Main Thread  Priority:" + t.getPriority());
    
            Thread t2 = new Thread();
            System.out.println("Thread(t2) Priority:" + t2.getPriority());
    
            // Change thread t2 priority to minimum
            t2.setPriority(Thread.MIN_PRIORITY);
            System.out.println("Thread(t2) Priority:" + t2.getPriority());
        }
    
    }
    

输出结果为：

    Main Thread  Priority:5
    Thread(t1) Priority:5
    Thread(t1) Priority:9
    Main Thread  Priority:5
    Thread(t2) Priority:5
    Thread(t2) Priority:1
    

在上面的代码中，Java虚拟机启动时，就会通过main方法启动一个线程，JVM就会一直运行下去，直到以下任意一个条件发生：

*   调用了exit()方法，并且exit()有权限被正常执行。
*   所有的“非守护线程”都死了(即JVM中仅仅只有“守护线程”)。