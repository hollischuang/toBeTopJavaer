
## 概念

学习过设计模式的人大概都知道[Head First设计模式][2]这本书，这本书中介绍的第一个模式就是策略模式。把策略模式放在第一个，笔者认为主要有两个原因：1、这的确是一个比较简单的模式。2、这个模式可以充分的体现面向对象设计原则中的`封装变化`、`多用组合，少用继承`、`针对接口编程，不针对实现编程`等原则。

> 策略模式(Strategy Pattern)：定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法独立于使用它的客户而变化，也称为政策模式(Policy)。

## 用途

结合策略模式的概念，我们找一个实际的场景来理解一下。

假设我们是一家新开的书店，为了招揽顾客，我们推出会员服务，我们把店里的会员分为三种，分别是初级会员、中级会员和高级会员。针对不同级别的会员我们给予不同的优惠。初级会员买书我们不打折、中级会员买书我们打九折、高级会员买书我们打八折。

[<img src="http://www.hollischuang.com/wp-content/uploads/2016/09/bookstore-300x300.jpg" alt="bookstore" width="300" height="300" class="alignright size-medium wp-image-1694" />][3] 我们希望用户在付款的时候，只要刷一下书的条形码，会员再刷一下他的会员卡，收银台的工组人员就能直接知道应该向顾客收取多少钱。

在不使用模式的情况下，我们可以在结算的方法中使用`if/else`语句来区别出不同的会员来计算价格。

但是，如果我们有一天想要把初级会员的折扣改成9.8折怎么办？有一天我要推出超级会员怎么办？有一天我要针对中级会员可打折的书的数量做限制怎么办？

使用`if\else`设计出来的系统，所有的算法都写在了一起，只要有改动我就要修改整个类。我们都知道，只要是修改代码就有可能引入问题。为了避免这个问题，我们可以使用策略模式。。。

> 对于收银台系统，计算应收款的时候，一个客户只可能是初级、中级、高级会员中的一种。不同的会员使用不同的算法来计算价格。收银台系统其实不关心具体的会员类型和折扣之间的关系。也不希望会员和折扣之间的任何改动会影响到收银台系统。

在介绍策略模式的具体实现方式之前，再来巩固一下几个面向对象设计原则：`封装变化`、`多用组合，少用继承`、`针对接口编程，不针对实现编程`。想一想如何运用到策略模式中，并且有什么好处。

## 实现方式

策略模式包含如下角色：

> Context: 环境类
> 
> Strategy: 抽象策略类
> 
> ConcreteStrategy: 具体策略类

[<img src="http://www.hollischuang.com/wp-content/uploads/2016/09/Strategy.jpg" alt="strategy" width="764" height="329" class="aligncenter size-full wp-image-1692" />][4]

我们运用策略模式来实现一下书店的收银台系统。我们可以把会员抽象成一个策略类，不同的会员类型是具体的策略类。不同策略类里面实现了计算价格这一算法。然后通过组合的方式把会员集成到收银台中。

先定义一个接口，这个接口就是抽象策略类，该接口定义了计算价格方法，具体实现方式由具体的策略类来定义。

    /**
     * Created by hollis on 16/9/19. 会员接口
     */
    public interface Member {
    
        /**
         * 计算应付价格
         * @param bookPrice 书籍原价(针对金额,建议使用BigDecimal,double会损失精度)
         * @return 应付金额
         */
        public double calPrice(double bookPrice);
    }
    

针对不同的会员，定义三种具体的策略类，每个类中都分别实现计算价格方法。

    /**
     * Created by hollis on 16/9/19. 初级会员
     */
    public class PrimaryMember implements Member {
    
        @Override
        public double calPrice(double bookPrice) {
            System.out.println("对于初级会员的没有折扣");
            return bookPrice;
        }
    }
    
    
    /**
     * Created by hollis on 16/9/19. 中级会员,买书打九折
     */
    public class IntermediateMember implements Member {
    
        @Override
        public double calPrice(double bookPrice) {
            System.out.println("对于中级会员的折扣为10%");
            return bookPrice * 0.9;
        }
    }
    
    
    /**
     * Created by hollis on 16/9/19. 高级会员,买书打八折
     */
    public class AdvancedMember implements Member {
    
        @Override
        public double calPrice(double bookPrice) {
            System.out.println("对于中级会员的折扣为20%");
            return bookPrice * 0.8;
        }
    }
    

上面几个类的定义体现了`封装变化`的设计原则，不同会员的具体折扣方式改变不会影响到其他的会员。

定义好了抽象策略类和具体策略类之后，我们再来定义环境类，所谓环境类，就是集成算法的类。这个例子中就是收银台系统。采用组合的方式把会员集成进来。

    /**
     * Created by hollis on 16/9/19. 书籍价格类
     */
    public class Cashier {
    
        /**
         * 会员,策略对象
         */
        private Member member;
    
        public Cashier(Member member){
            this.member = member;
        }
    
        /**
         * 计算应付价格
         * @param booksPrice
         * @return
         */
        public double quote(double booksPrice) {
            return this.member.calPrice(booksPrice);
        }
    }
    

这个Cashier类就是一个环境类，该类的定义体现了`多用组合，少用继承`、`针对接口编程，不针对实现编程`两个设计原则。由于这里采用了组合+接口的方式，后面我们在推出超级会员的时候无须修改Cashier类。只要再定义一个`SuperMember implements Member` 就可以了。

下面定义一个客户端来测试一下：

    /**
     * Created by hollis on 16/9/19.
     */
    public class BookStore {
    
        public static void main(String[] args) {
    
            //选择并创建需要使用的策略对象
            Member strategy = new AdvancedMember();
            //创建环境
            Cashier cashier = new Cashier(strategy);
            //计算价格
            double quote = cashier.quote(300);
            System.out.println("高级会员图书的最终价格为：" + quote);
    
            strategy = new IntermediateMember();
            cashier = new Cashier(strategy);
            quote = cashier.quote(300);
            System.out.println("中级会员图书的最终价格为：" + quote);
        }
    }
    
    //对于中级会员的折扣为20%
    //高级会员图书的最终价格为：240.0
    //对于中级会员的折扣为10%
    //中级会员图书的最终价格为：270.0
    

从上面的示例可以看出，策略模式仅仅封装算法，提供新的算法插入到已有系统中，策略模式并不决定在何时使用何种算法。在什么情况下使用什么算法是由客户端决定的。

*   策略模式的重心
    
    *   策略模式的重心不是如何实现算法，而是如何组织、调用这些算法，从而让程序结构更灵活，具有更好的维护性和扩展性。

*   算法的平等性
    
    *   策略模式一个很大的特点就是各个策略算法的平等性。对于一系列具体的策略算法，大家的地位是完全一样的，正因为这个平等性，才能实现算法之间可以相互替换。所有的策略算法在实现上也是相互独立的，相互之间是没有依赖的。
    
    *   所以可以这样描述这一系列策略算法：策略算法是相同行为的不同实现。

*   运行时策略的唯一性
    
    *   运行期间，策略模式在每一个时刻只能使用一个具体的策略实现对象，虽然可以动态地在不同的策略实现中切换，但是同时只能使用一个。

*   公有的行为
    
    *   经常见到的是，所有的具体策略类都有一些公有的行为。这时候，就应当把这些公有的行为放到共同的抽象策略角色Strategy类里面。当然这时候抽象策略角色必须要用Java抽象类实现，而不能使用接口。（[《JAVA与模式》之策略模式][5]）

## 策略模式的优缺点

### 优点

*   策略模式提供了对“开闭原则”的完美支持，用户可以在不修改原有系统的基础上选择算法或行为，也可以灵活地增加新的算法或行为。
*   策略模式提供了管理相关的算法族的办法。策略类的等级结构定义了一个算法或行为族。恰当使用继承可以把公共的代码移到父类里面，从而避免代码重复。
*   使用策略模式可以避免使用多重条件(if-else)语句。多重条件语句不易维护，它把采取哪一种算法或采取哪一种行为的逻辑与算法或行为的逻辑混合在一起，统统列在一个多重条件语句里面，比使用继承的办法还要原始和落后。

### 缺点

*   客户端必须知道所有的策略类，并自行决定使用哪一个策略类。这就意味着客户端必须理解这些算法的区别，以便适时选择恰当的算法类。
*   由于策略模式把每个具体的策略实现都单独封装成为类，如果备选的策略很多的话，那么对象的数目就会很可观。可以通过使用享元模式在一定程度上减少对象的数量。

文中所有代码见[GitHub][6]

## 参考资料

[《JAVA与模式》之策略模式][5]

 [1]: http://www.hollischuang.com/archives/category/%E7%BB%BC%E5%90%88%E5%BA%94%E7%94%A8/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F
 [2]: http://s.click.taobao.com/DxA2xSx
 [3]: http://www.hollischuang.com/wp-content/uploads/2016/09/bookstore.jpg
 [4]: http://www.hollischuang.com/wp-content/uploads/2016/09/Strategy.jpg
 [5]: http://www.cnblogs.com/java-my-life/archive/2012/05/10/2491891.html
 [6]: https://github.com/hollischuang/DesignPattern