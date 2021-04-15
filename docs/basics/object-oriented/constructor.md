构造函数，是一种特殊的方法。 主要用来在创建对象时初始化对象， 即为对象成员变量赋初始值，总与new运算符一起使用在创建对象的语句中。 

    /**
    * 矩形
    */
    class Rectangle {
    
         /**
          * 构造函数
          */
         public Rectangle(int length, int width) {
             this.length = length;
             this.width = width;
         }
         
         public static void main (String []args){
            //使用构造函数创建对象
            Rectangle rectangle = new Rectangle(10,5);
            
         }
    }
    
    

特别的一个类可以有多个构造函数，可根据其参数个数的不同或参数类型的不同来区分它们即构造函数的重载。
         

构造函数跟一般的实例方法十分相似；但是与其它方法不同，构造器没有返回类型，不会被继承，且可以有范围修饰符。

构造器的函数名称必须和它所属的类的名称相同。它承担着初始化对象数据成员的任务。

如果在编写一个可实例化的类时没有专门编写构造函数，多数编程语言会自动生成缺省构造器（默认构造函数）。默认构造函数一般会把成员变量的值初始化为默认值，如int -> 0，Integer -> null。


如果在编写一个可实例化的类时没有专门编写构造函数，默认情况下，一个Java类中会自动生成一个默认无参构造函数。默认构造函数一般会把成员变量的值初始化为默认值，如int -> 0，Integer -> null。

但是，如果我们手动在某个类中定义了一个有参数的构造函数，那么这个默认的无参构造函数就不会自动添加了。需要手动创建！

    /**
    * 矩形
    */
    class Rectangle {
    
         /**
          * 构造函数
          */
         public Rectangle(int length, int width) {
             this.length = length;
             this.width = width;
         }
         
         /**
          * 无参构造函数
          */
         public Rectangle() {
             
         }
    }
