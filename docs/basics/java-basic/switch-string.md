Java 7中，switch的参数可以是String类型了，这对我们来说是一个很方便的改进。到目前为止switch支持这样几种数据类型：`byte` `short` `int` `char` `String` 。但是，作为一个程序员我们不仅要知道他有多么好用，还要知道它是如何实现的，switch对整型的支持是怎么实现的呢？对字符型是怎么实现的呢？String类型呢？有一点Java开发经验的人这个时候都会猜测switch对String的支持是使用equals()方法和hashcode()方法。那么到底是不是这两个方法呢？接下来我们就看一下，switch到底是如何实现的。

<!--more-->

### 一、switch对整型支持的实现

下面是一段很简单的Java代码，定义一个int型变量a，然后使用switch语句进行判断。执行这段代码输出内容为5，那么我们将下面这段代码反编译，看看他到底是怎么实现的。

    public class switchDemoInt {
        public static void main(String[] args) {
            int a = 5;
            switch (a) {
            case 1:
                System.out.println(1);
                break;
            case 5:
                System.out.println(5);
                break;
            default:
                break;
            }
        }
    }
    //output 5


反编译后的代码如下：

    public class switchDemoInt
    {
        public switchDemoInt()
        {
        }
        public static void main(String args[])
        {
            int a = 5;
            switch(a)
            {
            case 1: // '\001'
                System.out.println(1);
                break;

            case 5: // '\005'
                System.out.println(5);
                break;
            }
        }
    }


我们发现，反编译后的代码和之前的代码比较除了多了两行注释以外没有任何区别，那么我们就知道，**switch对int的判断是直接比较整数的值**。

### 二、switch对字符型支持的实现

直接上代码：

    public class switchDemoInt {
        public static void main(String[] args) {
            char a = 'b';
            switch (a) {
            case 'a':
                System.out.println('a');
                break;
            case 'b':
                System.out.println('b');
                break;
            default:
                break;
            }
        }
    }


编译后的代码如下： 

    public class switchDemoChar
    {
        public switchDemoChar()
        {
        }
        public static void main(String args[])
        {
            char a = 'b';
            switch(a)
            {
            case 97: // 'a'
                System.out.println('a');
                break;
            case 98: // 'b'
                System.out.println('b');
                break;
            }
      }
    }


通过以上的代码作比较我们发现：对char类型进行比较的时候，实际上比较的是ascii码，编译器会把char型变量转换成对应的int型变量

### 三、switch对字符串支持的实现

还是先上代码：

    public class switchDemoString {
        public static void main(String[] args) {
            String str = "world";
            switch (str) {
            case "hello":
                System.out.println("hello");
                break;
            case "world":
                System.out.println("world");
                break;
            default:
                break;
            }
        }
    }


对代码进行反编译：

    public class switchDemoString
    {
        public switchDemoString()
        {
        }
        public static void main(String args[])
        {
            String str = "world";
            String s;
            switch((s = str).hashCode())
            {
            default:
                break;
            case 99162322:
                if(s.equals("hello"))
                    System.out.println("hello");
                break;
            case 113318802:
                if(s.equals("world"))
                    System.out.println("world");
                break;
            }
        }
    }


看到这个代码，你知道原来字符串的switch是通过`equals()`和`hashCode()`方法来实现的。**记住，switch中只能使用整型**，比如`byte`。`short`，`char`(ackii码是整型)以及`int`。还好`hashCode()`方法返回的是`int`，而不是`long`。通过这个很容易记住`hashCode`返回的是`int`这个事实。仔细看下可以发现，进行`switch`的实际是哈希值，然后通过使用equals方法比较进行安全检查，这个检查是必要的，因为哈希可能会发生碰撞。因此它的性能是不如使用枚举进行switch或者使用纯整数常量，但这也不是很差。因为Java编译器只增加了一个`equals`方法，如果你比较的是字符串字面量的话会非常快，比如”abc” ==”abc”。如果你把`hashCode()`方法的调用也考虑进来了，那么还会再多一次的调用开销，因为字符串一旦创建了，它就会把哈希值缓存起来。因此如果这个`switch`语句是用在一个循环里的，比如逐项处理某个值，或者游戏引擎循环地渲染屏幕，这里`hashCode()`方法的调用开销其实不会很大。

好，以上就是关于switch对整型、字符型、和字符串型的支持的实现方式，总结一下我们可以发现，**其实switch只支持一种数据类型，那就是整型，其他数据类型都是转换成整型之后再使用switch的。**
