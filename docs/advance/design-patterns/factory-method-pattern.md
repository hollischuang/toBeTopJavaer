## 概念

工厂方法模式(Factory Method Pattern)又称为工厂模式，也叫虚拟构造器(Virtual Constructor)模式或者多态工厂(Polymorphic Factory)模式，它属于类创建型模式。

工厂方法模式是一种实现了“工厂”概念的面向对象设计模式。就像其他创建型模式一样，它也是处理在不指定对象具体类型的情况下创建对象的问题。

> 工厂方法模式的实质是“定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行。”

## 用途

工厂方法模式和[简单工厂模式][2]虽然都是通过工厂来创建对象，他们之间最大的不同是——**工厂方法模式在设计上完全完全符合“[开闭原则][3]”。**

在以下情况下可以使用工厂方法模式：

> 一个类不知道它所需要的对象的类：在工厂方法模式中，客户端不需要知道具体产品类的类名，只需要知道所对应的工厂即可，具体的产品对象由具体工厂类创建；客户端需要知道创建具体产品的工厂类。
> 
> 一个类通过其子类来指定创建哪个对象：在工厂方法模式中，对于抽象工厂类只需要提供一个创建产品的接口，而由其子类来确定具体要创建的对象，利用面向对象的多态性和[里氏代换原则][3]，在程序运行时，子类对象将覆盖父类对象，从而使得系统更容易扩展。
> 
> 将创建对象的任务委托给多个工厂子类中的某一个，客户端在使用时可以无须关心是哪一个工厂子类创建产品子类，需要时再动态指定，可将具体工厂类的类名存储在配置文件或数据库中。

## 实现方式

工厂方法模式包含如下角色：

> Product：抽象产品（`Operation`）
> 
> ConcreteProduct：具体产品(`OperationAdd`)
> 
> Factory：抽象工厂(`IFactory`)
> 
> ConcreteFactory：具体工厂(`AddFactory`)

[<img src="http://www.hollischuang.com/wp-content/uploads/2016/04/QQ20160412-0.png" alt="QQ20160412-0" width="798" height="518" class="alignnone size-full wp-image-1402" />][4]

这里还用计算器的例子。在保持`Operation`，`OperationAdd`，`OperationDiv`，`OperationSub`，`OperationMul`等几个方法不变的情况下，修改简单工厂模式中的工厂类（`OperationFactory`）。替代原有的那个"万能"的大工厂类，这里使用工厂方法来代替：

    //工厂接口
    public interface IFactory {
        Operation CreateOption();
    }
    
    //加法类工厂
    public class AddFactory implements IFactory {
    
        public Operation CreateOption() {
            return new OperationAdd();
        }
    }
    
    //除法类工厂
    public class DivFactory implements IFactory {
    
        public Operation CreateOption() {
            return new OperationDiv();
        }
    }
    
    //除法类工厂
    public class MulFactory implements IFactory {
    
        public Operation CreateOption() {
            return new OperationMul();
        }
    }
    
    //减法类工厂
    public class SubFactory implements IFactory {
    
        public Operation CreateOption() {
            return new OperationSub();
        }
    }
    

这样，在客户端中想要执行加法运算时，需要以下方式：

    public class Main {
    
        public static void main(String[] args) {
            IFactory factory = new AddFactory();
            Operation operationAdd =  factory.CreateOption();
            operationAdd.setValue1(10);
            operationAdd.setValue2(5);
            System.out.println(operationAdd.getResult());
        }
    }
    

到这里，一个工厂方法模式就已经写好了。

* * *

从代码量上看，这种工厂方法模式比简单工厂方法模式更加复杂。针对不同的操作（Operation）类都有对应的工厂。很多人会有以下疑问：

> 貌似工厂方法模式比简单工厂模式要复杂的多？
> 
> 工厂方法模式和我自己创建对象没什么区别？为什么要多搞出一些工厂来？

下面就针对以上两个问题来深入理解一下工厂方法模式。

## 工厂方法模式的利与弊

### 为什么要使用工厂来创建对象？

> 封装对象的创建过程

在工厂方法模式中，工厂方法用来创建客户所需要的产品，同时还向客户**隐藏了哪种具体产品类将被实例化这一细节，用户只需要关心所需产品对应的工厂，无须关心创建细节，甚至无须知道具体产品类的类名。**

基于工厂角色和产品角色的多态性设计是工厂方法模式的关键。**它能够使工厂可以自主确定创建何种产品对象，而如何创建这个对象的细节则完全封装在具体工厂内部。**工厂方法模式之所以又被称为多态工厂模式，是因为所有的具体工厂类都具有同一抽象父类。

### 为什么每种对象要单独有一个工厂？

> 符合『[开放-封闭原则][5]』

主要目的是为了解耦。在系统中加入新产品时，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，而只要添加一个具体工厂和具体产品就可以了。这样，系统的可扩展性也就变得非常好，完全符合“[开闭原则][3]”。

以上就是工厂方法模式的优点。但是，工厂模式也有一些不尽如人意的地方：

> 在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加，在一定程度上增加了系统的复杂度，有更多的类需要编译和运行，会给系统带来一些额外的开销。
> 
> 由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度，且在实现时可能需要用到DOM、反射等技术，增加了系统的实现难度。

## 工厂方法与简单工厂的区别

工厂模式克服了简单工厂模式违背[开放-封闭原则][3]的缺点，又保持了封装对象创建过程的优点。

他们都是集中封装了对象的创建，使得要更换对象时，不需要做大的改动就可实现，降低了客户端与产品对象的耦合。

## 总结

工厂方法模式是简单工厂模式的进一步抽象和推广。

由于使用了面向对象的多态性，工厂方法模式保持了简单工厂模式的优点，而且克服了它的缺点。

在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做。这个核心类仅仅负责给出具体工厂必须实现的接口，而不负责产品类被实例化这种细节，这使得工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品。

工厂方法模式的主要优点是增加新的产品类时无须修改现有系统，并封装了产品对象的创建细节，系统具有良好的灵活性和可扩展性；其缺点在于增加新产品的同时需要增加新的工厂，导致系统类的个数成对增加，在一定程度上增加了系统的复杂性。

文中所有代码见[GitHub][6]

## 参考资料

[大话设计模式][7]

[深入浅出设计模式][8]

[工厂方法模式(Factory Method Pattern)][9]

 [1]: http://www.hollischuang.com/archives/category/%E7%BB%BC%E5%90%88%E5%BA%94%E7%94%A8/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F
 [2]: http://www.hollischuang.com/archives/1391
 [3]: http://www.hollischuang.com/archives/220
 [4]: http://www.hollischuang.com/wp-content/uploads/2016/04/QQ20160412-0.png
 [5]: http://www.hollischuang.com/archives/220http://
 [6]: https://github.com/hollischuang/DesignPattern
 [7]: http://s.click.taobao.com/t?e=m=2&s=R5B/xd29JVMcQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67jN2wQzI0ZBVHBMajAjK1gBpS4hLH/P02ckKYNRBWOBBey11vvWwHXSniyi5vWXIZkKWZZq7zWpCC8X3k5aQlui0qVGgqDL2o8YMXU3NNCg/&pvid=10_42.120.73.203_224_1460382841310
 [8]: http://s.click.taobao.com/t?e=m%3D2%26s%3DObpq8Qxse2EcQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67utJaEGcptl2kfkm8XrrgBtpS4hLH%2FP02ckKYNRBWOBBey11vvWwHXTpkOAWGyim%2Bw2PNKvM2u52N5aP5%2Bgx7zgh4LxdBQDQSXEqY%2Bakgpmw&pvid=10_121.0.29.199_322_1460465025379
 [9]: http://design-patterns.readthedocs.org/zh_CN/latest/creational_patterns/factory_method.html#id11