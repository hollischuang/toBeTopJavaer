如前文我们提到的，因为类加载器之间有严格的层次关系，那么也就使得Java类也随之具备了层次关系。

或者说这种层次关系是优先级。

比如一个定义在java.lang包下的类，因为它被存放在rt.jar之中，所以在被加载过程汇总，会被一直委托到Bootstrap ClassLoader，最终由Bootstrap ClassLoader所加载。

而一个用户自定义的com.hollis.ClassHollis类，他也会被一直委托到Bootstrap ClassLoader，但是因为Bootstrap ClassLoader不负责加载该类，那么会在由Extention ClassLoader尝试加载，而Extention ClassLoader也不负责这个类的加载，最终才会被Application ClassLoader加载。

这种机制有几个好处。

首先，通过委派的方式，可以避免类的重复加载，当父加载器已经加载过某一个类时，子加载器就不会再重新加载这个类。

另外，通过双亲委派的方式，还保证了安全性。因为Bootstrap ClassLoader在加载的时候，只会加载JAVA_HOME中的jar包里面的类，如java.lang.Integer，那么这个类是不会被随意替换的，除非有人跑到你的机器上， 破坏你的JDK。

那么，就可以避免有人自定义一个有破坏功能的java.lang.Integer被加载。这样可以有效的防止核心Java API被篡改。