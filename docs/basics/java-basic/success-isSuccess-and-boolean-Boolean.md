在日常开发中，我们会经常要在类中定义布尔类型的变量，比如在给外部系统提供一个RPC接口的时候，我们一般会定义一个字段表示本次请求是否成功的。

关于这个"本次请求是否成功"的字段的定义，其实是有很多种讲究和坑的，稍有不慎就会掉入坑里，作者在很久之前就遇到过类似的问题，本文就来围绕这个简单分析一下。到底该如何定一个布尔类型的成员变量。

一般情况下，我们可以有以下四种方式来定义一个布尔类型的成员变量：

    boolean success
    boolean isSuccess
    Boolean success
    Boolean isSuccess
    

以上四种定义形式，你日常开发中最常用的是哪种呢？到底哪一种才是正确的使用姿势呢？

通过观察我们可以发现，前两种和后两种的主要区别是变量的类型不同，前者使用的是boolean，后者使用的是Boolean。

另外，第一种和第三种在定义变量的时候，变量命名是success，而另外两种使用isSuccess来命名的。

首先，我们来分析一下，到底应该是用success来命名，还是使用isSuccess更好一点。

### success 还是 isSuccess

到底应该是用success还是isSuccess来给变量命名呢？从语义上面来讲，两种命名方式都可以讲的通，并且也都没有歧义。那么还有什么原则可以参考来让我们做选择呢。

在阿里巴巴Java开发手册中关于这一点，有过一个『强制性』规定：

![-w656][1]￼

那么，为什么会有这样的规定呢？我们看一下POJO中布尔类型变量不同的命名有什么区别吧。

    class Model1  {
        private Boolean isSuccess;
        public void setSuccess(Boolean success) {
            isSuccess = success;
        }
        public Boolean getSuccess() {
            return isSuccess;
        }
     }
    
    class Model2 {
        private Boolean success;
        public Boolean getSuccess() {
            return success;
        }
        public void setSuccess(Boolean success) {
            this.success = success;
        }
    }
    
    class Model3 {
        private boolean isSuccess;
        public boolean isSuccess() {
            return isSuccess;
        }
        public void setSuccess(boolean success) {
            isSuccess = success;
        }
    }
    
    class Model4 {
        private boolean success;
        public boolean isSuccess() {
            return success;
        }
        public void setSuccess(boolean success) {
            this.success = success;
        }
    }
    

以上代码的setter/getter是使用Intellij IDEA自动生成的，仔细观察以上代码，你会发现以下规律：

*   基本类型自动生成的getter和setter方法，名称都是`isXXX()`和`setXXX()`形式的。
*   包装类型自动生成的getter和setter方法，名称都是`getXXX()`和`setXXX()`形式的。

既然，我们已经达成一致共识使用基本类型boolean来定义成员变量了，那么我们再来具体看下Model3和Model4中的setter/getter有何区别。

我们可以发现，虽然Model3和Model4中的成员变量的名称不同，一个是success，另外一个是isSuccess，但是他们自动生成的getter和setter方法名称都是`isSuccess`和`setSuccess`。

**Java Bean中关于setter/getter的规范**

关于Java Bean中的getter/setter方法的定义其实是有明确的规定的，根据[JavaBeans(TM) Specification][2]规定，如果是普通的参数propertyName，要以以下方式定义其setter/getter：

    public <PropertyType> get<PropertyName>();
    public void set<PropertyName>(<PropertyType> a);
    

但是，布尔类型的变量propertyName则是单独定义的：

    public boolean is<PropertyName>();
    public void set<PropertyName>(boolean m);
    

![-w687][3]￼

通过对照这份JavaBeans规范，我们发现，在Model4中，变量名为isSuccess，如果严格按照规范定义的话，他的getter方法应该叫isIsSuccess。但是很多IDE都会默认生成为isSuccess。

那这样做会带来什么问题呢。

在一般情况下，其实是没有影响的。但是有一种特殊情况就会有问题，那就是发生序列化的时候。

**序列化带来的影响**

关于序列化和反序列化请参考[Java对象的序列化与反序列化][4]。我们这里拿比较常用的JSON序列化来举例，看看看常用的fastJson、jackson和Gson之间有何区别：

    public class BooleanMainTest {
    
        public static void main(String[] args) throws IOException {
            //定一个Model3类型
            Model3 model3 = new Model3();
            model3.setSuccess(true);
    
            //使用fastjson(1.2.16)序列化model3成字符串并输出
            System.out.println("Serializable Result With fastjson :" + JSON.toJSONString(model3));
    
            //使用Gson(2.8.5)序列化model3成字符串并输出
            Gson gson =new Gson();
            System.out.println("Serializable Result With Gson :" +gson.toJson(model3));
    
            //使用jackson(2.9.7)序列化model3成字符串并输出
            ObjectMapper om = new ObjectMapper();
            System.out.println("Serializable Result With jackson :" +om.writeValueAsString(model3));
        }
    
    }
    
    class Model3 implements Serializable {
    
        private static final long serialVersionUID = 1836697963736227954L;
        private boolean isSuccess;
        public boolean isSuccess() {
            return isSuccess;
        }
        public void setSuccess(boolean success) {
            isSuccess = success;
        }
        public String getHollis(){
            return "hollischuang";
        }
    }
    

以上代码的Model3中，只有一个成员变量即isSuccess，三个方法，分别是IDE帮我们自动生成的isSuccess和setSuccess，另外一个是作者自己增加的一个符合getter命名规范的方法。

以上代码输出结果：

    Serializable Result With fastjson :{"hollis":"hollischuang","success":true}
    Serializable Result With Gson :{"isSuccess":true}
    Serializable Result With jackson :{"success":true,"hollis":"hollischuang"}
    

在fastjson和jackson的结果中，原来类中的isSuccess字段被序列化成success，并且其中还包含hollis值。而Gson中只有isSuccess字段。

我们可以得出结论：fastjson和jackson在把对象序列化成json字符串的时候，是通过反射遍历出该类中的所有getter方法，得到getHollis和isSuccess，然后根据JavaBeans规则，他会认为这是两个属性hollis和success的值。直接序列化成json:{"hollis":"hollischuang","success":true}

但是Gson并不是这么做的，他是通过反射遍历该类中的所有属性，并把其值序列化成json:{"isSuccess":true}

可以看到，由于不同的序列化工具，在进行序列化的时候使用到的策略是不一样的，所以，对于同一个类的同一个对象的序列化结果可能是不同的。

前面提到的关于对getHollis的序列化只是为了说明fastjson、jackson和Gson之间的序列化策略的不同，我们暂且把他放到一边，我们把他从Model3中删除后，重新执行下以上代码，得到结果：

    Serializable Result With fastjson :{"success":true}
    Serializable Result With Gson :{"isSuccess":true}
    Serializable Result With jackson :{"success":true}
    

现在，不同的序列化框架得到的json内容并不相同，如果对于同一个对象，我使用fastjson进行序列化，再使用Gson反序列化会发生什么？

    public class BooleanMainTest {
        public static void main(String[] args) throws IOException {
            Model3 model3 = new Model3();
            model3.setSuccess(true);
            Gson gson =new Gson();
            System.out.println(gson.fromJson(JSON.toJSONString(model3),Model3.class));
        }
    }
    
    
    class Model3 implements Serializable {
        private static final long serialVersionUID = 1836697963736227954L;
        private boolean isSuccess;
        public boolean isSuccess() {
            return isSuccess;
        }
        public void setSuccess(boolean success) {
            isSuccess = success;
        }
        @Override
        public String toString() {
            return new StringJoiner(", ", Model3.class.getSimpleName() + "[", "]")
                .add("isSuccess=" + isSuccess)
                .toString();
        }
    }
    

以上代码，输出结果：

    Model3[isSuccess=false]
    

这和我们预期的结果完全相反，原因是因为JSON框架通过扫描所有的getter后发现有一个isSuccess方法，然后根据JavaBeans的规范，解析出变量名为success，把model对象序列化城字符串后内容为`{"success":true}`。

根据`{"success":true}`这个json串，Gson框架在通过解析后，通过反射寻找Model类中的success属性，但是Model类中只有isSuccess属性，所以，最终反序列化后的Model类的对象中，isSuccess则会使用默认值false。

但是，一旦以上代码发生在生产环境，这绝对是一个致命的问题。

所以，作为开发者，我们应该想办法尽量避免这种问题的发生，对于POJO的设计者来说，只需要做简单的一件事就可以解决这个问题了，那就是把isSuccess改为success。这样，该类里面的成员变量时success，getter方法是isSuccess，这是完全符合JavaBeans规范的。无论哪种序列化框架，执行结果都一样。就从源头避免了这个问题。

引用以下R大关于阿里巴巴Java开发手册这条规定的评价（https://www.zhihu.com/question/55642203 ）：

![-w665][5]￼

所以，**在定义POJO中的布尔类型的变量时，不要使用isSuccess这种形式，而要直接使用success！**

### Boolean还是boolean

前面我们介绍完了在success和isSuccess之间如何选择，那么排除错误答案后，备选项还剩下：

    boolean success
    Boolean success
    

那么，到底应该是用Boolean还是boolean来给定一个布尔类型的变量呢？

我们知道，boolean是基本数据类型，而Boolean是包装类型。关于基本数据类型和包装类之间的关系和区别请参考[一文读懂什么是Java中的自动拆装箱][6]

那么，在定义一个成员变量的时候到底是使用包装类型更好还是使用基本数据类型呢？

我们来看一段简单的代码

     /**
     * @author Hollis
     */
    public class BooleanMainTest {
        public static void main(String[] args) {
            Model model1 = new Model();
            System.out.println("default model : " + model1);
        }
    }
    
    class Model {
        /**
         * 定一个Boolean类型的success成员变量
         */
        private Boolean success;
        /**
         * 定一个boolean类型的failure成员变量
         */
        private boolean failure;
    
        /**
         * 覆盖toString方法，使用Java 8 的StringJoiner
         */
        @Override
        public String toString() {
            return new StringJoiner(", ", Model.class.getSimpleName() + "[", "]")
                .add("success=" + success)
                .add("failure=" + failure)
                .toString();
        }
    }
    

以上代码输出结果为：

    default model : Model[success=null, failure=false]
    

可以看到，当我们没有设置Model对象的字段的值的时候，Boolean类型的变量会设置默认值为`null`，而boolean类型的变量会设置默认值为`false`。

即对象的默认值是`null`，boolean基本数据类型的默认值是`false`。

在阿里巴巴Java开发手册中，对于POJO中如何选择变量的类型也有着一些规定：

<img src="http://www.hollischuang.com/wp-content/uploads/2018/12/640.jpeg" alt="" width="1080" height="445" class="aligncenter size-full wp-image-3558" />

这里建议我们使用包装类型，原因是什么呢？

举一个扣费的例子，我们做一个扣费系统，扣费时需要从外部的定价系统中读取一个费率的值，我们预期该接口的返回值中会包含一个浮点型的费率字段。当我们取到这个值得时候就使用公式：金额*费率=费用 进行计算，计算结果进行划扣。

如果由于计费系统异常，他可能会返回个默认值，如果这个字段是Double类型的话，该默认值为null，如果该字段是double类型的话，该默认值为0.0。

如果扣费系统对于该费率返回值没做特殊处理的话，拿到null值进行计算会直接报错，阻断程序。拿到0.0可能就直接进行计算，得出接口为0后进行扣费了。这种异常情况就无法被感知。

这种使用包装类型定义变量的方式，通过异常来阻断程序，进而可以被识别到这种线上问题。如果使用基本数据类型的话，系统可能不会报错，进而认为无异常。

**以上，就是建议在POJO和RPC的返回值中使用包装类型的原因。**

但是关于这一点，作者之前也有过不同的看法：对于布尔类型的变量，我认为可以和其他类型区分开来，作者并不认为使用null进而导致NPE是一种最好的实践。因为布尔类型只有true/false两种值，我们完全可以和外部调用方约定好当返回值为false时的明确语义。

后来，作者单独和《阿里巴巴Java开发手册》、《码出高效》的作者——孤尽 单独1V1(qing) Battle(jiao)了一下。最终达成共识，还是**尽量使用包装类型**。

**但是，作者还是想强调一个我的观点，尽量避免在你的代码中出现不确定的null值。**


### 总结

本文围绕布尔类型的变量定义的类型和命名展开了介绍，最终我们可以得出结论，在定义一个布尔类型的变量，尤其是一个给外部提供的接口返回值时，要使用success来命名，阿里巴巴Java开发手册建议使用封装类来定义POJO和RPC返回值中的变量。但是这不意味着可以随意的使用null，我们还是要尽量避免出现对null的处理的。

 [1]: http://www.hollischuang.com/wp-content/uploads/2018/12/15449439364854.jpg
 [2]: https://download.oracle.com/otndocs/jcp/7224-javabeans-1.01-fr-spec-oth-JSpec/
 [3]: http://www.hollischuang.com/wp-content/uploads/2018/12/15449455942045.jpg
 [4]: http://www.hollischuang.com/archives/1150
 [5]: http://www.hollischuang.com/wp-content/uploads/2018/12/15449492627754.jpg
 [6]: http://www.hollischuang.com/archives/2700
 [7]: http://www.hollischuang.com/archives/883
 [8]: http://www.hollischuang.com/archives/74
 [9]: http://www.hollischuang.com/wp-content/uploads/2018/12/15449430847727.jpg
