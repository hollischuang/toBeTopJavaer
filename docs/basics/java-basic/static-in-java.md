static表示“静态”的意思，用来修饰成员变量和成员方法，也可以形成静态static代码块

### 静态变量

我们用static表示变量的级别，一个类中的静态变量，不属于类的对象或者实例。因为静态变量与所有的对象实例共享，因此他们不具线程安全性。

通常，静态变量常用final关键来修饰，表示通用资源或可以被所有的对象所使用。如果静态变量未被私有化，可以用“类名.变量名”的方式来使用。

    //static variable example
    private static int count;
    public static String str;

### 静态方法

与静态变量一样，静态方法是属于类而不是实例。

一个静态方法只能使用静态变量和调用静态方法。通常静态方法通常用于想给其他的类使用而不需要创建实例。例如：Collections class(类集合)。

Java的包装类和实用类包含许多静态方法。main()方法就是Java程序入口点，是静态方法。

    //static method example
    public static void setCount(int count) {
        if(count &gt; 0)
        StaticExample.count = count;
    }
    
    //static util method
    public static int addInts(int i, int...js){
        int sum=i;
        for(int x : js) sum+=x;
        return sum;
    }
    
从Java8以上版本开始也可以有接口类型的静态方法了。

### 静态代码块

Java的静态块是一组指令在类装载的时候在内存中由Java ClassLoader执行。

静态块常用于初始化类的静态变量。大多时候还用于在类装载时候创建静态资源。

Java不允许在静态块中使用非静态变量。一个类中可以有多个静态块，尽管这似乎没有什么用。静态块只在类装载入内存时，执行一次。

    static{
        //can be used to initialize resources when class is loaded
        System.out.println(&quot;StaticExample static block&quot;);
        //can access only static variables and methods
        str=&quot;Test&quot;;
        setCount(2);
    }
    
### 静态类

Java可以嵌套使用静态类，但是静态类不能用于嵌套的顶层。

静态嵌套类的使用与其他顶层类一样，嵌套只是为了便于项目打包。


原文地址：https://zhuanlan.zhihu.com/p/26819685