Java泛型（ generics） 是JDK 5中引⼊的⼀个新特性， 允许在定义类和接⼜的时候使⽤类型参数（ type parameter） 。 

声明的类型参数在使⽤时⽤具体的类型来替换。 泛型最主要的应⽤是在JDK 5中的新集合类框架中。

泛型最⼤的好处是可以提⾼代码的复⽤性。 以List接⼜为例，我们可以将String、 Integer等类型放⼊List中， 如不⽤泛型， 存放String类型要写⼀个List接口， 存放Integer要写另外⼀个List接口， 泛型可以很好的解决这个问题。