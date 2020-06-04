Java中的类通过实现 `java.io.Serializable` 接口以启⽤其序列化功能。 未实现此接⼜的类将⽆法使其任何状态序列化或反序列化。 

可序列化类的所有⼦类型本⾝都是可序列化的。 

序列化接⼜没有⽅法或字段， 仅⽤于标识可序列化的语义。

当试图对⼀个对象进⾏序列化的时候， 如果遇到不⽀持`Serializable` 接口的对象。 在此情况下， 将抛`NotSerializableException`。

如果要序列化的类有⽗类， 要想同时将在⽗类中定义过的变量持久化下来， 那么⽗类也应该集成`java.io.Serializable`接口。

`Externalizable`继承了`Serializable`， 该接⼜中定义了两个抽象⽅法：`writeExternal()`与`readExternal()`。 当使⽤`Externalizable`接口来进⾏序列化与反序列化的时候需要开发⼈员重写writeExternal()与readExternal()⽅法。 

如果没有在这两个⽅法中定义序列化实现细节， 那么序列化之后， 对象内容为空。 

实现`Externalizable`接⼜的类必须要提供⼀个`public`的⽆参的构造器。

所以， 实现`Externalizable`， 并实现`writeExternal()`和`readExternal()`⽅法可以指定序列化哪些属性。
