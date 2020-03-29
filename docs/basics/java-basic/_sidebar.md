* 基本数据类型

    * [8种基本数据类型](/basics/java-basic/basic-data-types.md)

    * [整型中byte、short、int、long的取值范围](/basics/java-basic/integer-scope.md)

    * [什么是浮点型？](/basics/java-basic/float.md)

    * [什么是单精度和双精度？](/basics/java-basic/single-double-float.md)

    * [为什么不能用浮点型表示金额？](/basics/java-basic/float-amount.md)

* 自动拆装箱

    * [自动拆装箱](/basics/java-basic/boxing-unboxing.md)

    * [Integer的缓存机制](/basics/java-basic/integer-cache.md)

* String

    * [字符串的不可变性](/basics/java-basic/final-string.md)

    * [JDK 6和JDK 7中substring的原理及区别](/basics/java-basic/substring.md)

    * replaceFirst、replaceAll、replace区别

    * [String对“+”的重载](/basics/java-basic/string-append.md)

    * [字符串拼接的几种方式和区别](/basics/java-basic/string-concat.md)

    * [String.valueOf和Integer.toString的区别](/basics/java-basic/value-of-vs-to-string.md)

    * [switch对String的支持](/basics/java-basic/switch-string.md)
    * 字符串池
    * 常量池（运行时常量池、Class常量池）
    * intern
* Java中各种关键字
    * transient
    * instanceof
    * volatile
    * synchronized
    * final
    * static
    * const 
* 集合类

    * 常用集合类的使用
    
    * [ArrayList和LinkedList和Vector的区别](/basics/java-basic/arraylist-vs-linkedlist-vs-vector.md) 
    
    * [SynchronizedList和Vector的区别](/basics/java-basic/synchronizedlist-vector.md)
    
    * [HashMap、HashTable、ConcurrentHashMap区别](/basics/java-basic/HashMap-HashTable-ConcurrentHashMap.md)
    
    * [Set和List区别？](/basics/java-basic/set-vs-list.md)
    
    * [Set如何保证元素不重复?](/basics/java-basic/set-repetition.md)
    
    * [Java 8中stream相关用法](/basics/java-basic/stream.md)
    
    * Apache集合处理工具类的使用
    
    * 不同版本的JDK中HashMap的实现的区别以及原因
    
    * [Collection和Collections区别](/basics/java-basic/Collection-vs-Collections.md)
    
    * [Arrays.asList获得的List使用时需要注意什么](/basics/java-basic/Arrays-asList.md)
    
    * [Enumeration和Iterator区别](/basics/java-basic/Enumeration-vs-Iterator.md)
    
    * [fail-fast 和 fail-safe](/basics/java-basic/fail-fast-vs-fail-safe.md)
    
    * [CopyOnWriteArrayList](/basics/java-basic/CopyOnWriteArrayList.md)
    
    * [ConcurrentSkipListMap](/basics/java-basic/ConcurrentSkipListMap.md)

* 枚举

    * [枚举的用法](/basics/java-basic/enum-usage.md)
    
    * [枚举的实现](/basics/java-basic/enum-impl.md)
    
    * [枚举与单例](/basics/java-basic/enum-singleton.md)
    
    * Enum类
    
    * [Java枚举如何比较](/basics/java-basic/enum-compare.md)
    
    * [switch对枚举的支持](/basics/java-basic/enum-switch.md)
    
    * [枚举的序列化如何实现](/basics/java-basic/enum-serializable.md)
    
    * [枚举的线程安全性问题](/basics/java-basic/enum-thread-safe.md)
    
* IO
    
    * [字符流、字节流](/basics/java-basic/byte-stream-vs-character-stream.md)
    
    * [输入流、输出流](/basics/java-basic/input-stream-vs-output-stream.md)
    
    * [同步、异步](/basics/java-basic/synchronized-vs-asynchronization.md)
    
    * [阻塞、非阻塞](/basics/java-basic/block-vs-non-blocking.md)
    
    * [Linux 5种IO模型](/basics/java-basic/linux-io.md)
    
    * [BIO、NIO和AIO的区别、三种IO的用法与原理](/basics/java-basic/bio-vs-nio-vs-aio.md)
    
    * netty
    
* 反射

    * [反射](/basics/java-basic/reflection.md)与工厂模式、 

    * [反射有什么作用](/basics/java-basic/usage-of-reflection.md)

    * [Class类](/basics/java-basic/Class.md)

    * `java.lang.reflect.*`
    
* 动态代理
    
    * [静态代理](/basics/java-basic/static-proxy.md)
    
    * [动态代理](/basics/java-basic/dynamic-proxy.md)
    
    * [动态代理和反射的关系](/basics/java-basic/dynamic-proxy-vs-reflection.md)
    
    * [动态代理的几种实现方式](/basics/java-basic/dynamic-proxy-implementation.md)
    
    * [AOP](/basics/java-basic/aop-vs-proxy.md)
   
* 序列化
   
   * [什么是序列化与反序列化](basics/java-basic/serialize.md)
   * [Java如何实现序列化与反序列化](basics/java-basic/serialize-in-java.md)
   * [Serializable 和 Externalizable 有何不同](basics/java-basic/diff-serializable-vs-externalizable.md)
   * 为什么序列化
   * [serialVersionUID](basics/java-basic/serialVersionUID.md)
   * [为什么serialVersionUID不能随便改](basics/java-basic/serialVersionUID-modify.md)
   * [transient](basics/java-basic/transient.md)
   * [序列化底层原理](basics/java-basic/serialize-principle.md)
   * [序列化与单例模式](basics/java-basic/serialize-singleton.md)
   * [protobuf](basics/java-basic/protobuf.md)
   * 为什么说序列化并不安全
   
* 注解
   
   * [元注解](/basics/java-basic/meta-annotation.md)
   * [自定义注解](/basics/java-basic/custom-annotation.md)
   * Java中常用注解使用
   * 注解与反射的结合
   
   *  [如何自定义一个注解？](/basics/java-basic/create-annotation.md)
   
   * [Spring常用注解](/basics/java-basic/annotation-in-spring.md)
* JMS

    * 什么是Java消息服务
    * JMS消息传送模型
    
* JMX
    
    * `java.lang.management.*`
    * `javax.management.*`
    
* 泛型
    
    * [什么是泛型](/basics/java-basic/generics.md)
    * [类型擦除](/basics/java-basic/type-erasue.md)
    * [泛型带来的问题](/basics/java-basic/generics-problem.md)
    * [泛型中K T V E ？ object等的含义](/basics/java-basic/k-t-v-e.md)
    * 泛型各种用法
    
    * [限定通配符和非限定通配符](/basics/java-basic/Wildcard-Character.md)
    * [上下界限定符extends 和 super](/basics/java-basic/extends-vs-super.md)
    
    * [`List<Object>`和原始类型`List`之间的区别?](/basics/java-basic/genericity-list.md)
    
    * [`List<?>`和`List<Object>`之间的区别是什么?](/basics/java-basic/genericity-list-wildcard.md)
    
* 单元测试
    
    * junit
    * mock
    * mockito
    * 内存数据库（h2）
    
* 正则表达式
    
    * `java.lang.util.regex.*`
    
* 常用的Java工具库
    
    * `commons.lang`
    * `commons.*...` 
    * `guava-libraries` 
    * `netty`
    
* API&SPI
    
    * API
    * [API和SPI的关系和区别](/basics/java-basic/api-vs-spi.md)
    
    * [如何定义SPI](/basics/java-basic/create-spi.md)
    * [SPI的实现原理](/basics/java-basic/spi-principle.md)
    
* 异常
    
    * [Error和Exception](/basics/java-basic/error-vs-exception.md)
    * [异常类型](/basics/java-basic/exception-type.md)
    * [异常相关关键字](/basics/java-basic/keyword-about-exception.md)
    * [正确处理异常](/basics/java-basic/handle-exception.md)
    * [自定义异常](/basics/java-basic/define-exception.md)
    * [异常链](/basics/java-basic/exception-chain.md)
    * [try-with-resources](/basics/java-basic/try-with-resources.md)
    * [finally和return的执行顺序](/basics/java-basic/order-about-finllly-return.md)
    
* 时间处理
    
    * [时区](/basics/java-basic/time-zone.md)
    * [冬令时和夏令时](/basics/java-basic/StandardTime-vs-daylightSavingTime.md)
    * [时间戳](/basics/java-basic/timestamp.md)
    * Java中时间API
    
    * [格林威治时间](/basics/java-basic/GMT.md)
    * [CET,UTC,GMT,CST几种常见时间的含义和关系](/basics/java-basic/CET-UTC-GMT-CST.md)
    
    * [SimpleDateFormat的线程安全性问题](/basics/java-basic/simpledateformat-thread-safe.md)
    
    * [Java 8中的时间处理](/basics/java-basic/time-in-java8.md)
    
    * [如何在东八区的计算机上获取美国时间](/basics/java-basic/get-los_angeles-time.md)
    
    * [yyyy和YYYY有什么区别？](/basics/java-basic/YYYY-vs-yyyy.md)
    
* 编码方式
    
    * [什么是ASCII？](/basics/java-basic/ASCII.md)
    
    * [Unicode](/basics/java-basic/UNICODE.md)
    
    * [有了Unicode为啥还需要UTF-8](/basics/java-basic/why-utf8.md)
    
    * [UTF8、UTF16、UTF32区别](/basics/java-basic/UTF8-UTF16-UTF32.md)
    
    * 有了UTF8为什么还需要GBK？
    
    * [GBK、GB2312、GB18030之间的区别](/basics/java-basic/gbk-gb2312-gb18030.md)
    
    * [URL编解码](/basics/java-basic/url-encode.md)
    * [Big Endian和Little Endian](/basics/java-basic/big-endian-vs-little-endian.md)
    
    * 如何解决乱码问题
    
* 语法糖
    
    *  [Java中语法糖原理、解语法糖](/basics/java-basic/syntactic-sugar.md)
    
    *  [语法糖介绍](/basics/java-basic/syntactic-sugar.md)
    
* 阅读源代码
    
    * String
    * Integer
    * Long
    * Enum
    * BigDecimal
    * ThreadLocal
    * ClassLoader & URLClassLoader
    * ArrayList & LinkedList
    * HashMap & LinkedHashMap & TreeMap & CouncurrentHashMap
    * HashSet & LinkedHashSet & TreeSet
    