关于单例模式，我的博客中有很多文章介绍过。作为23种设计模式中最为常用的设计模式，单例模式并没有想象的那么简单。因为在设计单例的时候要考虑很多问题，比如线程安全问题、序列化对单例的破坏等。

单例相关文章一览：

[设计模式（二）——单例模式][1] 
[设计模式（三）——JDK中的那些单例][2] 
[单例模式的七种写法][3] 
[单例与序列化的那些事儿][4] 
[不使用synchronized和lock，如何实现一个线程安全的单例？][5] 
[不使用synchronized和lock，如何实现一个线程安全的单例？（二）][6]

如果你对单例不是很了解，或者对于单例的线程安全问题以及序列化会破坏单例等问题不是很清楚，可以先阅读以上文章。上面六篇文章看完之后，相信你一定会对单例模式有更多，更深入的理解。

我们知道，单例模式，一般有七种写法，那么这七种写法中，最好的是哪一种呢？为什么呢？本文就来抽丝剥茧一下。

### 哪种写单例的方式最好

在StakcOverflow中，有一个关于[What is an efficient way to implement a singleton pattern in Java?][7]的讨论：

<img src="https://www.hollischuang.com/wp-content/uploads/2018/06/enum.png" alt="" width="1500" height="1158" class="aligncenter size-full wp-image-3683" />

如上图，得票率最高的回答是：使用枚举。

回答者引用了Joshua Bloch大神在《Effective Java》中明确表达过的观点：

> 使用枚举实现单例的方法虽然还没有广泛采用，但是单元素的枚举类型已经成为实现Singleton的最佳方法。

如果你真的深入理解了单例的用法以及一些可能存在的坑的话，那么你也许也能得到相同的结论，那就是：使用枚举实现单例是一种很好的方法。

### 枚举单例写法简单

如果你看过《[单例模式的七种写法][3]》中的实现单例的所有方式的代码，那就会发现，各种方式实现单例的代码都比较复杂。主要原因是在考虑线程安全问题。

我们简单对比下“双重校验锁”方式和枚举方式实现单例的代码。

“双重校验锁”实现单例：

    public class Singleton {  
        private volatile static Singleton singleton;  
        private Singleton (){}  
        public static Singleton getSingleton() {  
        if (singleton == null) {  
            synchronized (Singleton.class) {  
            if (singleton == null) {  
                singleton = new Singleton();  
            }  
            }  
        }  
        return singleton;  
        }  
    }  
    

枚举实现单例：

    public enum Singleton {  
        INSTANCE;  
        public void whateverMethod() {  
        }  
    }  
    

相比之下，你就会发现，枚举实现单例的代码会精简很多。

上面的双重锁校验的代码之所以很臃肿，是因为大部分代码都是在保证线程安全。为了在保证线程安全和锁粒度之间做权衡，代码难免会写的复杂些。但是，这段代码还是有问题的，因为他无法解决反序列化会破坏单例的问题。

### 枚举可解决线程安全问题

上面提到过。使用非枚举的方式实现单例，都要自己来保证线程安全，所以，这就导致其他方法必然是比较臃肿的。那么，为什么使用枚举就不需要解决线程安全问题呢？

其实，并不是使用枚举就不需要保证线程安全，只不过线程安全的保证不需要我们关心而已。也就是说，其实在“底层”还是做了线程安全方面的保证的。

那么，“底层”到底指的是什么？

这就要说到关于枚举的实现了。这部分内容可以参考我的另外一篇博文[深度分析Java的枚举类型—-枚举的线程安全性及序列化问题][8]，这里我简单说明一下：

定义枚举时使用enum和class一样，是Java中的一个关键字。就像class对应用一个Class类一样，enum也对应有一个Enum类。

通过将定义好的枚举[反编译][9]，我们就能发现，其实枚举在经过`javac`的编译之后，会被转换成形如`public final class T extends Enum`的定义。

而且，枚举中的各个枚举项同事通过`static`来定义的。如：

    public enum T {
        SPRING,SUMMER,AUTUMN,WINTER;
    }
    

反编译后代码为：

    public final class T extends Enum
    {
        //省略部分内容
        public static final T SPRING;
        public static final T SUMMER;
        public static final T AUTUMN;
        public static final T WINTER;
        private static final T ENUM$VALUES[];
        static
        {
            SPRING = new T("SPRING", 0);
            SUMMER = new T("SUMMER", 1);
            AUTUMN = new T("AUTUMN", 2);
            WINTER = new T("WINTER", 3);
            ENUM$VALUES = (new T[] {
                SPRING, SUMMER, AUTUMN, WINTER
            });
        }
    }
    

了解JVM的类加载机制的朋友应该对这部分比较清楚。`static`类型的属性会在类被加载之后被初始化，我们在[深度分析Java的ClassLoader机制（源码级别）][10]和[Java类的加载、链接和初始化][11]两个文章中分别介绍过，当一个Java类第一次被真正使用到的时候静态资源被初始化、Java类的加载和初始化过程都是线程安全的（因为虚拟机在加载枚举的类的时候，会使用ClassLoader的loadClass方法，而这个方法使用同步代码块保证了线程安全）。所以，创建一个enum类型是线程安全的。

也就是说，我们定义的一个枚举，在第一次被真正用到的时候，会被虚拟机加载并初始化，而这个初始化过程是线程安全的。而我们知道，解决单例的并发问题，主要解决的就是初始化过程中的线程安全问题。

所以，由于枚举的以上特性，枚举实现的单例是天生线程安全的。

### 枚举可解决反序列化会破坏单例的问题

前面我们提到过，就是使用双重校验锁实现的单例其实是存在一定问题的，就是这种单例有可能被序列化锁破坏，关于这种破坏及解决办法，参看[单例与序列化的那些事儿][4]，这里不做更加详细的说明了。

那么，对于序列化这件事情，为什么枚举又有先天的优势了呢？答案可以在[Java Object Serialization Specification][12] 中找到答案。其中专门对枚举的序列化做了如下规定：

<img src="http://www.hollischuang.com/wp-content/uploads/2018/06/serialization.png" alt="serialization" width="1406" height="259" class="aligncenter size-full wp-image-2502" />

大概意思就是：在序列化的时候Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过`java.lang.Enum`的`valueOf`方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了`writeObject`、`readObject`、`readObjectNoData`、`writeReplace`和`readResolve`等方法。

普通的Java类的反序列化过程中，会通过反射调用类的默认构造函数来初始化对象。所以，即使单例中构造函数是私有的，也会被反射给破坏掉。由于反序列化后的对象是重新new出来的，所以这就破坏了单例。

但是，枚举的反序列化并不是通过反射实现的。所以，也就不会发生由于反序列化导致的单例破坏问题。这部分内容在[深度分析Java的枚举类型—-枚举的线程安全性及序列化问题][8]中也有更加详细的介绍，还展示了部分代码，感兴趣的朋友可以前往阅读。

### 总结

在所有的单例实现方式中，枚举是一种在代码写法上最简单的方式，之所以代码十分简洁，是因为Java给我们提供了`enum`关键字，我们便可以很方便的声明一个枚举类型，而不需要关心其初始化过程中的线程安全问题，因为枚举类在被虚拟机加载的时候会保证线程安全的被初始化。

除此之外，在序列化方面，Java中有明确规定，枚举的序列化和反序列化是有特殊定制的。这就可以避免反序列化过程中由于反射而导致的单例被破坏问题。

 [1]: http://www.hollischuang.com/archives/1373
 [2]: http://www.hollischuang.com/archives/1383
 [3]: http://www.hollischuang.com/archives/205
 [4]: http://www.hollischuang.com/archives/1144
 [5]: http://www.hollischuang.com/archives/1860
 [6]: http://www.hollischuang.com/archives/1866
 [7]: https://stackoverflow.com/questions/70689/what-is-an-efficient-way-to-implement-a-singleton-pattern-in-java
 [8]: http://www.hollischuang.com/archives/197
 [9]: http://www.hollischuang.com/archives/58
 [10]: http://www.hollischuang.com/archives/199
 [11]: http://www.hollischuang.com/archives/201
 [12]: https://docs.oracle.com/javase/7/docs/platform/serialization/spec/serial-arch.html#6469