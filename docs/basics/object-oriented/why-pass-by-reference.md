对于初学者来说，要想把这个问题回答正确，是比较难的。在第二天整理答案的时候，我发现我竟然无法通过简单的语言把这个事情描述的很容易理解，遗憾的是，我也没有在网上找到哪篇文章可以把这个事情讲解的通俗易懂。所以，就有了我写这篇文章的初衷。

### 辟谣时间

关于这个问题，在[StackOverflow][2]上也引发过广泛的讨论，看来很多程序员对于这个问题的理解都不尽相同，甚至很多人理解的是错误的。还有的人可能知道Java中的参数传递是值传递，但是说不出来为什么。

在开始深入讲解之前，有必要纠正一下大家以前的那些错误看法了。如果你有以下想法，那么你有必要好好阅读本文。

> 错误理解一：值传递和引用传递，区分的条件是传递的内容，如果是个值，就是值传递。如果是个引用，就是引用传递。
>
> 错误理解二：Java是引用传递。
>
> 错误理解三：传递的参数如果是普通类型，那就是值传递，如果是对象，那就是引用传递。


### Java中的值传递

值传递和引用传的的定义如下：

> 值传递（pass by value）是指在调用函数时将实际参数`复制`一份传递到函数中，这样在函数中如果对`参数`进行修改，将不会影响到实际参数。
>
> 引用传递（pass by reference）是指在调用函数时将实际参数的地址`直接`传递到函数中，那么在函数中对`参数`所进行的修改，将影响到实际参数。


你有一把钥匙，当你的朋友想要去你家的时候，如果你`直接`把你的钥匙给他了，这就是引用传递。这种情况下，如果他对这把钥匙做了什么事情，比如他在钥匙上刻下了自己名字，那么这把钥匙还给你的时候，你自己的钥匙上也会多出他刻的名字。

你有一把钥匙，当你的朋友想要去你家的时候，你`复刻`了一把新钥匙给他，自己的还在自己手里，这就是值传递。这种情况下，他对这把钥匙做什么都不会影响你手里的这把钥匙。


我们举一个实际的代码例子：
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

我们再来看下面的例子：

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
输出结果：
```
print in pass , user is User{name='hollischuang', gender='Male'}
print in main , user is User{name='hollischuang', gender='Male'}
```

过程如下：

[<img src="http://www.hollischuang.com/wp-content/uploads/2018/04/pass21.png" alt="pass2" width="832" height="732" class="aligncenter size-full wp-image-2307" />][5]

同样的，在参数传递的过程中，实际参数的地址`0X1213456`被拷贝给了形参，只是，在这个方法中，并没有对形参本身进行修改，而是修改的形参持有的地址中存储的内容。

所以，值传递和引用传递的区别并不是传递的内容。而是实参到底有没有被复制一份给形参。在判断实参内容有没有受影响的时候，要看传的的是什么，如果你传递的是个地址，那么就看这个地址的变化会不会有影响，而不是看地址指向的对象的变化。就像钥匙和房子的关系。

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
