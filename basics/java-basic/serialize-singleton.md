本文将通过实例+阅读Java源码的方式介绍序列化是如何破坏单例模式的，以及如何避免序列化对单例的破坏。

> 单例模式，是设计模式中最简单的一种。通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。关于单例模式的使用方式，可以阅读[单例模式的七种写法][1]

但是，单例模式真的能够实现实例的唯一性吗？

答案是否定的，很多人都知道使用反射可以破坏单例模式，除了反射以外，使用序列化与反序列化也同样会破坏单例。

## 序列化对单例的破坏

首先来写一个单例的类：

code 1

    package com.hollis;
    import java.io.Serializable;
    /**
     * Created by hollis on 16/2/5.
     * 使用双重校验锁方式实现单例
     */
    public class Singleton implements Serializable{
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
    

接下来是一个测试类：

code 2

    package com.hollis;
    import java.io.*;
    /**
     * Created by hollis on 16/2/5.
     */
    public class SerializableDemo1 {
        //为了便于理解，忽略关闭流操作及删除文件操作。真正编码时千万不要忘记
        //Exception直接抛出
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            //Write Obj to file
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
            oos.writeObject(Singleton.getSingleton());
            //Read Obj from file
            File file = new File("tempFile");
            ObjectInputStream ois =  new ObjectInputStream(new FileInputStream(file));
            Singleton newInstance = (Singleton) ois.readObject();
            //判断是否是同一个对象
            System.out.println(newInstance == Singleton.getSingleton());
        }
    }
    //false
    

输出结构为false，说明：

> 通过对Singleton的序列化与反序列化得到的对象是一个新的对象，这就破坏了Singleton的单例性。

这里，在介绍如何解决这个问题之前，我们先来深入分析一下，为什么会这样？在反序列化的过程中到底发生了什么。

## ObjectInputStream

对象的序列化过程通过ObjectOutputStream和ObjectInputputStream来实现的，那么带着刚刚的问题，分析一下ObjectInputputStream 的`readObject` 方法执行情况到底是怎样的。

为了节省篇幅，这里给出ObjectInputStream的`readObject`的调用栈：

<img src="https://www.hollischuang.com/wp-content/uploads/2016/02/640.png" alt="" width="840" height="309" class="aligncenter size-full wp-image-3561" />

这里看一下重点代码，`readOrdinaryObject`方法的代码片段： code 3

    private Object readOrdinaryObject(boolean unshared)
            throws IOException
        {
            //此处省略部分代码
    
            Object obj;
            try {
                obj = desc.isInstantiable() ? desc.newInstance() : null;
            } catch (Exception ex) {
                throw (IOException) new InvalidClassException(
                    desc.forClass().getName(),
                    "unable to create instance").initCause(ex);
            }
    
            //此处省略部分代码
    
            if (obj != null &&
                handles.lookupException(passHandle) == null &&
                desc.hasReadResolveMethod())
            {
                Object rep = desc.invokeReadResolve(obj);
                if (unshared && rep.getClass().isArray()) {
                    rep = cloneArray(rep);
                }
                if (rep != obj) {
                    handles.setObject(passHandle, obj = rep);
                }
            }
    
            return obj;
        }
    

code 3 中主要贴出两部分代码。先分析第一部分：

code 3.1

    Object obj;
    try {
        obj = desc.isInstantiable() ? desc.newInstance() : null;
    } catch (Exception ex) {
        throw (IOException) new InvalidClassException(desc.forClass().getName(),"unable to create instance").initCause(ex);
    }
    

这里创建的这个obj对象，就是本方法要返回的对象，也可以暂时理解为是ObjectInputStream的`readObject`返回的对象。

<img src="https://www.hollischuang.com/wp-content/uploads/2016/02/641.jpeg" alt="" width="1080" height="336" class="aligncenter size-full wp-image-3563" />

> `isInstantiable`：如果一个serializable/externalizable的类可以在运行时被实例化，那么该方法就返回true。针对serializable和externalizable我会在其他文章中介绍。
> 
> `desc.newInstance`：该方法通过反射的方式调用无参构造方法新建一个对象。

所以。到目前为止，也就可以解释，为什么序列化可以破坏单例了？

> 答：序列化会通过反射调用无参数的构造方法创建一个新的对象。

那么，接下来我们再看刚开始留下的问题，如何防止序列化/反序列化破坏单例模式。

## 防止序列化破坏单例模式

先给出解决方案，然后再具体分析原理：

只要在Singleton类中定义`readResolve`就可以解决该问题：

code 4

    package com.hollis;
    import java.io.Serializable;
    /**
     * Created by hollis on 16/2/5.
     * 使用双重校验锁方式实现单例
     */
    public class Singleton implements Serializable{
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
    
        private Object readResolve() {
            return singleton;
        }
    }
    

还是运行以下测试类：

    package com.hollis;
    import java.io.*;
    /**
     * Created by hollis on 16/2/5.
     */
    public class SerializableDemo1 {
        //为了便于理解，忽略关闭流操作及删除文件操作。真正编码时千万不要忘记
        //Exception直接抛出
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            //Write Obj to file
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
            oos.writeObject(Singleton.getSingleton());
            //Read Obj from file
            File file = new File("tempFile");
            ObjectInputStream ois =  new ObjectInputStream(new FileInputStream(file));
            Singleton newInstance = (Singleton) ois.readObject();
            //判断是否是同一个对象
            System.out.println(newInstance == Singleton.getSingleton());
        }
    }
    //true
    

本次输出结果为true。具体原理，我们回过头继续分析code 3中的第二段代码:

code 3.2

    if (obj != null &&
                handles.lookupException(passHandle) == null &&
                desc.hasReadResolveMethod())
            {
                Object rep = desc.invokeReadResolve(obj);
                if (unshared && rep.getClass().isArray()) {
                    rep = cloneArray(rep);
                }
                if (rep != obj) {
                    handles.setObject(passHandle, obj = rep);
                }
            }
    

`hasReadResolveMethod`:如果实现了serializable 或者 externalizable接口的类中包含`readResolve`则返回true

`invokeReadResolve`:通过反射的方式调用要被反序列化的类的readResolve方法。

所以，原理也就清楚了，主要在Singleton中定义readResolve方法，并在该方法中指定要返回的对象的生成策略，就可以防止单例被破坏。

## 总结

在涉及到序列化的场景时，要格外注意他对单例的破坏。

## 推荐阅读

[深入分析Java的序列化与反序列化][2]

 [1]: http://www.hollischuang.com/archives/205
 [2]: http://www.hollischuang.com/archives/1140