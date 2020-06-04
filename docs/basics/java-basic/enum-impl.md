Java SE5提供了一种新的类型-Java的枚举类型，关键字enum可以将一组具名的值的有限集合创建为一种新的类型，而这些具名的值可以作为常规的程序组件使用，这是一种非常有用的功能。

要想看源码，首先得有一个类吧，那么枚举类型到底是什么类呢？是enum吗？答案很明显不是，enum就和class一样，只是一个关键字，他并不是一个类，那么枚举是由什么类维护的呢，我们简单的写一个枚举：

    public enum t {
        SPRING,SUMMER;
    }
然后我们使用反编译，看看这段代码到底是怎么实现的，反编译后代码内容如下：

    public final class T extends Enum
    {
        private T(String s, int i)
        {
            super(s, i);
        }
        public static T[] values()
        {
            T at[];
            int i;
            T at1[];
            System.arraycopy(at = ENUM$VALUES, 0, at1 = new T[i = at.length], 0, i);
            return at1;
        }
    
        public static T valueOf(String s)
        {
            return (T)Enum.valueOf(demo/T, s);
        }
    
        public static final T SPRING;
        public static final T SUMMER;
        private static final T ENUM$VALUES[];
        static
        {
            SPRING = new T("SPRING", 0);
            SUMMER = new T("SUMMER", 1);
            ENUM$VALUES = (new T[] {
                SPRING, SUMMER
            });
        }
    }
    
通过反编译后代码我们可以看到，public final class T extends Enum，说明，该类是继承了Enum类的，同时final关键字告诉我们，这个类也是不能被继承的。

当我们使用enmu来定义一个枚举类型的时候，编译器会自动帮我们创建一个final类型的类继承Enum类，所以枚举类型不能被继承。