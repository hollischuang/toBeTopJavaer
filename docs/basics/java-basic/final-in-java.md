final是Java中的一个关键字，它所表示的是“这部分是无法修改的”。

使用 final 可以定义 ：变量、方法、类。

### final变量

如果将变量设置为final，则不能更改final变量的值(它将是常量)。


    class Test{
         final String name = "Hollis";
     
    }

一旦final变量被定义之后，是无法进行修改的。

### final方法

如果任何方法声明为final，则不能覆盖它。

    class Parent {
        final void name() {
            System.out.println("Hollis");
        }
    }
    
当我们定义以上类的子类的时候，无法覆盖其name方法，会编译失败。


### final类

如果把任何一个类声明为final，则不能继承它。


    final class Parent {
        
    }
    
    
以上类不能被继承！