

### 一、各种语言中的编译器是如何处理泛型的

通常情况下，一个编译器处理泛型有两种方式：

1\.`Code specialization`。在实例化一个泛型类或泛型方法时都产生一份新的目标代码（字节码or二进制代码）。例如，针对一个泛型List`，可能需要 针对`String`，`Integer`，`Float`产生三份目标代码。

2\.`Code sharing`。对每个泛型类只生成唯一的一份目标代码；该泛型类的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。

**C++**中的模板（`template`）是典型的`Code specialization`实现。**C++**编译器会为每一个泛型类实例生成一份执行代码。执行代码中`Integer List`和`String List`是两种不同的类型。这样会导致**代码膨胀（code bloat）**。 **C#**里面泛型无论在程序源码中、编译后的`IL`中（Intermediate Language，中间语言，这时候泛型是一个占位符）或是运行期的CLR中都是切实存在的，`List<Integer>`与`List<String>`就是两个不同的类型，它们在系统运行期生成，有自己的虚方法表和类型数据，这种实现称为类型膨胀，基于这种方法实现的泛型被称为`真实泛型`。 **Java**语言中的泛型则不一样，它只在程序源码中存在，在编译后的字节码文件中，就已经被替换为原来的原生类型（Raw Type，也称为裸类型）了，并且在相应的地方插入了强制转型代码，因此对于运行期的Java语言来说，`ArrayList<Integer>`与`ArrayList<String>`就是同一个类。所以说泛型技术实际上是Java语言的一颗语法糖，Java语言中的泛型实现方法称为**类型擦除**，基于这种方法实现的泛型被称为`伪泛型`。

`C++`和`C#`是使用`Code specialization`的处理机制，前面提到，他有一个缺点，那就是**会导致代码膨胀**。另外一个弊端是在引用类型系统中，浪费空间，因为引用类型集合中元素本质上都是一个指针。没必要为每个类型都产生一份执行代码。而这也是Java编译器中采用`Code sharing`方式处理泛型的主要原因。

`Java`编译器通过`Code sharing`方式为每个泛型类型创建唯一的字节码表示，并且将该泛型类型的实例都映射到这个唯一的字节码表示上。将多种泛型类形实例映射到唯一的字节码表示是通过**类型擦除**（`type erasue`）实现的。

* * *

### 二、什么是类型擦除

前面我们多次提到这个词：**类型擦除**（`type erasue`）**，那么到底什么是类型擦除呢？

> 类型擦除指的是通过类型参数合并，将泛型类型实例关联到同一份字节码上。编译器只为泛型类型生成一份字节码，并将其实例关联到这份字节码上。类型擦除的关键在于从泛型类型中清除类型参数的相关信息，并且再必要的时候添加类型检查和类型转换的方法。 类型擦除可以简单的理解为将泛型java代码转换为普通java代码，只不过编译器更直接点，将泛型java代码直接转换成普通java字节码。 类型擦除的主要过程如下： 1.将所有的泛型参数用其最左边界（最顶级的父类型）类型替换。（这部分内容可以看：[Java泛型中extends和super的理解][2]） 2.移除所有的类型参数。

* * *

### 三、Java编译器处理泛型的过程

**code 1:**

    public static void main(String[] args) {  
        Map<String, String> map = new HashMap<String, String>();  
        map.put("name", "hollis");  
        map.put("age", "22");  
        System.out.println(map.get("name"));  
        System.out.println(map.get("age"));  
    }  


**反编译后的code 1:**

    public static void main(String[] args) {  
        Map map = new HashMap();  
        map.put("name", "hollis");  
        map.put("age", "22"); 
        System.out.println((String) map.get("name"));  
        System.out.println((String) map.get("age"));  
    }  


我们发现泛型都不见了，程序又变回了Java泛型出现之前的写法，泛型类型都变回了原生类型，

* * *

**code 2:**

    interface Comparable<A> {
        public int compareTo(A that);
    }
    
    public final class NumericValue implements Comparable<NumericValue> {
        private byte value;
    
        public NumericValue(byte value) {
            this.value = value;
        }
    
        public byte getValue() {
            return value;
        }
    
        public int compareTo(NumericValue that) {
            return this.value - that.value;
        }
    }


**反编译后的code 2:**

     interface Comparable {
      public int compareTo( Object that);
    } 
    
    public final class NumericValue
        implements Comparable
    {
        public NumericValue(byte value)
        {
            this.value = value;
        }
        public byte getValue()
        {
            return value;
        }
        public int compareTo(NumericValue that)
        {
            return value - that.value;
        }
        public volatile int compareTo(Object obj)
        {
            return compareTo((NumericValue)obj);
        }
        private byte value;
    }

* * *

**code 3:**

    public class Collections {
        public static <A extends Comparable<A>> A max(Collection<A> xs) {
            Iterator<A> xi = xs.iterator();
            A w = xi.next();
            while (xi.hasNext()) {
                A x = xi.next();
                if (w.compareTo(x) < 0)
                    w = x;
            }
            return w;
        }
    }


**反编译后的code 3:**

    public class Collections
    {
        public Collections()
        {
        }
        public static Comparable max(Collection xs)
        {
            Iterator xi = xs.iterator();
            Comparable w = (Comparable)xi.next();
            while(xi.hasNext())
            {
                Comparable x = (Comparable)xi.next();
                if(w.compareTo(x) < 0)
                    w = x;
            }
            return w;
        }
    }


第2个泛型类`Comparable <A>`擦除后 A被替换为最左边界`Object`。`Comparable<NumericValue>`的类型参数`NumericValue`被擦除掉，但是这直 接导致`NumericValue`没有实现接口`Comparable的compareTo(Object that)`方法，于是编译器充当好人，添加了一个**桥接方法**。 第3个示例中限定了类型参数的边界`<A extends Comparable<A>>A`，A必须为`Comparable<A>`的子类，按照类型擦除的过程，先讲所有的类型参数 ti换为最左边界`Comparable<A>`，然后去掉参数类型`A`，得到最终的擦除后结果。

* * *

### 四、泛型带来的问题

**一、当泛型遇到重载：**

    public class GenericTypes {  
    
        public static void method(List<String> list) {  
            System.out.println("invoke method(List<String> list)");  
        }  
    
        public static void method(List<Integer> list) {  
            System.out.println("invoke method(List<Integer> list)");  
        }  
    }  


上面这段代码，有两个重载的函数，因为他们的参数类型不同，一个是`List<String>`另一个是`List<Integer>` ，但是，这段代码是编译通不过的。因为我们前面讲过，参数`List<Integer>`和`List<String>`编译之后都被擦除了，变成了一样的原生类型List<e>，擦除动作导致这两个方法的特征签名变得一模一样。</e>

**二、当泛型遇到catch:**

如果我们自定义了一个泛型异常类GenericException<t>，那么，不要尝试用多个catch取匹配不同的异常类型，例如你想要分别捕获GenericException<String>、GenericException<Integer>，这也是有问题的。</Integer></String></t>

三、当泛型内包含静态变量

    public class StaticTest{
        public static void main(String[] args){
            GT<Integer> gti = new GT<Integer>();
            gti.var=1;
            GT<String> gts = new GT<String>();
            gts.var=2;
            System.out.println(gti.var);
        }
    }
    class GT<T>{
        public static int var=0;
        public void nothing(T x){}
    }


答案是——2！由于经过类型擦除，所有的泛型类实例都关联到同一份字节码上，泛型类的所有静态变量是共享的。

* * *

### 五、总结

1\.虚拟机中没有泛型，只有普通类和普通方法,所有泛型类的类型参数在编译时都会被擦除,泛型类并没有自己独有的Class类对象。比如并不存在`List<String>`.class或是`List<Integer>.class`，而只有`List.class`。 2.创建泛型对象时请指明类型，让编译器尽早的做参数检查（**Effective Java，第23条：请不要在新代码中使用原生态类型**） 3.不要忽略编译器的警告信息，那意味着潜在的`ClassCastException`等着你。 4.静态变量是被泛型类的所有实例所共享的。对于声明为`MyClass<T>`的类，访问其中的静态变量的方法仍然是 `MyClass.myStaticVar`。不管是通过`new MyClass<String>`还是`new MyClass<Integer>`创建的对象，都是共享一个静态变量。 5.泛型的类型参数不能用在`Java`异常处理的`catch`语句中。因为异常处理是由JVM在运行时刻来进行的。由于类型信息被擦除，`JVM`是无法区分两个异常类型`MyException<String>`和`MyException<Integer>`的。对于`JVM`来说，它们都是 `MyException`类型的。也就无法执行与异常对应的`catch`语句。

[1]: http://docs.oracle.com/javase/tutorial/java/generics/erasure.html
[2]: /archives/255