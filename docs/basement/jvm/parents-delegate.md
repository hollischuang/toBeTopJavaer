虚拟机在加载类的过程中需要使用类加载器进行加载，而在Java中，类加载器有很多，那么当JVM想要加载一个.class文件的时候，到底应该由哪个类加载器加载呢？

这就不得不提到”双亲委派机制”。

首先，我们需要知道的是，Java语言系统中支持以下4种类加载器：


* Bootstrap ClassLoader 启动类加载器
* Extention ClassLoader 标准扩展类加载器
* Application ClassLoader 应用类加载器
* User ClassLoader 用户自定义类加载器

这四种类加载器之间，是存在着一种层次关系的，如下图

![](https://www.hollischuang.com/wp-content/uploads/2021/01/16102749464329.jpg)

一般认为上一层加载器是下一层加载器的父加载器，那么，除了BootstrapClassLoader之外，所有的加载器都是有父加载器的。

那么，所谓的双亲委派机制，指的就是：当一个类加载器收到了类加载的请求的时候，他不会直接去加载指定的类，而是把这个请求委托给自己的父加载器去加载。只有父加载器无法加载这个类的时候，才会由当前这个加载器来负责类的加载。

那么，什么情况下父加载器会无法加载某一个类呢？

其实，Java中提供的这四种类型的加载器，是有各自的职责的：

* Bootstrap ClassLoader ，主要负责加载Java核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等。
* Extention ClassLoader，主要负责加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件。
* Application ClassLoader ，主要负责加载当前应用的classpath下的所有类
* User ClassLoader ， 用户自定义的类加载器,可加载指定路径的class文件

那么也就是说，一个用户自定义的类，如com.hollis.ClassHollis 是无论如何也不会被Bootstrap和Extention加载器加载的。