### JVM内存分配策略

关于JVM的内存结构及内存分配方式，不是本文的重点，这里只做简单回顾。以下是我们知道的一些常识：

1、根据Java虚拟机规范，Java虚拟机所管理的内存包括方法区、虚拟机栈、本地方法栈、堆、程序计数器等。

2、我们通常认为JVM中运行时数据存储包括堆和栈。这里所提到的栈其实指的是虚拟机栈，或者说是虚拟栈中的局部变量表。

3、栈中存放一些基本类型的变量数据（int/short/long/byte/float/double/Boolean/char）和对象引用。

4、堆中主要存放对象，即通过new关键字创建的对象。

5、数组引用变量是存放在栈内存中，数组元素是存放在堆内存中。

在《深入理解Java虚拟机中》关于Java堆内存有这样一段描述：

但是，随着JIT编译期的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。

这里只是简单提了一句，并没有深入分析，很多人看到这里由于对JIT、逃逸分析等技术不了解，所以也无法真正理解上面这段话的含义。

**PS：这里默认大家都了解什么是JIT，不了解的朋友可以先自行Google了解下，或者加入我的知识星球，阅读那篇球友专享文章。**

其实，在编译期间，JIT会对代码做很多优化。其中有一部分优化的目的就是减少内存堆分配压力，其中一种重要的技术叫做**逃逸分析**。

### 逃逸分析

逃逸分析(Escape Analysis)是目前Java虚拟机中比较前沿的优化技术。这是一种可以有效减少Java 程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。

逃逸分析的基本行为就是分析对象动态作用域：当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为方法逃逸。

例如：

    public static StringBuffer craeteStringBuffer(String s1, String s2) {
        StringBuffer sb = new StringBuffer();
        sb.append(s1);
        sb.append(s2);
        return sb;
    }
    

StringBuffer sb是一个方法内部变量，上述代码中直接将sb返回，这样这个StringBuffer有可能被其他方法所改变，这样它的作用域就不只是在方法内部，虽然它是一个局部变量，称其逃逸到了方法外部。甚至还有可能被外部线程访问到，譬如赋值给类变量或可以在其他线程中访问的实例变量，称为线程逃逸。

上述代码如果想要StringBuffer sb不逃出方法，可以这样写：

    public static String createStringBuffer(String s1, String s2) {
        StringBuffer sb = new StringBuffer();
        sb.append(s1);
        sb.append(s2);
        return sb.toString();
    }
    

不直接返回 StringBuffer，那么StringBuffer将不会逃逸出方法。

使用逃逸分析，编译器可以对代码做如下优化：

一、同步省略。如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。

二、将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。

三、分离对象或标量替换。有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

上面的关于同步省略的内容，我在《[深入理解多线程（五）—— Java虚拟机的锁优化技术][1]》中有介绍过，即锁优化中的锁消除技术，依赖的也是逃逸分析技术。

本文，主要来介绍逃逸分析的第二个用途：将堆分配转化为栈分配。

> 其实，以上三种优化中，栈上内存分配其实是依靠标量替换来实现的。由于不是本文重点，这里就不展开介绍了。如果大家感兴趣，我后面专门出一篇文章，全面介绍下逃逸分析。

在Java代码运行时，通过JVM参数可指定是否开启逃逸分析， `-XX:+DoEscapeAnalysis` ： 表示开启逃逸分析 `-XX:-DoEscapeAnalysis` ： 表示关闭逃逸分析 从jdk 1.7开始已经默认开始逃逸分析，如需关闭，需要指定`-XX:-DoEscapeAnalysis`

### 对象的栈上内存分配

我们知道，在一般情况下，对象和数组元素的内存分配是在堆内存上进行的。但是随着JIT编译器的日渐成熟，很多优化使这种分配策略并不绝对。JIT编译器就可以在编译期间根据逃逸分析的结果，来决定是否可以将对象的内存分配从堆转化为栈。

我们来看以下代码：

    public static void main(String[] args) {
        long a1 = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            alloc();
        }
        // 查看执行时间
        long a2 = System.currentTimeMillis();
        System.out.println("cost " + (a2 - a1) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(100000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }
    
    private static void alloc() {
        User user = new User();
    }
    
    static class User {
    
    }
    

其实代码内容很简单，就是使用for循环，在代码中创建100万个User对象。

**我们在alloc方法中定义了User对象，但是并没有在方法外部引用他。也就是说，这个对象并不会逃逸到alloc外部。经过JIT的逃逸分析之后，就可以对其内存分配进行优化。**

我们指定以下JVM参数并运行：

    -Xmx4G -Xms4G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError 
    

在程序打印出 `cost XX ms` 后，代码运行结束之前，我们使用`[jmap][1]`命令，来查看下当前堆内存中有多少个User对象：

    ➜  ~ jps
    2809 StackAllocTest
    2810 Jps
    ➜  ~ jmap -histo 2809
    
     num     #instances         #bytes  class name
    ----------------------------------------------
       1:           524       87282184  [I
       2:       1000000       16000000  StackAllocTest$User
       3:          6806        2093136  [B
       4:          8006        1320872  [C
       5:          4188         100512  java.lang.String
       6:           581          66304  java.lang.Class
    

从上面的jmap执行结果中我们可以看到，堆中共创建了100万个`StackAllocTest$User`实例。

在关闭逃避分析的情况下（-XX:-DoEscapeAnalysis），虽然在alloc方法中创建的User对象并没有逃逸到方法外部，但是还是被分配在堆内存中。也就说，如果没有JIT编译器优化，没有逃逸分析技术，正常情况下就应该是这样的。即所有对象都分配到堆内存中。

接下来，我们开启逃逸分析，再来执行下以上代码。

    -Xmx4G -Xms4G -XX:+DoEscapeAnalysis -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError 
    

在程序打印出 `cost XX ms` 后，代码运行结束之前，我们使用`jmap`命令，来查看下当前堆内存中有多少个User对象：

    ➜  ~ jps
    709
    2858 Launcher
    2859 StackAllocTest
    2860 Jps
    ➜  ~ jmap -histo 2859
    
     num     #instances         #bytes  class name
    ----------------------------------------------
       1:           524      101944280  [I
       2:          6806        2093136  [B
       3:         83619        1337904  StackAllocTest$User
       4:          8006        1320872  [C
       5:          4188         100512  java.lang.String
       6:           581          66304  java.lang.Class
    

从以上打印结果中可以发现，开启了逃逸分析之后（-XX:+DoEscapeAnalysis），在堆内存中只有8万多个`StackAllocTest$User`对象。也就是说在经过JIT优化之后，堆内存中分配的对象数量，从100万降到了8万。

> 除了以上通过jmap验证对象个数的方法以外，读者还可以尝试将堆内存调小，然后执行以上代码，根据GC的次数来分析，也能发现，开启了逃逸分析之后，在运行期间，GC次数会明显减少。正是因为很多堆上分配被优化成了栈上分配，所以GC次数有了明显的减少。

### 总结

所以，如果以后再有人问你：是不是所有的对象和数组都会在堆内存分配空间？

那么你可以告诉他：不一定，随着JIT编译器的发展，在编译期间，如果JIT经过逃逸分析，发现有些对象没有逃逸出方法，那么有可能堆内存分配会被优化成栈内存分配。但是这也并不是绝对的。就像我们前面看到的一样，在开启逃逸分析之后，也并不是所有User对象都没有在堆上分配。

 [1]: http://www.hollischuang.com/archives/2344