

 `try()` ⾥⾯有⼀个`return`语句， 那么后⾯的`finally{}`⾥⾯的code会不会被执⾏， 什么时候执⾏， 是在`return`前还是`return`后?


如果try中有return语句， 那么finally中的代码还是会执⾏。因为return表⽰的是要整个⽅法体返回， 所以，finally中的语句会在return之前执⾏。

但是return前执行的finally块内，对数据的修改效果对于引用类型和值类型会不同

```java
// 测试 修改值类型
static int f() {
	int ret = 0;
	try {
		return ret;  // 返回 0，finally内的修改效果不起作用
	} finally {
		ret++;
		System.out.println("finally执行");
	}
}

// 测试 修改引用类型
static int[] f2(){
	int[] ret = new int[]{0};
	try {
		return ret;  // 返回 [1]，finally内的修改效果起了作用
	} finally {
		ret[0]++;
		System.out.println("finally执行");
	}
}
```

