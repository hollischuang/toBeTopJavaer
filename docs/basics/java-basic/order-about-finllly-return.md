

 `try()` ⾥⾯有⼀个`return`语句， 那么后⾯的`finally{}`⾥⾯的code会不会被执⾏， 什么时候执⾏， 是在`return`前还是`return`后?


如果try中有return语句， 那么finally中的代码还是会执⾏。因为return表⽰的是要整个⽅法体返回， 所以，finally中的语句会在return之前执⾏。