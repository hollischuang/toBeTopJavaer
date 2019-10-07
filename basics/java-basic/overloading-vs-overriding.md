![overloading-vs-overriding][1]

重载（Overloading）和重写（Overriding）是Java中两个比较重要的概念。但是对于新手来说也比较容易混淆。本文通过两个简单的例子说明了他们之间的区别。

## 定义

### 重载

简单说，就是函数或者方法有同样的名称，但是参数列表不相同的情形，这样的同名不同参数的函数或者方法之间，互相称之为重载函数或者方法。

### 重写

重写指的是在Java的子类与父类中有两个名称、参数列表都相同的方法的情况。由于他们具有相同的方法签名，所以子类中的新方法将覆盖父类中原有的方法。

## 重载 VS 重写

关于重载和重写，你应该知道以下几点：

> 1、重载是一个编译期概念、重写是一个运行期间概念。
> 
> 2、重载遵循所谓“编译期绑定”，即在编译时根据参数变量的类型判断应该调用哪个方法。
> 
> 3、重写遵循所谓“运行期绑定”，即在运行的时候，根据引用变量所指向的实际对象的类型来调用方法
> 
> 4、因为在编译期已经确定调用哪个方法，所以重载并不是多态。而重写是多态。重载只是一种语言特性，是一种语法规则，与多态无关，与面向对象也无关。（注：严格来说，重载是编译时多态，即静态多态。但是，Java中提到的多态，在不特别说明的情况下都指动态多态）

## 重写的例子

下面是一个重写的例子，看完代码之后不妨猜测一下输出结果：

    class Dog{
        public void bark(){
            System.out.println("woof ");
        }
    }
    class Hound extends Dog{
        public void sniff(){
            System.out.println("sniff ");
        }
    
        public void bark(){
            System.out.println("bowl");
        }
    }
    
    public class OverridingTest{
        public static void main(String [] args){
            Dog dog = new Hound();
            dog.bark();
        }
    }
    

输出结果：

    bowl
    

上面的例子中，`dog`对象被定义为`Dog`类型。在编译期，编译器会检查Dog类中是否有可访问的`bark()`方法，只要其中包含`bark（）`方法，那么就可以编译通过。在运行期，`Hound`对象被`new`出来，并赋值给`dog`变量，这时，JVM是明确的知道`dog`变量指向的其实是`Hound`对象的引用。所以，当`dog`调用`bark()`方法的时候，就会调用`Hound`类中定义的`bark（）`方法。这就是所谓的动态多态性。

### 重写的条件

> 参数列表必须完全与被重写方法的相同；
> 
> 返回类型必须完全与被重写方法的返回类型相同；
> 
> 访问级别的限制性一定不能比被重写方法的强；
> 
> 访问级别的限制性可以比被重写方法的弱；
> 
> 重写方法一定不能抛出新的检查异常或比被重写的方法声明的检查异常更广泛的检查异常
> 
> 重写的方法能够抛出更少或更有限的异常（也就是说，被重写的方法声明了异常，但重写的方法可以什么也不声明）
> 
> 不能重写被标示为final的方法；
> 
> 如果不能继承一个方法，则不能重写这个方法。

## 重载的例子

    class Dog{
        public void bark(){
            System.out.println("woof ");
        }
    
        //overloading method
        public void bark(int num){
            for(int i=0; i<num; i++)
                System.out.println("woof ");
        }
    }
    

上面的代码中，定义了两个bark方法，一个是没有参数的bark方法，另外一个是包含一个int类型参数的bark方法。在编译期，编译期可以根据方法签名（方法名和参数情况）情况确定哪个方法被调用。

### 重载的条件

> 被重载的方法必须改变参数列表；
> 
> 被重载的方法可以改变返回类型；
> 
> 被重载的方法可以改变访问修饰符；
> 
> 被重载的方法可以声明新的或更广的检查异常；
> 
> 方法能够在同一个类中或者在一个子类中被重载。

## 参考资料

[Overriding vs. Overloading in Java][2]

 [1]: http://www.hollischuang.com/wp-content/uploads/2016/03/overloading-vs-overriding.png
 [2]: http://www.programcreek.com/2009/02/overriding-and-overloading-in-java-with-examples/