Java中定义枚举是使用enum关键字的，但是Java中其实还有一个java.lang.Enum类。这是一个抽象类，定义如下：


    package java.lang;

    public abstract class Enum<E extends Enum<E>> implements Constable, Comparable<E>, Serializable {
        private final String name;
        private final int ordinal;

    }
    
这个类我们在日常开发中不会用到，但是其实我们使用enum定义的枚举，其实现方式就是通过继承Enum类实现的。

当我们使用enmu来定义一个枚举类型的时候，编译器会自动帮我们创建一个final类型的类继承Enum类，所以枚举类型不能被继承。