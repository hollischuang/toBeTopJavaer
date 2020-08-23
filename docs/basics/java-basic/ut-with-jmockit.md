JMockit是基于JavaSE5中的java.lang.instrument包开发，内部使用ASM库来动态修改java的字节码，使得java这种静态语言可以想动态脚本语言一样动态设置被Mock对象私有属性，模拟静态、私有方法行为等等，对于手机开发，嵌入式开发等要求代码尽量简洁的情况下，或者对于被测试代码不想做任何修改的前提下，使用JMockit可以轻松搞定很多测试场景。

[<img src="http://www.hollischuang.com/wp-content/uploads/2015/09/20140104100723093.jpg" alt="20140104100723093" width="885" height="1010" class="alignleft size-full wp-image-571" />][1]

通过如下方式在maven中添加JMockit的相关依赖：

            <dependency>  
                <groupId>com.googlecode.jmockit</groupId>  
                <artifactId>jmockit</artifactId>  
                <version>1.5</version>  
                <scope>test</scope>  
            </dependency>  
            <dependency>  
                <groupId>com.googlecode.jmockit</groupId>  
                <artifactId>jmockit-coverage</artifactId>  
                <version>0.999.24</version>  
                <scope>test</scope>  
            </dependency>
    

JMockit有两种Mock方式：基于行为的Mock方式和基于状态的Mock方式：

引用单元测试中mock的使用及mock神器jmockit实践中JMockit API和工具如下：

[<img src="http://www.hollischuang.com/wp-content/uploads/2015/09/20140104102342843.jpg" alt="20140104102342843" width="913" height="472" class="alignleft size-full wp-image-572" />][2]

### (1).基于行为的Mock方式：

非常类似与EasyMock和PowerMock的工作原理，基本步骤为：

1\.录制方法预期行为。

2\.真实调用。

3\.验证录制的行为被调用。

通过一个简单的例子来介绍JMockit的基本流程：

**要Mock测试的方法如下：**

    public class MyObject {
        public String hello(String name){
            return "Hello " + name;
        }
    }
    

**使用JMockit编写的单元测试如下：**

    @Mocked  //用@Mocked标注的对象，不需要赋值，jmockit自动mock  
    MyObject obj;  
    
    @Test  
    public void testHello() {  
        new NonStrictExpectations() {//录制预期模拟行为  
            {  
                obj.hello("Zhangsan");  
                returns("Hello Zhangsan");  
                //也可以使用：result = "Hello Zhangsan";  
            }  
        };  
        assertEquals("Hello Zhangsan", obj.hello("Zhangsan"));//调用测试方法  
        new Verifications() {//验证预期Mock行为被调用  
            {  
                obj.hello("Hello Zhangsan");  
                times = 1;  
            }  
        };  
    }  
    

JMockit也可以分类为非局部模拟与局部模拟，区分在于Expectations块是否有参数，有参数的是局部模拟，反之是非局部模拟。

而Expectations块一般由Expectations类和NonStrictExpectations类定义，类似于EasyMock和PowerMock中的Strict Mock和一般性Mock。

用Expectations类定义的，则mock对象在运行时只能按照 Expectations块中定义的顺序依次调用方法，不能多调用也不能少调用，所以可以省略掉Verifications块；

而用NonStrictExpectations类定义的，则没有这些限制，所以如果需要验证，则要添加Verifications块。

上述的例子使用了非局部模拟，下面我们使用局部模拟来改写上面的测试，代码如下：

    @Test  
    public void testHello() {  
        final MyObject obj = new MyObject();  
        new NonStrictExpectations(obj) {//录制预期模拟行为  
            {  
                obj.hello("Zhangsan");  
                returns("Hello Zhangsan");  
                //也可以使用：result = "Hello Zhangsan";  
            }  
        };  
        assertEquals("Hello Zhangsan", obj.hello("Zhangsan"));//调用测试方法  
        new Verifications() {//验证预期Mock行为被调用  
            {  
                obj.hello("Hello Zhangsan");  
                times = 1;  
            }  
        };  
    }  
    

**模拟静态方法：**

    @Test  
    public void testMockStaticMethod() {  
        new NonStrictExpectations(ClassMocked.class) {  
            {  
                ClassMocked.getDouble(1);//也可以使用参数匹配：ClassMocked.getDouble(anyDouble);  
                result = 3;  
            }  
        };  
    
        assertEquals(3, ClassMocked.getDouble(1));  
    
        new Verifications() {  
            {  
                ClassMocked.getDouble(1);  
                times = 1;  
            }  
        };  
    }  
    

**模拟私有方法：**

如果ClassMocked类中的getTripleString(int)方法指定调用一个私有的multiply3(int)的方法，我们可以使用如下方式来Mock：

    @Test  
    public void testMockPrivateMethod() throws Exception {  
        final ClassMocked obj = new ClassMocked();  
        new NonStrictExpectations(obj) {  
            {  
                this.invoke(obj, "multiply3", 1);//如果私有方法是静态的，可以使用：this.invoke(null, "multiply3")  
                result = 4;  
            }  
        };  
    
        String actual = obj.getTripleString(1);  
        assertEquals("4", actual);  
    
        new Verifications() {  
            {  
                this.invoke(obj, "multiply3", 1);  
                times = 1;  
            }  
        };  
    }  
    

**设置Mock对象私有属性的值：** 我们知道EasyMock和PowerMock的Mock对象是通过JDK/CGLIB动态代理实现的，本质上是类的继承或者接口的实现，但是在java面向对象编程中，基类对象中的私有属性是无法被子类继承的，所以如果被Mock对象的方法中使用到了其自身的私有属性，并且这些私有属性没有提供对象访问方法，则使用传统的Mock方法是无法进行测试的，JMockit提供了设置Mocked对象私有属性值的方法，代码如下： 被测试代码：

    public class ClassMocked {  
        private String name = "name_init";  
    
        public String getName() {  
            return name;  
        }  
    
        private static String className="Class3Mocked_init";  
    
        public static String getClassName(){  
            return className;  
        }  
    }  
    

**使用JMockit设置私有属性：**

    @Test  
    public void testMockPrivateProperty() throws IOException {  
        final ClassMocked obj = new ClassMocked();  
        new NonStrictExpectations(obj) {  
            {  
                this.setField(obj, "name", "name has bean change!");  
            }  
        };  
    
        assertEquals("name has bean change!", obj.getName());  
    }  
    

**使用JMockit设置静态私有属性：**

    @Test  
    public void testMockPrivateStaticProperty() throws IOException {  
        new NonStrictExpectations(Class3Mocked.class) {  
            {  
                this.setField(ClassMocked.class, "className", "className has bean change!");  
            }  
        };  
    
        assertEquals("className has bean change!", ClassMocked.getClassName());  
    }  
    

### (2).基于状态的Mock方式：

JMockit上面的基于行为Mock方式和传统的EasyMock和PowerMock流程基本类似，相当于把被模拟的方法当作黑盒来处理，而JMockit的基于状态的Mock可以直接改写被模拟方法的内部逻辑，更像是真正意义上的白盒测试，下面通过简单例子介绍JMockit基于状态的Mock。 被测试的代码如下：

    public class StateMocked {  
    
        public static int getDouble(int i){  
            return i*2;  
        }  
    
        public int getTriple(int i){  
            return i*3;  
        }  
    } 
    

**改写普通方法内容：**

    @Test  
    public void testMockNormalMethodContent() throws IOException {  
        StateMocked obj = new StateMocked();  
        new MockUp<StateMocked>() {//使用MockUp修改被测试方法内部逻辑  
            @Mock  
          public int getTriple(int i) {  
                return i * 30;  
            }  
        };  
        assertEquals(30, obj.getTriple(1));  
        assertEquals(60, obj.getTriple(2));  
        Mockit.tearDownMocks();//注意：在JMockit1.5之后已经没有Mockit这个类，使用MockUp代替，mockUp和tearDown方法在MockUp类中  
    }  
    

**修改静态方法的内容：** 基于状态的JMockit改写静态/final方法内容和测试普通方法没有什么区别，需要注意的是在MockUp中的方法除了不包含static关键字以外，其他都和被Mock的方法签名相同，并且使用@Mock标注，测试代码如下：

    @Test  
        public void testGetTriple() {  
            new MockUp<StateMocked>() {  
                @Mock    
                public int getDouble(int i){    
                    return i*20;    
                }  
            };    
            assertEquals(20, StateMocked.getDouble(1));    
            assertEquals(40, StateMocked.getDouble(2));   
        }  
    

原文链接: http://blog.csdn.net/chjttony/article/details/17838693

 [1]: http://www.hollischuang.com/wp-content/uploads/2015/09/20140104100723093.jpg
 [2]: http://www.hollischuang.com/wp-content/uploads/2015/09/20140104102342843.jpg