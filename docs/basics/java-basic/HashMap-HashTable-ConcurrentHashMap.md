### HashMap和HashTable有何不同？


线程安全：

HashTable 中的方法是同步的，而HashMap中的方法在默认情况下是非同步的。在多线程并发的环境下，可以直接使用HashTable，但是要使用HashMap的话就要自己增加同步处理了。

继承关系：
HashTable是基于陈旧的Dictionary类继承来的。
HashMap继承的抽象类AbstractMap实现了Map接口。


允不允许null值：
HashTable中，key和value都不允许出现null值，否则会抛出NullPointerException异常。
HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。


默认初始容量和扩容机制：
HashTable中的hash数组初始大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。原因参考全网把Map中的hash()分析的最透彻的文章，别无二家。-HollisChuang's Blog

哈希值的使用不同 ：
HashTable直接使用对象的hashCode。
HashMap重新计算hash值。


遍历方式的内部实现上不同 ：
Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。
HashMap 实现 Iterator，支持fast-fail，Hashtable的 Iterator 遍历支持fast-fail，用 Enumeration 不支持 fast-fail

### HashMap 和 ConcurrentHashMap 的区别？

ConcurrentHashMap和HashMap的实现方式不一样，虽然都是使用桶数组实现的，但是还是有区别，ConcurrentHashMap对桶数组进行了分段，而HashMap并没有。


ConcurrentHashMap在每一个分段上都用锁进行了保护。HashMap没有锁机制。所以，前者线程安全的，后者不是线程安全的。

PS：以上区别基于jdk1.8以前的版本。