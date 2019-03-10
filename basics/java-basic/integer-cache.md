英文原文：[Java Integer Cache][1] 翻译地址：[Java中整型的缓存机制][2] 原文作者：[Java Papers][3] 翻译作者：[Hollis][4] 转载请注明出处。

本文将介绍Java中Integer的缓存相关知识。这是在Java 5中引入的一个有助于节省内存、提高性能的功能。首先看一个使用Integer的示例代码，从中学习其缓存行为。接着我们将为什么这么实现以及他到底是如何实现的。你能猜出下面的Java程序的输出结果吗。如果你的结果和真正结果不一样，那么你就要好好看看本文了。

    package com.javapapers.java;
    
    public class JavaIntegerCache {
        public static void main(String... strings) {
    
            Integer integer1 = 3;
            Integer integer2 = 3;
    
            if (integer1 == integer2)
                System.out.println("integer1 == integer2");
            else
                System.out.println("integer1 != integer2");
    
            Integer integer3 = 300;
            Integer integer4 = 300;
    
            if (integer3 == integer4)
                System.out.println("integer3 == integer4");
            else
                System.out.println("integer3 != integer4");
    
        }
    }
    

我们普遍认为上面的两个判断的结果都是false。虽然比较的值是相等的，但是由于比较的是对象，而对象的引用不一样，所以会认为两个if判断都是false的。在Java中，`==`比较的是对象应用，而`equals`比较的是值。所以，在这个例子中，不同的对象有不同的引用，所以在进行比较的时候都将返回false。奇怪的是，这里两个类似的if条件判断返回不同的布尔值。

上面这段代码真正的输出结果：

    integer1 == integer2
    integer3 != integer4
    

## Java中Integer的缓存实现

在Java 5中，在Integer的操作上引入了一个新功能来节省内存和提高性能。整型对象通过使用相同的对象引用实现了缓存和重用。

> 适用于整数值区间-128 至 +127。
> 
> 只适用于自动装箱。使用构造函数创建对象不适用。

Java的编译器把基本数据类型自动转换成封装类对象的过程叫做`自动装箱`，相当于使用`valueOf`方法：

    Integer a = 10; //this is autoboxing
    Integer b = Integer.valueOf(10); //under the hood
    

现在我们知道了这种机制在源码中哪里使用了，那么接下来我们就看看JDK中的`valueOf`方法。下面是`JDK 1.8.0 build 25`的实现：

    /**
         * Returns an {@code Integer} instance representing the specified
         * {@code int} value.  If a new {@code Integer} instance is not
         * required, this method should generally be used in preference to
         * the constructor {@link #Integer(int)}, as this method is likely
         * to yield significantly better space and time performance by
         * caching frequently requested values.
         *
         * This method will always cache values in the range -128 to 127,
         * inclusive, and may cache other values outside of this range.
         *
         * @param  i an {@code int} value.
         * @return an {@code Integer} instance representing {@code i}.
         * @since  1.5
         */
        public static Integer valueOf(int i) {
            if (i >= IntegerCache.low && i <= IntegerCache.high)
                return IntegerCache.cache[i + (-IntegerCache.low)];
            return new Integer(i);
        }
    

在创建对象之前先从IntegerCache.cache中寻找。如果没找到才使用new新建对象。

## IntegerCache Class

IntegerCache是Integer类中定义的一个`private static`的内部类。接下来看看他的定义。

      /**
         * Cache to support the object identity semantics of autoboxing for values between
         * -128 and 127 (inclusive) as required by JLS.
         *
         * The cache is initialized on first usage.  The size of the cache
         * may be controlled by the {@code -XX:AutoBoxCacheMax=} option.
         * During VM initialization, java.lang.Integer.IntegerCache.high property
         * may be set and saved in the private system properties in the
         * sun.misc.VM class.
         */
    
        private static class IntegerCache {
            static final int low = -128;
            static final int high;
            static final Integer cache[];
    
            static {
                // high value may be configured by property
                int h = 127;
                String integerCacheHighPropValue =
                    sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
                if (integerCacheHighPropValue != null) {
                    try {
                        int i = parseInt(integerCacheHighPropValue);
                        i = Math.max(i, 127);
                        // Maximum array size is Integer.MAX_VALUE
                        h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                    } catch( NumberFormatException nfe) {
                        // If the property cannot be parsed into an int, ignore it.
                    }
                }
                high = h;
    
                cache = new Integer[(high - low) + 1];
                int j = low;
                for(int k = 0; k < cache.length; k++)
                    cache[k] = new Integer(j++);
    
                // range [-128, 127] must be interned (JLS7 5.1.7)
                assert IntegerCache.high >= 127;
            }
    
            private IntegerCache() {}
        }
    

其中的javadoc详细的说明了缓存支持-128到127之间的自动装箱过程。最大值127可以通过`-XX:AutoBoxCacheMax=size`修改。 缓存通过一个for循环实现。从低到高并创建尽可能多的整数并存储在一个整数数组中。这个缓存会在Integer类第一次被使用的时候被初始化出来。以后，就可以使用缓存中包含的实例对象，而不是创建一个新的实例(在自动装箱的情况下)。

实际上这个功能在Java 5中引入的时候,范围是固定的-128 至 +127。后来在Java 6中，可以通过`java.lang.Integer.IntegerCache.high`设置最大值。这使我们可以根据应用程序的实际情况灵活地调整来提高性能。到底是什么原因选择这个-128到127范围呢？因为这个范围的数字是最被广泛使用的。 在程序中，第一次使用Integer的时候也需要一定的额外时间来初始化这个缓存。

## Java语言规范中的缓存行为

在[Boxing Conversion][5]部分的Java语言规范(JLS)规定如下：

> 如果一个变量p的值是：
> 
> -128至127之间的整数(§3.10.1)
> 
> true 和 false的布尔值 (§3.10.3)
> 
> ‘\u0000’至 ‘\u007f’之间的字符(§3.10.4)
> 
> 中时，将p包装成a和b两个对象时，可以直接使用a==b判断a和b的值是否相等。

## 其他缓存的对象

这种缓存行为不仅适用于Integer对象。我们针对所有的整数类型的类都有类似的缓存机制。

> 有ByteCache用于缓存Byte对象
> 
> 有ShortCache用于缓存Short对象
> 
> 有LongCache用于缓存Long对象
> 
> 有CharacterCache用于缓存Character对象

`Byte`, `Short`, `Long`有固定范围: -128 到 127。对于`Character`, 范围是 0 到 127。除了`Integer`以外，这个范围都不能改变。

 [1]: http://javapapers.com/java/java-integer-cache/
 [2]: http://www.hollischuang.com/?p=1174
 [3]: http://javapapers.com/
 [4]: http://www.hollischuang.com
 [5]: http://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.7