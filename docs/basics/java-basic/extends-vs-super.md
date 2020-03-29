`<? extends T>`和`<? super T>`是Java泛型中的“通配符（Wildcards）”和“边界（Bounds）”的概念。

`<? extends T>`：是指 “上界通配符（Upper Bounds Wildcards）”，即泛型中的类必须为当前类的子类或当前类。

`<? super T>`：是指 “下界通配符（Lower Bounds Wildcards）”，即泛型中的类必须为当前类或者其父类。


先看一个列子：

    public class Food {}
    public class Fruit extends Food {}
    public class Apple extends Fruit {}
    public class Banana extends Fruit{}
    
    public class GenericTest {
    
        public void testExtends(List<? extends Fruit> list){
    
            //报错,extends为上界通配符,只能取值,不能放.
            //因为Fruit的子类不只有Apple还有Banana,这里不能确定具体的泛型到底是Apple还是Banana，所以放入任何一种类型都会报错
            //list.add(new Apple());
    
            //可以正常获取
            Fruit fruit = list.get(1);
        }
    
        public void testSuper(List<? super Fruit> list){
    
            //super为下界通配符，可以存放元素，但是也只能存放当前类或者子类的实例，以当前的例子来讲，
            //无法确定Fruit的父类是否只有Food一个(Object是超级父类)
            //因此放入Food的实例编译不通过
            list.add(new Apple());
    //        list.add(new Food());
    
            Object object = list.get(1);
        }
    }
    
在testExtends方法中，因为泛型中用的是extends，在向list中存放元素的时候，我们并不能确定List中的元素的具体类型，即可能是Apple也可能是Banana。因此调用add方法时，不论传入new Apple()还是new Banana()，都会出现编译错误。


理解了extends之后，再看super就很容易理解了，即我们不能确定testSuper方法的参数中的泛型是Fruit的哪个父类，因此在调用get方法时只能返回Object类型。结合extends可见，在获取泛型元素时，使用extends获取到的是泛型中的上边界的类型(本例子中为Fruit),范围更小。

在使用泛型时，存取元素时用super,获取元素时，用extends。

频繁往外读取内容的，适合用上界Extends。经常往里插入的，适合用下界Super。

本文来源：https://juejin.im/post/5c653fe06fb9a049e3089d88