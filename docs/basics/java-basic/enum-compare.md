java 枚举值比较用 == 和 equals 方法没啥区别，两个随便用都是一样的效果。

因为枚举 Enum 类的 equals 方法默认实现就是通过 == 来比较的；

类似的 Enum 的 compareTo 方法比较的是 Enum 的 ordinal 顺序大小；

类似的还有 Enum 的 name 方法和 toString 方法一样都返回的是 Enum 的 name 值。