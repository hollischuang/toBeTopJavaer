注解和反射经常结合在一起使用，在很多框架的代码中都能看到他们结合使用的影子


可以通过反射来判断类，方法，字段上是否有某个注解以及获取注解中的值, 获取某个类中方法上的注解代码示例如下：

``` 
Class<?> clz = bean.getClass();
Method[] methods = clz.getMethods();
for (Method method : methods) {
    if (method.isAnnotationPresent(EnableAuth.class)) {
        String name = method.getAnnotation(EnableAuth.class).name();
    }
}
```

通过isAnnotationPresent判断是否存在某个注解，通过getAnnotation获取注解对象，然后获取值。

### 示例

示例参考：https://blog.csdn.net/KKALL1314/article/details/96481557

自己写了一个例子，实现功能如下：

一个类的某些字段上被注解标识，在读取该属性时，将注解中的默认值赋给这些属性，没有标记的属性不赋值

``` 
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
@Inherited
public @interface MyAnno {
    String value() default "有注解";
}

```

定义一个类

``` 
@Data
@ToString
public class Person {
    @MyAnno
    private String stra;
    private String strb;
    private String strc;

    public Person(String str1,String str2,String str3){
        super();
        this.stra = str1;
        this.strb = str2;
        this.strc = str3;
    }

}

```

这里给str1加了注解，并利用反射解析并赋值：

``` 
public class MyTest {
    public static void main(String[] args) {
        //初始化全都赋值无注解
        Person person = new Person("无注解","无注解","无注解");
        //解析注解
        doAnnoTest(person);
        System.out.println(person.toString());
    }

  private static void doAnnoTest(Object obj) {
        Class clazz = obj.getClass();
        Field[] declareFields = clazz.getDeclaredFields();
        for (Field field:declareFields) {
            //检查该字段是否使用了某个注解
            if(field.isAnnotationPresent(MyAnno.class)){
                MyAnno anno = field.getAnnotation(MyAnno.class);
                if(anno!=null){
                    String fieldName = field.getName();
                    try {
                        Method setMethod = clazz.getDeclaredMethod("set" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1),String.class);
                        //获取注解的属性
                        String annoValue = anno.value();
                        //将注解的属性值赋给对应的属性
                        setMethod.invoke(obj,annoValue);
                    }catch (NoSuchMethodException e){
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    } catch (InvocationTargetException e) {
                        e.printStackTrace();
                    }

                }
            }
            
        }
    }

}

```
运行结果：

```

Person(stra=有注解, strb=无注解, strc=无注解)

``

当开发者使用了Annotation 修饰了类、方法、Field 等成员之后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT（Annotation Processing Tool)。

注解的提取需要借助于 Java 的反射技术，反射比较慢，所以注解使用时也需要谨慎计较时间成本。
