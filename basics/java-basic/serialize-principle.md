## 序列化与反序列化

序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。一般将一个对象存储至一个储存媒介，例如档案或是记亿体缓冲等。在网络传输过程中，可以是字节或是XML等格式。而字节的或XML编码格式可以还原完全相等的对象。这个相反的过程又称为反序列化。

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
    

更多关于Serializable的使用，请参考[代码实例][2]

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

更多Externalizable接口使用实例请参考[代码实例][3]

## ObjectOutput和ObjectInput 接口

ObjectInput接口 扩展自 DataInput 接口以包含对象的读操作。

> DataInput 接口用于从二进制流中读取字节，并根据所有 Java 基本类型数据进行重构。同时还提供根据 UTF-8 修改版格式的数据重构 String 的工具。
> 
> 对于此接口中的所有数据读取例程来说，如果在读取所需字节数之前已经到达文件末尾 (end of file)，则将抛出 EOFException（IOException 的一种）。如果因为到达文件末尾以外的其他原因无法读取字节，则将抛出 IOException 而不是 EOFException。尤其是，在输入流已关闭的情况下，将抛出 IOException。

ObjectOutput 扩展 DataOutput 接口以包含对象的写入操作。

> DataOutput 接口用于将数据从任意 Java 基本类型转换为一系列字节，并将这些字节写入二进制流。同时还提供了一个将 String 转换成 UTF-8 修改版格式并写入所得到的系列字节的工具。
> 
> 对于此接口中写入字节的所有方法，如果由于某种原因无法写入某个字节，则抛出 IOException。

## ObjectOutputStream类和ObjectInputStream类

通过前面的代码片段中我们也能知道，我们一般使用ObjectOutputStream的`writeObject`方法把一个对象进行持久化。再使用ObjectInputStream的`readObject`从持久化存储中把对象读取出来。

更多关于ObjectInputStream和ObjectOutputStream的相关知识欢迎阅读我的另外两篇博文：[深入分析Java的序列化与反序列化][4]、[单例与序列化的那些事儿][5]

## Transient 关键字

Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。关于Transient 关键字的拓展知识欢迎阅读[深入分析Java的序列化与反序列化][4]

## 序列化ID

虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（就是 `private static final long serialVersionUID`)

序列化 ID 在 Eclipse 下提供了两种生成策略，一个是固定的 1L，一个是随机生成一个不重复的 long 类型数据（实际上是使用 JDK 工具生成），在这里有一个建议，如果没有特殊需求，就是用默认的 1L 就可以，这样可以确保代码一致时反序列化成功。那么随机生成的序列化 ID 有什么作用呢，有些时候，通过改变序列化 ID 可以用来限制某些用户的使用。

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