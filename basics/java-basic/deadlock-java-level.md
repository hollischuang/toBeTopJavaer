java级别死锁

## 一、什么是死锁
死锁不仅在个人学习中，甚至在开发中也并不常见。但是一旦出现死锁，后果将非常严重。
首先什么是死锁呢？打个比方，就好像有两个人打架，互相限制住了(锁住，抱住)彼此一样，互相动弹不得，而且互相欧气，你不松手我就不松手。好了谁也动弹不得。
在多线程的环境下，势必会对资源进行抢夺。当两个线程锁住了当前资源，但都需要对方的资源才能进行下一步操作，这个时候两方就会一直等待对方的资源释放。这就形成了死锁。这些永远在互相等待的进程称为死锁进程。

那么我们来总结一下死锁产生的条件：

  1. 互斥:资源的锁是排他性的，加锁期间只能有一个线程拥有该资源。其他线程只能等待锁释放才能尝试获取该资源。
  2. 请求和保持:当前线程已经拥有至少一个资源，但其同时又发出新的资源请求，而被请求的资源被其他线程拥有。此时进入保持当前资源并等待下个资源的状态。
  3. 不剥夺：线程已拥有的资源，只能由自己释放，不能被其他线程剥夺。
  4. 循环等待：是指有多个线程互相的请求对方的资源，但同时拥有对方下一步所需的资源。形成一种循环，类似2)请求和保持。但此处指多个线程的关系。并不是指单个线程一直在循环中等待。

什么？还是不理解？那我们直接上代码，动手写一个死锁。

## 二、动手写死锁
根据条件，我们让两个线程互相请求保持。
```java
public class DeadLockDemo implements Runnable{

    public static int flag = 1;

    //static 变量是 类对象共享的
    static Object o1 = new Object();
    static Object o2 = new Object();

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "：此时 flag = " + flag);
        if(flag == 1){
            synchronized (o1){
                try {
                    System.out.println("我是" + Thread.currentThread().getName() + "锁住 o1");
                    Thread.sleep(3000);
                    System.out.println(Thread.currentThread().getName() + "醒来->准备获取 o2");
                }catch (Exception e){
                    e.printStackTrace();
                }
                synchronized (o2){
                    System.out.println(Thread.currentThread().getName() + "拿到 o2");//第24行
                }
            }
        }
        if(flag == 0){
            synchronized (o2){
                try {
                    System.out.println("我是" + Thread.currentThread().getName() + "锁住 o2");
                    Thread.sleep(3000);
                    System.out.println(Thread.currentThread().getName() + "醒来->准备获取 o2");
                }catch (Exception e){
                    e.printStackTrace();
                }
                synchronized (o1){
                    System.out.println(Thread.currentThread().getName() + "拿到 o1");//第38行
                }
            }
        }
    }

    public static  void main(String args[]){

        DeadLockDemo t1 = new DeadLockDemo();
        DeadLockDemo t2 = new DeadLockDemo();
        t1.flag = 1;
        new Thread(t1).start();

        //让main线程休眠1秒钟,保证t2开启锁住o2.进入死锁
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        t2.flag = 0;
        new Thread(t2).start();

    }
}

```
代码中，
t1创建，t1先拿到o1的锁，开始休眠3秒。然后
t2线程创建，t2拿到o2的锁，开始休眠3秒。然后
t1先醒来，准备拿o2的锁，发现o2已经加锁，只能等待o2的锁释放。
t2后醒来，准备拿o1的锁，发现o1已经加锁，只能等待o1的锁释放。
t1,t2形成死锁。


我们查看运行状态，

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329173537340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RpbW8xMTYwMTM5MjEx,size_16,color_FFFFFF,t_70)

## 三、发现排查死锁情况
我们利用jdk提供的工具定位死锁问题：

  1. jps显示所有当前Java虚拟机进程名及pid.
  2. jstack打印进程堆栈信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329174354777.png)

列出所有java进程。
我们检查一下DeadLockDemo，为什么这个线程不退栈。

```shell
jstack 11170
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329174417873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RpbW8xMTYwMTM5MjEx,size_16,color_FFFFFF,t_70)

我们直接翻到最后:已经检测出了一个java级别死锁。其中两个线程分别卡在了代码第38行和第24行。检查我们代码的对应位置，即可排查错误。此处我们是第二个锁始终拿不到，所以死锁了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329174407208.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RpbW8xMTYwMTM5MjEx,size_16,color_FFFFFF,t_70)


## 四、解决办法
死锁一旦发生，我们就无法解决了。所以我们只能避免死锁的发生。
既然死锁需要满足四种条件，那我们就从条件下手，只要打破任意规则即可。

  1. （互斥）尽量少用互斥锁，能加读锁，不加写锁。当然这条无法避免。
  2. （请求和保持）采用资源静态分配策略（进程资源静态分配方式是指一个进程在建立时就分配了它需要的全部资源）.我们尽量不让线程同时去请求多个锁，或者在拥有一个锁又请求不到下个锁时，不保持等待，先释放资源等待一段时间在重新请求。
  3. （不剥夺）允许进程剥夺使用其他进程占有的资源。优先级。
  4. （循环等待）尽量调整获得锁的顺序，不发生嵌套资源请求。加入超时。

