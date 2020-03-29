
## Java对象的序列化与反序列化

在Java中，我们可以通过多种方式来创建对象，并且只要对象没有被回收我们都可以复用该对象。但是，我们创建出来的这些Java对象都是存在于JVM的堆内存中的。只有JVM处于运行状态的时候，这些对象才可能存在。一旦JVM停止运行，这些对象的状态也就随之而丢失了。

但是在真实的应用场景中，我们需要将这些对象持久化下来，并且能够在需要的时候把对象重新读取出来。Java的对象序列化可以帮助我们实现该功能。

对象序列化机制（object serialization）是Java语言内建的一种对象持久化方式，通过对象序列化，可以把对象的状态保存为字节数组，并且可以在有需要的时候将这个字节数组通过反序列化的方式再转换成对象。对象序列化可以很容易的在JVM中的活动对象和字节数组（流）之间进行转换。

在Java中，对象的序列化与反序列化被广泛应用到RMI(远程方法调用)及网络传输中。

## 相关接口及类

Java为了方便开发人员将Java对象进行序列化及反序列化提供了一套方便的API来支持。其中包括以下接口和类：

> java.io.Serializable
> 
> java.io.Externalizable
> 
> ObjectOutput
> 
> ObjectInput
> 
> ObjectOutputStream
> 
> ObjectInputStream

## Serializable 接口

类通过实现 `java.io.Serializable` 接口以启用其序列化功能。未实现此接口的类将无法使其任何状态序列化或反序列化。可序列化类的所有子类型本身都是可序列化的。**序列化接口没有方法或字段，仅用于标识可序列化的语义。** ([该接口并没有方法和字段，为什么只有实现了该接口的类的对象才能被序列化呢？][1])

当试图对一个对象进行序列化的时候，如果遇到不支持 Serializable 接口的对象。在此情况下，将抛出 `NotSerializableException`。

如果要序列化的类有父类，要想同时将在父类中定义过的变量持久化下来，那么父类也应该集成`java.io.Serializable`接口。

下面是一个实现了`java.io.Serializable`接口的类

    package com.hollischaung.serialization.SerializableDemos;
    import java.io.Serializable;
    /**
     * Created by hollis on 16/2/17.
     * 实现Serializable接口
     */
    public class User1 implements Serializable {
    
        private String name;
        private int age;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
    

通过下面的代码进行序列化及反序列化

    package com.hollischaung.serialization.SerializableDemos;
    
    import org.apache.commons.io.FileUtils;
    import org.apache.commons.io.IOUtils;
    
    import java.io.*;
    /**
     * Created by hollis on 16/2/17.
     * SerializableDemo1 结合SerializableDemo2说明 一个类要想被序列化必须实现Serializable接口
     */
    public class SerializableDemo1 {
    
        public static void main(String[] args) {
            //Initializes The Object
            User1 user = new User1();
            user.setName("hollis");
            user.setAge(23);
            System.out.println(user);
    
            //Write Obj to File
            ObjectOutputStream oos = null;
            try {
                oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
                oos.writeObject(user);
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                IOUtils.closeQuietly(oos);
            }
    
            //Read Obj from File
            File file = new File("tempFile");
            ObjectInputStream ois = null;
            try {
                ois = new ObjectInputStream(new FileInputStream(file));
                User1 newUser = (User1) ois.readObject();
                System.out.println(newUser);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } finally {
                IOUtils.closeQuietly(ois);
                try {
                    FileUtils.forceDelete(file);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
    
        }
    }
    
    //OutPut:
    //User{name='hollis', age=23}
    //User{name='hollis', age=23}
    


## Externalizable接口

除了Serializable 之外，java中还提供了另一个序列化接口`Externalizable`

为了了解Externalizable接口和Serializable接口的区别，先来看代码，我们把上面的代码改成使用Externalizable的形式。

    package com.hollischaung.serialization.ExternalizableDemos;
    
    import java.io.Externalizable;
    import java.io.IOException;
    import java.io.ObjectInput;
    import java.io.ObjectOutput;
    
    /**
     * Created by hollis on 16/2/17.
     * 实现Externalizable接口
     */
    public class User1 implements Externalizable {
    
        private String name;
        private int age;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public void writeExternal(ObjectOutput out) throws IOException {
    
        }
    
        public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
    
        }
    
        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
    

 

    package com.hollischaung.serialization.ExternalizableDemos;
    
    import java.io.*;
    
    /**
     * Created by hollis on 16/2/17.
     */
    public class ExternalizableDemo1 {
    
        //为了便于理解和节省篇幅，忽略关闭流操作及删除文件操作。真正编码时千万不要忘记
        //IOException直接抛出
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            //Write Obj to file
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
            User1 user = new User1();
            user.setName("hollis");
            user.setAge(23);
            oos.writeObject(user);
            //Read Obj from file
            File file = new File("tempFile");
            ObjectInputStream ois =  new ObjectInputStream(new FileInputStream(file));
            User1 newInstance = (User1) ois.readObject();
            //output
            System.out.println(newInstance);
        }
    }
    //OutPut:
    //User{name='null', age=0}
    

通过上面的实例可以发现，对User1类进行序列化及反序列化之后得到的对象的所有属性的值都变成了默认值。也就是说，之前的那个对象的状态并没有被持久化下来。这就是Externalizable接口和Serializable接口的区别：

Externalizable继承了Serializable，该接口中定义了两个抽象方法：`writeExternal()`与`readExternal()`。当使用Externalizable接口来进行序列化与反序列化的时候需要开发人员重写`writeExternal()`与`readExternal()`方法。由于上面的代码中，并没有在这两个方法中定义序列化实现细节，所以输出的内容为空。还有一点值得注意：在使用Externalizable进行序列化的时候，在读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。所以，实现Externalizable接口的类必须要提供一个public的无参的构造器。

按照要求修改之后代码如下：

    package com.hollischaung.serialization.ExternalizableDemos;
    
    import java.io.Externalizable;
    import java.io.IOException;
    import java.io.ObjectInput;
    import java.io.ObjectOutput;
    
    /**
     * Created by hollis on 16/2/17.
     * 实现Externalizable接口,并实现writeExternal和readExternal方法
     */
    public class User2 implements Externalizable {
    
        private String name;
        private int age;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public void writeExternal(ObjectOutput out) throws IOException {
            out.writeObject(name);
            out.writeInt(age);
        }
    
        public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
            name = (String) in.readObject();
            age = in.readInt();
        }
    
        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
    

 

    package com.hollischaung.serialization.ExternalizableDemos;
    
    import java.io.*;
    
    /**
     * Created by hollis on 16/2/17.
     */
    public class ExternalizableDemo2 {
    
        //为了便于理解和节省篇幅，忽略关闭流操作及删除文件操作。真正编码时千万不要忘记
        //IOException直接抛出
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            //Write Obj to file
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
            User2 user = new User2();
            user.setName("hollis");
            user.setAge(23);
            oos.writeObject(user);
            //Read Obj from file
            File file = new File("tempFile");
            ObjectInputStream ois =  new ObjectInputStream(new FileInputStream(file));
            User2 newInstance = (User2) ois.readObject();
            //output
            System.out.println(newInstance);
        }
    }
    //OutPut:
    //User{name='hollis', age=23}
    

这次，就可以把之前的对象状态持久化下来了。

> 如果User类中没有无参数的构造函数，在运行时会抛出异常：`java.io.InvalidClassException`


## 参考资料

[维基百科][6]

[理解Java对象序列化][7]

[Java 序列化的高级认识][8]

## 推荐阅读

[深入分析Java的序列化与反序列化][4]

[单例与序列化的那些事儿][5]

 [1]: http://www.hollischuang.com/archives/1140#What%20Serializable%20Did
 [2]: https://github.com/hollischuang/java-demo/tree/master/src/main/java/com/hollischaung/serialization/SerializableDemos
 [3]: https://github.com/hollischuang/java-demo/tree/master/src/main/java/com/hollischaung/serialization/ExternalizableDemos
 [4]: http://www.hollischuang.com/archives/1140
 [5]: http://www.hollischuang.com/archives/1144
 [6]: https://zh.wikipedia.org/wiki/序列化
 [7]: http://www.blogjava.net/jiangshachina/archive/2012/02/13/369898.html
 [8]: https://www.ibm.com/developerworks/cn/java/j-lo-serial/