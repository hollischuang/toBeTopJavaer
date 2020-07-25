instanceof 是 Java 的一个二元操作符，类似于 ==，>，< 等操作符。

instanceof 是 Java 的保留关键字。它的作用是测试它左边的对象是否是它右边的类的实例，返回 boolean 的数据类型。

以下实例创建了 displayObjectClass() 方法来演示 Java instanceof 关键字用法：

    public static void displayObjectClass(Object o) {
      if (o instanceof Vector)
         System.out.println("对象是 java.util.Vector 类的实例");
      else if (o instanceof ArrayList)
         System.out.println("对象是 java.util.ArrayList 类的实例");
      else
        System.out.println("对象是 " + o.getClass() + " 类的实例");
   }