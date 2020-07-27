Java 1.7 之前 switch 参数可用类型为 short、byte、int、char，枚举类型之所以能使用其实是编译器层面实现的

编译器会将枚举 switch 转换为类似 

```
switch(s.ordinal()) { 
    case Status.START.ordinal() 
}
 
```

 
形式，所以实质还是 int 参数类型，感兴趣的可以自己写个使用枚举的 switch 代码然后通过 javap -v 去看下字节码就明白了。