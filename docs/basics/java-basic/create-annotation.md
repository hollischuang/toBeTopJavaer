在Java中，类使用class定义，接口使用interface定义，注解和接口的定义差不多，增加了一个@符号，即@interface，代码如下：

    public @interface EnableAuth {
    
    }

注解中可以定义成员变量，用于信息的描述，跟接口中方法的定义类似，代码如下：
    
    public @interface EnableAuth {
        String name();
    }

还可以添加默认值：

    public @interface EnableAuth {
        String name() default "猿天地";
    }

上面的介绍只是完成了自定义注解的第一步，开发中日常使用注解大部分是用在类上，方法上，字段上，示列代码如下：

    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface EnableAuth {
    
    }

Target 

用于指定被修饰的注解修饰哪些程序单元，也就是上面说的类，方法，字段

Retention 

用于指定被修饰的注解被保留多长时间，分别SOURCE（注解仅存在于源码中，在class字节码文件中不包含）,CLASS（默认的保留策略，注解会在class字节码文件中存在，但运行时无法获取）,RUNTIME（注解会在class字节码文件中存在，在运行时可以通过反射获取到）三种类型，如果想要在程序运行过程中通过反射来获取注解的信息需要将Retention设置为RUNTIME

Documented 

用于指定被修饰的注解类将被javadoc工具提取成文档

Inherited 

用于指定被修饰的注解类将具有继承性