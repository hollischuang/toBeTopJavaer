
* * *

## 定义一个字符串

    String s = "abcd";
    

![String-Immutability-1][1]

`s`中保存了string对象的引用。下面的箭头可以理解为“存储他的引用”。

## 使用变量来赋值变量

    String s2 = s;
    

![String-Immutability-2][2]

s2保存了相同的引用值，因为他们代表同一个对象。

## 字符串连接

    s = s.concat("ef");
    

![string-immutability][3]

`s`中保存的是一个重新创建出来的string对象的引用。

## 总结

一旦一个string对象在内存(堆)中被创建出来，他就无法被修改。特别要注意的是，String类的所有方法都没有改变字符串本身的值，都是返回了一个新的对象。

如果你需要一个可修改的字符串，应该使用StringBuffer 或者 StringBuilder。否则会有大量时间浪费在垃圾回收上，因为每次试图修改都有新的string对象被创建出来。

 [1]: http://www.programcreek.com/wp-content/uploads/2009/02/String-Immutability-1.jpeg
 [2]: http://www.programcreek.com/wp-content/uploads/2009/02/String-Immutability-2.jpeg
 [3]: http://www.programcreek.com/wp-content/uploads/2009/02/string-immutability-650x279.jpeg