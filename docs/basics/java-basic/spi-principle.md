看ServiceLoader类的签名类的成员变量：

    public final class ServiceLoader<S> implements Iterable<S>{
    private static final String PREFIX = "META-INF/services/";
    
        // 代表被加载的类或者接口
        private final Class<S> service;
    
        // 用于定位，加载和实例化providers的类加载器
        private final ClassLoader loader;
    
        // 创建ServiceLoader时采用的访问控制上下文
        private final AccessControlContext acc;
    
        // 缓存providers，按实例化的顺序排列
        private LinkedHashMap<String,S> providers = new LinkedHashMap<>();
    
        // 懒查找迭代器
        private LazyIterator lookupIterator;
    
        ......
    }
    
参考具体源码，梳理了一下，实现的流程如下：

1 应用程序调用ServiceLoader.load方法

ServiceLoader.load方法内先创建一个新的ServiceLoader，并实例化该类中的成员变量，包括：

loader(ClassLoader类型，类加载器)

acc(AccessControlContext类型，访问控制器)

providers(LinkedHashMap类型，用于缓存加载成功的类)

lookupIterator(实现迭代器功能)

2 应用程序通过迭代器接口获取对象实例

ServiceLoader先判断成员变量providers对象中(LinkedHashMap类型)是否有缓存实例对象，如果有缓存，直接返回。
如果没有缓存，执行类的装载：

读取META-INF/services/下的配置文件，获得所有能被实例化的类的名称

通过反射方法Class.forName()加载类对象，并用instance()方法将类实例化

把实例化后的类缓存到providers对象中(LinkedHashMap类型）

然后返回实例对象。