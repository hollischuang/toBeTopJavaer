### 枚举是如何保证线程安全的

要想看源码，首先得有一个类吧，那么枚举类型到底是什么类呢？是enum吗？答案很明显不是，enum就和class一样，只是一个关键字，他并不是一个类，那么枚举是由什么类维护的呢，我们简单的写一个枚举：

    public enum t {
        SPRING,SUMMER,AUTUMN,WINTER;
    }
    

然后我们使用反编译，看看这段代码到底是怎么实现的，反编译（<a href="/archives/58" target="_blank">Java的反编译</a>）后代码内容如下：

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
        public static final T AUTUMN;
        public static final T WINTER;
        private static final T ENUM$VALUES[];
        static
        {
            SPRING = new T("SPRING", 0);
            SUMMER = new T("SUMMER", 1);
            AUTUMN = new T("AUTUMN", 2);
            WINTER = new T("WINTER", 3);
            ENUM$VALUES = (new T[] {
                SPRING, SUMMER, AUTUMN, WINTER
            });
        }
    }
    

通过反编译后代码我们可以看到，`public final class T extends Enum`，说明，该类是继承了Enum类的，同时final关键字告诉我们，这个类也是不能被继承的。当我们使用`enmu`来定义一个枚举类型的时候，编译器会自动帮我们创建一个final类型的类继承Enum类,所以枚举类型不能被继承，我们看到这个类中有几个属性和方法。

我们可以看到：

            public static final T SPRING;
            public static final T SUMMER;
            public static final T AUTUMN;
            public static final T WINTER;
            private static final T ENUM$VALUES[];
            static
            {
                SPRING = new T("SPRING", 0);
                SUMMER = new T("SUMMER", 1);
                AUTUMN = new T("AUTUMN", 2);
                WINTER = new T("WINTER", 3);
                ENUM$VALUES = (new T[] {
                    SPRING, SUMMER, AUTUMN, WINTER
                });
            }
    

都是static类型的，因为static类型的属性会在类被加载之后被初始化，我们在<a href="/archives/199" target="_blank">深度分析Java的ClassLoader机制（源码级别）</a>和<a href="/archives/201" target="_blank">Java类的加载、链接和初始化</a>两个文章中分别介绍过，当一个Java类第一次被真正使用到的时候静态资源被初始化、Java类的加载和初始化过程都是线程安全的。所以，**创建一个enum类型是线程安全的**。
