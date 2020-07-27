ConcurrentSkipListMap是一个内部使用跳表，并且支持排序和并发的一个Map，是线程安全的。一般很少会被用到，也是一个比较偏门的数据结构。

简单介绍下跳表

    跳表是一种允许在一个有顺序的序列中进行快速查询的数据结构。

    在普通的顺序链表中查询一个元素，需要从链表头部开始一个一个节点进行遍历，然后找到节点。如图1。

    跳表可以解决这种查询时间过长，其元素遍历的图示如图2，跳表是一种使用”空间换时间”的概念用来提高查询效率的链表。



ConcurrentSkipListMap 和 ConcurrentHashMap 的主要区别：
a.底层实现方式不同。ConcurrentSkipListMap底层基于跳表。ConcurrentHashMap底层基于Hash桶和红黑树。
b.ConcurrentHashMap不支持排序。ConcurrentSkipListMap支持排序。
