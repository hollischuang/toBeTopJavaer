步骤1、定义一组接口 (假设是org.foo.demo.IShout)，并写出接口的一个或多个实现，(假设是org.foo.demo.animal.Dog、org.foo.demo.animal.Cat)。

    public interface IShout {
        void shout();
    }
    public class Cat implements IShout {
        @Override
        public void shout() {
            System.out.println("miao miao");
        }
    }
    public class Dog implements IShout {
        @Override
        public void shout() {
            System.out.println("wang wang");
        }
    }

步骤2、在 src/main/resources/ 下建立 /META-INF/services 目录， 新增一个以接口命名的文件 (org.foo.demo.IShout文件)，内容是要应用的实现类（这里是org.foo.demo.animal.Dog和org.foo.demo.animal.Cat，每行一个类）。

    org.foo.demo.animal.Dog
    org.foo.demo.animal.Cat

步骤3、使用 ServiceLoader 来加载配置文件中指定的实现。

    public class SPIMain {
        public static void main(String[] args) {
            ServiceLoader<IShout> shouts = ServiceLoader.load(IShout.class);
            for (IShout s : shouts) {
                s.shout();
            }
        }
    }

代码输出：

    wang wang
    miao miao