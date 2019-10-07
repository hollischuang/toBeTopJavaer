1. asList 得到的只是一个 Arrays 的内部类，一个原来数组的视图 List，因此如果对它进行增删操作会报错

2. 用 ArrayList 的构造器可以将其转变成真正的 ArrayList