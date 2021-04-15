很多人看到父加载器、子加载器这样的名字，就会认为Java中的类加载器之间存在着继承关系。

甚至网上很多文章也会有类似的错误观点。

这里需要明确一下，双亲委派模型中，类加载器之间的父子关系一般不会以继承（Inheritance）的关系来实现，而是都使用组合（Composition）关系来复用父加载器的代码的。

如下为ClassLoader中父加载器的定义：

    public abstract class ClassLoader {
        // The parent class loader for delegation
        private final ClassLoader parent;
    }