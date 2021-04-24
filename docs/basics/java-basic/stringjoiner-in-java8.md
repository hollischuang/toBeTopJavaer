在上一节中，我们介绍了几种Java中字符串拼接的方式，以及优缺点。其中还有一个重要的拼接方式我没有介绍，那就是Java 8中提供的StringJoiner ，本文就来介绍一下这个字符串拼接的新兵。

如果你想知道一共有多少种方法可以进行字符串拼接，教你一个简单的办法，在Intellij IDEA中，定义一个Java Bean，然后尝试使用快捷键自动生成一个toString方法，IDEA会提示多种toString生成策略可供选择。

![][2]￼

目前我使用的IDEA的toString生成策略默认的是使用JDK 1.8提供的StringJoiner。

### 介绍

StringJoiner是java.util包中的一个类，用于构造一个由分隔符分隔的字符序列（可选），并且可以从提供的前缀开始并以提供的后缀结尾。虽然这也可以在StringBuilder类的帮助下在每个字符串之后附加分隔符，但StringJoiner提供了简单的方法来实现，而无需编写大量代码。

StringJoiner类共有2个构造函数，5个公有方法。其中最常用的方法就是add方法和toString方法，类似于StringBuilder中的append方法和toString方法。

### 用法

StringJoiner的用法比较简单，下面的代码中，我们使用StringJoiner进行了字符串拼接。

    public class StringJoinerTest {
    
        public static void main(String[] args) {
            StringJoiner sj = new StringJoiner("Hollis");
    
            sj.add("hollischuang");
            sj.add("Java干货");
            System.out.println(sj.toString());
    
            StringJoiner sj1 = new StringJoiner(":","[","]");
    
            sj1.add("Hollis").add("hollischuang").add("Java干货");
            System.out.println(sj1.toString());
        }
    }
    

以上代码输出结果：

    hollischuangHollisJava干货
    [Hollis:hollischuang:Java干货]
    

值得注意的是，当我们`StringJoiner(CharSequence delimiter)`初始化一个`StringJoiner`的时候，这个`delimiter`其实是分隔符，并不是可变字符串的初始值。

`StringJoiner(CharSequence delimiter,CharSequence prefix,CharSequence suffix)`的第二个和第三个参数分别是拼接后的字符串的前缀和后缀。

### 原理

介绍了简单的用法之后，我们再来看看这个StringJoiner的原理，看看他到底是如何实现的。主要看一下add方法：

    public StringJoiner add(CharSequence newElement) {
        prepareBuilder().append(newElement);
        return this;
    }
    
    private StringBuilder prepareBuilder() {
        if (value != null) {
            value.append(delimiter);
        } else {
            value = new StringBuilder().append(prefix);
        }
        return value;
    }
    

看到了一个熟悉的身影——StringBuilder ，没错，StringJoiner其实就是依赖StringBuilder实现的。

当我们发现StringJoiner其实是通过StringBuilder实现之后，我们大概就可以猜到，**他的性能损耗应该和直接使用StringBuilder差不多**！

### 为什么需要StringJoiner

在了解了StringJoiner的用法和原理后，可能很多读者就会产生一个疑问，明明已经有一个StringBuilder了，为什么Java 8中还要定义一个StringJoiner呢？到底有什么好处呢？

如果读者足够了解Java 8的话，或许可以猜出个大概，这肯定和Stream有关。

作者也在[Java doc][3]中找到了答案：

> A StringJoiner may be employed to create formatted output from a Stream using Collectors.joining(CharSequence)

试想，在Java中，如果我们有这样一个List：

    List<String> list = ImmutableList.of("Hollis","hollischuang","Java干货");
    

如果我们想要把他拼接成一个以下形式的字符串：

    Hollis,hollischuang,Java干货
    

可以通过以下方式：

    StringBuilder builder = new StringBuilder();
    
    if (!list.isEmpty()) {
        builder.append(list.get(0));
        for (int i = 1, n = list.size(); i < n; i++) {
            builder.append(",").append(list.get(i));
        }
    }
    builder.toString();
    

还可以使用：

    list.stream().reduce(new StringBuilder(), (sb, s) -> sb.append(s).append(','), StringBuilder::append).toString();
    

但是输出结果稍有些不同，需要进行二次处理：

    Hollis,hollischuang,Java干货,
    

还可以使用"+"进行拼接：

    list.stream().reduce((a,b)->a + "," + b).toString();
    

以上几种方式，要么是代码复杂，要么是性能不高，或者无法直接得到想要的结果。

为了满足类似这样的需求，Java 8中提供的StringJoiner就派上用场了。以上需求只需要一行代码：

    list.stream().collect(Collectors.joining(":"))
    

即可。上面用的表达式中，Collectors.joining的源代码如下：

    public static Collector<CharSequence, ?, String> joining(CharSequence delimiter,
                                                             CharSequence prefix,
                                                             CharSequence suffix) {
        return new CollectorImpl<>(
                () -> new StringJoiner(delimiter, prefix, suffix),
                StringJoiner::add, StringJoiner::merge,
                StringJoiner::toString, CH_NOID);
    }
    

其实现原理就是借助了StringJoiner。

当然，或许在`Collector`中直接使用`StringBuilder`似乎也可以实现类似的功能，只不过稍微麻烦一些。所以，Java 8中提供了`StringJoiner`来丰富`Stream`的用法。

而且`StringJoiner`也可以方便的增加前缀和后缀，比如我们希望得到的字符串是`[Hollis,hollischuang,Java干货]`而不是`Hollis,hollischuang,Java`干货的话，StringJoiner的优势就更加明显了。

### 总结

本文介绍了Java 8中提供的可变字符串类——StringJoiner，可以用于字符串拼接。

StringJoiner其实是通过StringBuilder实现的，所以他的性能和StringBuilder差不多，他也是非线程安全的。

如果日常开发中中，需要进行字符串拼接，如何选择？

1、如果只是简单的字符串拼接，考虑直接使用"+"即可。

2、如果是在for循环中进行字符串拼接，考虑使用`StringBuilder`和`StringBuffer`。

3、如果是通过一个`List`进行字符串拼接，则考虑使用`StringJoiner`。

 [1]: http://www.hollischuang.com/archives/3186
 [2]: http://www.hollischuang.com/wp-content/uploads/2019/02/15508994967943.jpg
 [3]: https://docs.oracle.com/javase/8/docs/api/java/util/StringJoiner.html