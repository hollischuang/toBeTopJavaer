

###一、当泛型遇到重载

    public class GenericTypes {  
    
        public static void method(List<String> list) {  
            System.out.println("invoke method(List<String> list)");  
        }  
    
        public static void method(List<Integer> list) {  
            System.out.println("invoke method(List<Integer> list)");  
        }  
    }  
    

上面这段代码，有两个重载的函数，因为他们的参数类型不同，一个是`List<String>`另一个是`List<Integer>` ，但是，这段代码是编译通不过的。因为我们前面讲过，参数`List<Integer>`和`List<String>`编译之后都被擦除了，变成了一样的原生类型List<e>，擦除动作导致这两个方法的特征签名变得一模一样。</e>

### 二、当泛型遇到catch

如果我们自定义了一个泛型异常类GenericException<t>，那么，不要尝试用多个catch取匹配不同的异常类型，例如你想要分别捕获GenericException<string>、GenericException<integer>，这也是有问题的。</integer></string></t>

### 三、当泛型内包含静态变量

    public class StaticTest{
        public static void main(String[] args){
            GT<Integer> gti = new GT<Integer>();
            gti.var=1;
            GT<String> gts = new GT<String>();
            gts.var=2;
            System.out.println(gti.var);
        }
    }
    class GT<T>{
        public static int var=0;
        public void nothing(T x){}
    }
    

答案是——2！

由于经过类型擦除，所有的泛型类实例都关联到同一份字节码上，泛型类的所有静态变量是共享的。
