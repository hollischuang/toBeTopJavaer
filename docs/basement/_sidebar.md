* JVM
        
    * JVM内存结构
        
        * 运行时数据区
        
        * 运行时数据区哪些是线程独享
        
        * 堆和栈区别
        
        * 方法区在不同版本JDK中的位置
        
        * 堆外内存
          
        * TLAB
          
        * [Java中的对象一定在堆上分配吗？](/basement/jvm/stack-alloc.md)
        
    * 垃圾回收
        
        * GC算法：标记清除、引用计数、复制、标记压缩、分代回收、增量式回收
        
        * GC参数
        
        * 对象存活的判定
        
        * 垃圾收集器（CMS、G1、ZGC、Epsilon）
        
    * JVM参数及调优
                
        * -Xmx
        
        * -Xmn
        
        * -Xms
        
        * -Xss
        
        * -XX:SurvivorRatio
    
        * -XX:PermSize
        
        * -XX:MaxPermSize
        
        * -XX:MaxTenuringThreshold
                
    * Java对象模型
        
        * oop-klass
        
        * 对象头
        
    * HotSpot
        
        * 即时编译器
        
        * 编译优化
        
    * Java内存模型
        
        * 计算机内存模型
        
        * 缓存一致性
        
        * MESI协议
        
        * 可见性
        
        * 原子性
        
        * 顺序性
        
        * happens-before
        
        * as-if-serial
        
        * 内存屏障
        
        * synchronized
        
        * volatile
        
        * final
        
        * 锁

* 虚拟机性能监控与故障处理工具
        
    * jps
    
    * jstack
    
    * jmap
    
    * jstat
    
    * jconsole
    
    * jinfo
    
    * jhat
    
    * javap
    
    * btrace
    
    * TProfiler
    
    * jlink
    
    * Arthas
        
* 类加载机制
        
    * classLoader
    
    * 类加载过程是线程安全的吗？
    
    * 类加载过程
    
    * 双亲委派（破坏双亲委派）
    
    * 模块化（jboss modules、osgi、jigsaw）
    
* 打包工具
    
    * jar
    
    * jlink
    
    * jpackage
        
* 编译与反编译
        
    * 什么是编译
    
    * 什么是反编译
    
    * 编译工具：javac
            
    * 反编译工具：javap 、jad 、CRF
    
* JIT

    * JIT优化（逃逸分析、栈上分配、标量替换、锁优化）
    
    