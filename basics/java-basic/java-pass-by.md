# 为什么说Java中只有值传递

对于初学者来说，要想把这个问题回答正确，是比较难的。在第二天整理答案的时候，我发现我竟然无法通过简单的语言把这个事情描述的很容易理解，遗憾的是，我也没有在网上找到哪篇文章可以把这个事情讲解的通俗易懂。所以，就有了我写这篇文章的初衷。这篇文章中，我从什么是方法的实际参数和形式参数开始，给你讲解为什么说Java中只有值传递。

### 辟谣时间

关于这个问题，在[StackOverflow][2]上也引发过广泛的讨论，看来很多程序员对于这个问题的理解都不尽相同，甚至很多人理解的是错误的。还有的人可能知道Java中的参数传递是值传递，但是说不出来为什么。

在开始深入讲解之前，有必要纠正一下大家以前的那些错误看法了。如果你有以下想法，那么你有必要好好阅读本文。

> 错误理解一：值传递和引用传递，区分的条件是传递的内容，如果是个值，就是值传递。如果是个引用，就是引用传递。
>
> 错误理解二：Java是引用传递。
>
> 错误理解三：传递的参数如果是普通类型，那就是值传递，如果是对象，那就是引用传递。

### 实参与形参

我们都知道，在Java中定义方法的时候是可以定义参数的。比如Java中的main方法，`public static void main(String[] args)`，这里面的args就是参数。参数在程序语言中分为形式参数和实际参数。

> 形式参数：是在定义函数名和函数体的时候使用的参数,目的是用来接收调用该函数时传入的参数。
>
> 实际参数：在调用有参函数时，主调函数和被调函数之间有数据传递关系。在主调函数中调用一个函数时，函数名后面括号中的参数称为“实际参数”。

简单举个例子：

```
public static void main(String[] args) {
  ParamTest pt = new ParamTest();
  pt.sout("Hollis");//实际参数为 Hollis
}

public void sout(String name) { //形式参数为 name
  System.out.println(name);
}
```

实际参数是调用有参方法的时候真正传递的内容，而形式参数是用于接收实参内容的参数。

### 值传递与引用传递

上面提到了，当我们调用一个有参函数的时候，会把实际参数传递给形式参数。但是，在程序语言中，这个传递过程中传递的两种情况，即值传递和引用传递。我们来看下程序语言中是如何定义和区分值传递和引用传递的。

> 值传递（pass by value）是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。
>
> 引用传递（pass by reference）是指在调用函数时将实际参数的地址直接传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

有了上面的概念，然后大家就可以写代码实践了，来看看Java中到底是值传递还是引用传递 ，于是，最简单的一段代码出来了：
```
public static void main(String[] args) {
  ParamTest pt = new ParamTest();

  int i = 10;
  pt.pass(i);
  System.out.println("print in main , i is " + i);
}

public void pass(int j) {
  j = 20;
  System.out.println("print in pass , j is " + j);
}
```

上面的代码中，我们在pass方法中修改了参数j的值，然后分别在pass方法和main方法中打印参数的值。输出结果如下：

```
print in pass , j is 20
print in main , i is 10
```

可见，pass方法内部对name的值的修改并没有改变实际参数i的值。那么，按照上面的定义，有人得到结论：Java的方法传递是值传递。

但是，很快就有人提出质疑了（哈哈，所以，不要轻易下结论咯。）。然后，他们会搬出以下代码：
```
public static void main(String[] args) {
  ParamTest pt = new ParamTest();

  User hollis = new User();
  hollis.setName("Hollis");
  hollis.setGender("Male");
  pt.pass(hollis);
  System.out.println("print in main , user is " + hollis);
}

public void pass(User user) {
  user.setName("hollischuang");
  System.out.println("print in pass , user is " + user);
}
```

同样是一个pass方法，同样是在pass方法内修改参数的值。输出结果如下：
```
print in pass , user is User{name='hollischuang', gender='Male'}
print in main , user is User{name='hollischuang', gender='Male'}
```

经过pass方法执行后，实参的值竟然被改变了，那按照上面的引用传递的定义，实际参数的值被改变了，这不就是引用传递了么。于是，根据上面的两段代码，有人得出一个新的结论：Java的方法中，在传递普通类型的时候是值传递，在传递对象类型的时候是引用传递。

但是，这种表述仍然是错误的。不信你看下面这个参数类型为对象的参数传递：
```
public static void main(String[] args) {
  ParamTest pt = new ParamTest();

  String name = "Hollis";
  pt.pass(name);
  System.out.println("print in main , name is " + name);
}

public void pass(String name) {
  name = "hollischuang";
  System.out.println("print in pass , name is " + name);
}
```

上面的代码输出结果为
```
print in pass , name is hollischuang
print in main , name is Hollis
```

这又作何解释呢？同样传递了一个对象，但是原始参数的值并没有被修改，难道传递对象又变成值传递了？

### Java中的值传递

上面，我们举了三个例子，表现的结果却不一样，这也是导致很多初学者，甚至很多高级程序员对于Java的传递类型有困惑的原因。

其实，我想告诉大家的是，上面的概念没有错，只是代码的例子有问题。来，我再来给大家画一下概念中的重点，然后再举几个真正恰当的例子。

> 值传递（pass by value）是指在调用函数时将实际参数`复制`一份传递到函数中，这样在函数中如果对`参数`进行修改，将不会影响到实际参数。
>
> 引用传递（pass by reference）是指在调用函数时将实际参数的地址`直接`传递到函数中，那么在函数中对`参数`所进行的修改，将影响到实际参数。

那么，我来给大家总结一下，值传递和引用传递之前的区别的重点是什么。

[<img src="http://www.hollischuang.com/wp-content/uploads/2018/04/pass.jpg" alt="pass" width="474" height="73" class="aligncenter size-full wp-image-2289" />][3]

我们上面看过的几个pass的例子中，都只关注了实际参数内容是否有改变。如传递的是User对象，我们试着改变他的name属性的值，然后检查是否有改变。其实，在实验方法上就错了，当然得到的结论也就有问题了。

为什么说实验方法错了呢？这里我们来举一个形象的例子。再来深入理解一下值传递和引用传递，然后你就知道为啥错了。

你有一把钥匙，当你的朋友想要去你家的时候，如果你`直接`把你的钥匙给他了，这就是引用传递。这种情况下，如果他对这把钥匙做了什么事情，比如他在钥匙上刻下了自己名字，那么这把钥匙还给你的时候，你自己的钥匙上也会多出他刻的名字。

你有一把钥匙，当你的朋友想要去你家的时候，你`复刻`了一把新钥匙给他，自己的还在自己手里，这就是值传递。这种情况下，他对这把钥匙做什么都不会影响你手里的这把钥匙。

但是，不管上面那种情况，你的朋友拿着你给他的钥匙，进到你的家里，把你家的电视砸了。那你说你会不会受到影响？而我们在pass方法中，改变user对象的name属性的值的时候，不就是在“砸电视”么。

还拿上面的一个例子来举例，我们`真正的改变参数`，看看会发生什么？
```
public static void main(String[] args) {
  ParamTest pt = new ParamTest();

  User hollis = new User();
  hollis.setName("Hollis");
  hollis.setGender("Male");
  pt.pass(hollis);
  System.out.println("print in main , user is " + hollis);
}

public void pass(User user) {
  user = new User();
  user.setName("hollischuang");
  user.setGender("Male");
  System.out.println("print in pass , user is " + user);
}
```

上面的代码中，我们在pass方法中，改变了user对象，输出结果如下：
```
print in pass , user is User{name='hollischuang', gender='Male'}
print in main , user is User{name='Hollis', gender='Male'}
```

我们来画一张图，看一下整个过程中发生了什么，然后我再告诉你，为啥Java中只有值传递。

[<img src="http://www.hollischuang.com/wp-content/uploads/2018/04/pass1.png" alt="pass1" width="859" height="721" class="aligncenter size-full wp-image-2293" />][4]

稍微解释下这张图，当我们在main中创建一个User对象的时候，在堆中开辟一块内存，其中保存了name和gender等数据。然后hollis持有该内存的地址`0x123456`（图1）。当尝试调用pass方法，并且hollis作为实际参数传递给形式参数user的时候，会把这个地址`0x123456`交给user，这时，user也指向了这个地址（图2）。然后在pass方法内对参数进行修改的时候，即`user = new User();`，会重新开辟一块`0X456789`的内存，赋值给user。后面对user的任何修改都不会改变内存`0X123456`的内容（图3）。

上面这种传递是什么传递？肯定不是引用传递，如果是引用传递的话，在`user=new User()`的时候，实际参数的引用也应该改为指向`0X456789`，但是实际上并没有。

通过概念我们也能知道，这里是把实际参数的引用的地址复制了一份，传递给了形式参数。所以，**上面的参数其实是值传递，把实参对象引用的地址当做值传递给了形式参数。**

我们再来回顾下之前的那个“砸电视”的例子，看那个例子中的传递过程发生了什么。

[<img src="http://www.hollischuang.com/wp-content/uploads/2018/04/pass21.png" alt="pass2" width="832" height="732" class="aligncenter size-full wp-image-2307" />][5]

同样的，在参数传递的过程中，实际参数的地址`0X1213456`被拷贝给了形参，只是，在这个方法中，并没有对形参本身进行修改，而是修改的形参持有的地址中存储的内容。

所以，值传递和引用传递的区别并不是传递的内容。而是实参到底有没有被复制一份给形参。在判断实参内容有没有受影响的时候，要看传的的是什么，如果你传递的是个地址，那么就看这个地址的变化会不会有影响，而不是看地址指向的对象的变化。就像钥匙和房子的关系。

那么，既然这样，为啥上面同样是传递对象，传递的String对象和User对象的表现结果不一样呢？我们在pass方法中使用`name = "hollischuang";`试着去更改name的值，阴差阳错的直接改变了name的引用的地址。因为这段代码，会new一个String，在把引用交给name，即等价于`name = new String("hollischuang");`。而原来的那个"Hollis"字符串还是由实参持有着的，所以，并没有修改到实际参数的值。

[<img src="http://www.hollischuang.com/wp-content/uploads/2018/04/pass3.png" alt="pass3" width="515" height="399" class="aligncenter size-full wp-image-2311" />][6]

**所以说，Java中其实还是值传递的，只不过对于对象参数，值的内容是对象的引用。**

### 总结

无论是值传递还是引用传递，其实都是一种求值策略([Evaluation strategy][7])。在求值策略中，还有一种叫做按共享传递(call by sharing)。其实Java中的参数传递严格意义上说应该是按共享传递。

> 按共享传递，是指在调用函数时，传递给函数的是实参的地址的拷贝（如果实参在栈中，则直接拷贝该值）。在函数内部对参数进行操作时，需要先拷贝的地址寻找到具体的值，再进行操作。如果该值在栈中，那么因为是直接拷贝的值，所以函数内部对参数进行操作不会对外部变量产生影响。如果原来拷贝的是原值在堆中的地址，那么需要先根据该地址找到堆中对应的位置，再进行操作。因为传递的是地址的拷贝所以函数内对值的操作对外部变量是可见的。

简单点说，Java中的传递，是值传递，而这个值，实际上是对象的引用。

而按共享传递其实只是按值传递的一个特例罢了。所以我们可以说Java的传递是按共享传递，或者说Java中的传递是值传递。

### 参考资料

[Evaluation strategy][7] 

[关于值传递和引用传递][8] 

[按值传递、按引用传递、按共享传递][9] 

[Is Java “pass-by-reference” or “pass-by-value”?][2]

[1]: http://www.hollischuang.com/wp-content/uploads/2018/04/question.png
[2]: https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value
[3]: http://www.hollischuang.com/wp-content/uploads/2018/04/pass.jpg
[4]: http://www.hollischuang.com/wp-content/uploads/2018/04/pass1.png
[5]: http://www.hollischuang.com/wp-content/uploads/2018/04/pass21.png
[6]: http://www.hollischuang.com/wp-content/uploads/2018/04/pass3.png
[7]: https://en.wikipedia.org/wiki/Evaluation_strategy
[8]: http://chenwenbo.github.io/2016/05/11/%E5%85%B3%E4%BA%8E%E5%80%BC%E4%BC%A0%E9%80%92%E5%92%8C%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92/
[9]: http://menzhongxin.com/2017/02/07/%E6%8C%89%E5%80%BC%E4%BC%A0%E9%80%92-%E6%8C%89%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92%E5%92%8C%E6%8C%89%E5%85%B1%E4%BA%AB%E4%BC%A0%E9%80%92/
