我们在《[深入分析Java的编译原理][1]》中提到过，为了让Java语言具有良好的跨平台能力，Java独具匠心的提供了一种可以在所有平台上都能使用的一种中间代码——字节码（ByteCode）。

有了字节码，无论是哪种平台（如Windows、Linux等），只要安装了虚拟机，都可以直接运行字节码。

同样，有了字节码，也解除了Java虚拟机和Java语言之间的耦合。这话可能很多人不理解，Java虚拟机不就是运行Java语言的么？这种解耦指的是什么？

其实，目前Java虚拟机已经可以支持很多除Java语言以外的语言了，如Kotlin、Groovy、JRuby、Jython、Scala等。之所以可以支持，就是因为这些语言也可以被编译成字节码。而虚拟机并不关心字节码是有哪种语言编译而来的。

经常使用IDE的开发者可能会发现，当我们在Intelij IDEA中，鼠标右键想要创建Java类的时候，IDE还会提示创建其他类型的文件，这就是IDE默认支持的一些可以运行在JVM上面的语言，没有提示的，可以通过插件来支持。

<img src="https://www.hollischuang.com/wp-content/uploads/2018/11/languages.png" />

目前，可以直接在JVM上运行的语言有很多，今天介绍其中比较重要的九种。每种语言通过一段『HelloWorld』代码进行演示，看看不同语言的语法有何不同。

### Kotlin

Kotlin是一种在Java虚拟机上运行的静态类型编程语言，它也可以被编译成为JavaScript源代码。Kotlin的设计初衷就是用来生产高性能要求的程序的，所以运行起来和Java也是不相上下。Kotlin可以从 JetBrains InteilliJ Idea IDE这个开发工具以插件形式使用。

#### Hello World In Kotlin

```kotlin
fun main(args: Array<String>) {
    println("Hello, world!")
}
```

### Groovy

Apache的Groovy是Java平台上设计的面向对象编程语言。它的语法风格与Java很像，Java程序员能够很快的熟练使用 Groovy，实际上，Groovy编译器是可以接受完全纯粹的Java语法格式的。

使用Groovy的一个重要特点就是使用类型推断，即能够让编译器能够在程序员没有明确说明的时候推断出变量的类型。Groovy可以使用其他Java语言编写的库。Groovy的语法与Java非常相似，大多数Java代码也匹配Groovy的语法规则，尽管可能语义不同。

#### Hello World In Groovy

```groovy
static void main(String[] args) {
    println('Hello, world!');
}
```

### Scala

Scala是一门多范式的编程语言，设计初衷是要集成面向对象编程和函数式编程的各种特性。

Scala经常被我们描述为多模式的编程语言，因为它混合了来自很多编程语言的元素的特征。但无论如何它本质上还是一个纯粹的面向对象语言。它相比传统编 程语言最大的优势就是提供了很好并行编程基础框架措施了。Scala代码能很好的被优化成字节码，运行起来和原生Java一样快。

#### Hello World In Scala

```scala
object HelloWorld {
    def main(args: Array[String]) {
       System.out.println("Hello, world!");
    }
 }
```

### Jruby

JRuby是用来桥接Java与Ruby的，它是使用比Groovy更加简短的语法来编写代码，能够让每行代码执行更多的任务。就和Ruby一样，JRuby不仅仅只提供了高级的语法格式。它同样提供了纯粹的面向对象的实现，闭包等等，而且JRuby跟Ruby自身相比多了很多基于Java类库 可以调用，虽然Ruby也有很多类库，但是在数量以及广泛性上是无法跟Java标准类库相比的。

#### Hello World In Jruby

```ruby
puts 'Hello, world!'
```

### Jython

Jython，是一个用Java语言写的Python解释器。Jython能够用Python语言来高效生成动态编译的Java字节码。

#### Hello World In Jython

```py
print "Hello, world!"
```

### Fantom

Fantom是一种通用的面向对象编程语言，由Brian和Andy Frank创建，运行在Java Runtime Environment，JavaScript和.NET Common Language Runtime上。其主要设计目标是提供标准库API，以抽象出代码是否最终将在JRE或CLR上运行的问题。

Fantom是与Groovy以及JRuby差不多的一样面向对 象的编程语言，但是悲剧的是Fantom无法使用Java类库，而是使用它自己扩展的类库。

#### Hello World In Fantom

```fantom
class Hello {
    static Void main() { echo("Hello, world!") }
}
```

### Clojure

Clojure是Lisp编程语言在Java平台上的现代、函数式及动态方言。 与其他Lisp一样，Clojure视代码为数据且拥有一套Lisp宏系统。

虽然Clojure也能被直接编译成Java字节码，但是无法使用动态语言特性以及直 接调用Java类库。与其他的JVM脚本语言不一样，Clojure并不算是面向对象的。

#### Hello World In Clojure

```clojure
(defn -main [& args]
    (println "Hello, World!"))
```

### Rhino

Rhino是一个完全以Java编写的JavaScript引擎，目前由Mozilla基金会所管理。

Rhino的特点是为JavaScript加了个壳，然后嵌入到Java中，这样能够让Java程序员直接使用。其中Rhino的JavaAdapters能够让JavaScript通过调用Java的类来实现特定的功能。

#### Hello World In Rhino

```js
print('Hello, world!')
```

### Ceylon

Ceylon是一种面向对象，强烈静态类型的编程语言，强调不变性，由Red Hat创建。 Ceylon程序在Java虚拟机上运行，​​可以编译为JavaScript。 语言设计侧重于源代码可读性，可预测性，可扩展性，模块性和元编程性。

#### Hello World In Ceylon

```ceylon
shared void run() {
    print("Hello, world!");
}
```

### 总结

好啦，以上就是目前主流的可以在JVM上面运行的9种语言。加上Java正好10种。如果你是一个Java开发，那么有必要掌握以上9中的一种，这样可以在一些有特殊需求的场景中有更多的选择。推荐在Groovy、Scala、Kotlin中选一个。

[1]: https://www.hollischuang.com/archives/2322
