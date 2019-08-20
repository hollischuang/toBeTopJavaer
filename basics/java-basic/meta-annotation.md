说简单点，就是  定义其他注解的注解 。
比如Override这个注解，就不是一个元注解。而是通过元注解定义出来的。

    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.SOURCE)
    public @interface Override {
    }

这里面的
@Target
@Retention
就是元注解。

元注解有四个:@Target（表示该注解可以用于什么地方）、@Retention（表示再什么级别保存该注解信息）、@Documented（将此注解包含再javadoc中）、@Inherited（允许子类继承父类中的注解）。