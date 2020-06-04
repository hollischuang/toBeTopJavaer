“异常链”是Java中⾮常流⾏的异常处理概念， 是指在进⾏⼀个异常处理时抛出了另外⼀个异常， 由此产⽣了⼀个异常链条。 

该技术⼤多⽤于将“ 受检查异常” （ checked exception） 封装成为“⾮受检查异常”（ unchecked exception)或者RuntimeException。 

顺便说⼀下， 如果因为因为异常你决定抛出⼀个新的异常， 你⼀定要包含原有的异常， 这样， 处理程序才可以通过getCause()和initCause()⽅法来访问异常最终的根源。

从 Java 1.4版本开始，几乎所有的异常都支持异常链。

以下是Throwable中支持异常链的方法和构造函数。

    Throwable getCause()
    Throwable initCause(Throwable)
    Throwable(String, Throwable)
    Throwable(Throwable)
    
initCause和Throwable构造函数的Throwable参数是导致当前异常的异常。 getCause返回导致当前异常的异常，initCause设置当前异常的原因。

以下示例显示如何使用异常链。

    try {
    
    } catch (IOException e) {
        throw new SampleException("Other IOException", e);
    }
    
在此示例中，当捕获到IOException时，将创建一个新的SampleException异常，并附加原始的异常原因，并将异常链抛出到下一个更高级别的异常处理程序。