很多人在通过阅读源码的方式学习Java，这是个很好的方式。而JDK的源码自然是首选。在JDK的众多类中，我觉得HashMap及其相关的类是设计的比较好的。很多人读过HashMap的代码，不知道你们有没有和我一样，觉得HashMap中关于容量相关的参数定义的太多了，傻傻分不清楚。

其实，这篇文章介绍的内容比较简单，只要认真的看看HashMap的原理还是可以理解的，单独写一篇文章的原因是因为我后面还有几篇关于HashMap源码分析的文章，这些概念不熟悉的话阅读后面的文章会很吃力。

先来看一下，HashMap中都定义了哪些成员变量。

[<img src="http://www.hollischuang.com/wp-content/uploads/2018/05/paramInMap.png" alt="paramInMap" width="523" height="288" class="aligncenter size-full wp-image-2424" />][1]

上面是一张HashMap中主要的成员变量的图，其中有一个是我们本文主要关注的： `size`、`loadFactor`、`threshold`、`DEFAULT_LOAD_FACTOR`和`DEFAULT_INITIAL_CAPACITY`。

我们先来简单解释一下这些参数的含义，然后再分析他们的作用。

HashMap类中有以下主要成员变量：

*   transient int size; 
    *   记录了Map中KV对的个数
*   loadFactor 
    *   装载因子，用来衡量HashMap满的程度。loadFactor的默认值为0.75f（`static final float DEFAULT_LOAD_FACTOR = 0.75f;`）。
*   int threshold; 
    *   临界值，当实际KV个数超过threshold时，HashMap会将容量扩容，threshold＝容量*装载因子
*   除了以上这些重要成员变量外，HashMap中还有一个和他们紧密相关的概念：capacity 
    *   容量，如果不指定，默认容量是16(`static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;`)

可能看完了你还是有点蒙，size和capacity之间有啥关系？为啥要定义这两个变量。loadFactor和threshold又是干啥的？

### size 和 capacity

HashMap中的size和capacity之间的区别其实解释起来也挺简单的。我们知道，HashMap就像一个“桶”，那么capacity就是这个桶“当前”最多可以装多少元素，而size表示这个桶已经装了多少元素。来看下以下代码：

        Map<String, String> map = new HashMap<String, String>();
        map.put("hollis", "hollischuang");
    
        Class<?> mapType = map.getClass();
        Method capacity = mapType.getDeclaredMethod("capacity");
        capacity.setAccessible(true);
        System.out.println("capacity : " + capacity.invoke(map));
    
        Field size = mapType.getDeclaredField("size");
        size.setAccessible(true);
        System.out.println("size : " + size.get(map));
    

我们定义了一个新的HashMap，并想其中put了一个元素，然后通过反射的方式打印capacity和size。输出结果为：**capacity : 16、size : 1**

默认情况下，一个HashMap的容量（capacity）是16，设计成16的好处我在《[全网把Map中的hash()分析的最透彻的文章，别无二家。][2]》中也简单介绍过，主要是可以使用按位与替代取模来提升hash的效率。

为什么我刚刚说capacity就是这个桶“当前”最多可以装多少元素呢？当前怎么理解呢。其实，HashMap是具有扩容机制的。在一个HashMap第一次初始化的时候，默认情况下他的容量是16，当达到扩容条件的时候，就需要进行扩容了，会从16扩容成32。

我们知道，HashMap的重载的构造函数中，有一个是支持传入initialCapacity的，那么我们尝试着设置一下，看结果如何。

        Map<String, String> map = new HashMap<String, String>(1);
    
        Class<?> mapType = map.getClass();
        Method capacity = mapType.getDeclaredMethod("capacity");
        capacity.setAccessible(true);
        System.out.println("capacity : " + capacity.invoke(map));
    
        Map<String, String> map = new HashMap<String, String>(7);
    
        Class<?> mapType = map.getClass();
        Method capacity = mapType.getDeclaredMethod("capacity");
        capacity.setAccessible(true);
        System.out.println("capacity : " + capacity.invoke(map));
    
    
        Map<String, String> map = new HashMap<String, String>(9);
    
        Class<?> mapType = map.getClass();
        Method capacity = mapType.getDeclaredMethod("capacity");
        capacity.setAccessible(true);
        System.out.println("capacity : " + capacity.invoke(map));
    

分别执行以上3段代码，分别输出：**capacity : 2、capacity : 8、capacity : 16**。

也就是说，默认情况下HashMap的容量是16，但是，如果用户通过构造函数指定了一个数字作为容量，那么Hash会选择大于该数字的第一个2的幂作为容量。(1->1、7->8、9->16)

> 这里有一个小建议：在初始化HashMap的时候，应该尽量指定其大小。尤其是当你已知map中存放的元素个数时。（《阿里巴巴Java开发规约》）

### loadFactor 和 threshold

前面我们提到过，HashMap有扩容机制，就是当达到扩容条件时会进行扩容，从16扩容到32、64、128...

那么，这个扩容条件指的是什么呢？

其实，HashMap的扩容条件就是当HashMap中的元素个数（size）超过临界值（threshold）时就会自动扩容。

在HashMap中，threshold = loadFactor * capacity。

loadFactor是装载因子，表示HashMap满的程度，默认值为0.75f，设置成0.75有一个好处，那就是0.75正好是3/4，而capacity又是2的幂。所以，两个数的乘积都是整数。

对于一个默认的HashMap来说，默认情况下，当其size大于12(16*0.75)时就会触发扩容。

验证代码如下：

        Map<String, String> map = new HashMap<>();
        map.put("hollis1", "hollischuang");
        map.put("hollis2", "hollischuang");
        map.put("hollis3", "hollischuang");
        map.put("hollis4", "hollischuang");
        map.put("hollis5", "hollischuang");
        map.put("hollis6", "hollischuang");
        map.put("hollis7", "hollischuang");
        map.put("hollis8", "hollischuang");
        map.put("hollis9", "hollischuang");
        map.put("hollis10", "hollischuang");
        map.put("hollis11", "hollischuang");
        map.put("hollis12", "hollischuang");
        Class<?> mapType = map.getClass();
    
        Method capacity = mapType.getDeclaredMethod("capacity");
        capacity.setAccessible(true);
        System.out.println("capacity : " + capacity.invoke(map));
    
        Field size = mapType.getDeclaredField("size");
        size.setAccessible(true);
        System.out.println("size : " + size.get(map));
    
        Field threshold = mapType.getDeclaredField("threshold");
        threshold.setAccessible(true);
        System.out.println("threshold : " + threshold.get(map));
    
        Field loadFactor = mapType.getDeclaredField("loadFactor");
        loadFactor.setAccessible(true);
        System.out.println("loadFactor : " + loadFactor.get(map));
    
        map.put("hollis13", "hollischuang");
        Method capacity = mapType.getDeclaredMethod("capacity");
        capacity.setAccessible(true);
        System.out.println("capacity : " + capacity.invoke(map));
    
        Field size = mapType.getDeclaredField("size");
        size.setAccessible(true);
        System.out.println("size : " + size.get(map));
    
        Field threshold = mapType.getDeclaredField("threshold");
        threshold.setAccessible(true);
        System.out.println("threshold : " + threshold.get(map));
    
        Field loadFactor = mapType.getDeclaredField("loadFactor");
        loadFactor.setAccessible(true);
        System.out.println("loadFactor : " + loadFactor.get(map));
    

输出结果：

    capacity : 16
    size : 12
    threshold : 12
    loadFactor : 0.75
    
    capacity : 32
    size : 13
    threshold : 24
    loadFactor : 0.75
    

当HashMap中的元素个数达到13的时候，capacity就从16扩容到32了。

HashMap中还提供了一个支持传入initialCapacity,loadFactor两个参数的方法，来初始化容量和装载因子。不过，一般不建议修改loadFactor的值。

### 总结

HashMap中size表示当前共有多少个KV对，capacity表示当前HashMap的容量是多少，默认值是16，每次扩容都是成倍的。loadFactor是装载因子，当Map中元素个数超过`loadFactor* capacity`的值时，会触发扩容。`loadFactor* capacity`可以用threshold表示。

PS：文中分析基于JDK1.8.0_73

 [1]: http://www.hollischuang.com/wp-content/uploads/2018/05/paramInMap.png
 [2]: http://www.hollischuang.com/archives/2091
