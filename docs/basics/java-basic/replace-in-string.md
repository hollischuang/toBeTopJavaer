replace、replaceAll和replaceFirst是Java中常用的替换字符的方法,它们的方法定义是：

replace(CharSequence target, CharSequence replacement) ，用replacement替换所有的target，两个参数都是字符串。

replaceAll(String regex, String replacement) ，用replacement替换所有的regex匹配项，regex很明显是个正则表达式，replacement是字符串。

replaceFirst(String regex, String replacement) ，基本和replaceAll相同，区别是只替换第一个匹配项。

可以看到，其中replaceAll以及replaceFirst是和正则表达式有关的，而replace和正则表达式无关。

replaceAll和replaceFirst的区别主要是替换的内容不同，replaceAll是替换所有匹配的字符，而replaceFirst()仅替换第一次出现的字符

### 用法例子

一以下例子参考：http://www.51gjie.com/java/771.html

1. replaceAll() 替换符合正则的所有文字

```
//文字替换（全部） 
Pattern pattern = Pattern.compile("正则表达式"); 
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World"); 
//替换所有符合正则的数据 
System.out.println(matcher.replaceAll("Java")); 

```
   

2. replaceFirst() 替换第一个符合正则的数据

```
//文字替换（首次出现字符） 
Pattern pattern = Pattern.compile("正则表达式"); 
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World"); 
//替换第一个符合正则的数据 
System.out.println(matcher.replaceFirst("Java")); 
    
```
    
3. replaceAll()替换所有html标签

```
//去除html标记 
Pattern pattern = Pattern.compile("<.+?>", Pattern.DOTALL); 
Matcher matcher = pattern.matcher("<a href=\"index.html\">主页</a>"); 
String string = matcher.replaceAll(""); 
System.out.println(string); 

```

4. replaceAll() 替换指定文字 
```
//替换指定{}中文字 
String str = "Java目前的发展史是由{0}年-{1}年";
String[][] object = {
    new String[] {
        "\\{0\\}",
        "1995"
    },
    new String[] {
        "\\{1\\}",
        "2007"
    }
};
System.out.println(replace(str, object));
public static String replace(final String sourceString, Object[] object) {
    String temp = sourceString;
    for (int i = 0; i < object.length; i++) {
        String[] result = (String[]) object[i];
        Pattern pattern = Pattern.compile(result[0]);
        Matcher matcher = pattern.matcher(temp);
        temp = matcher.replaceAll(result[1]);
    }
    return temp;
}

```

5. replace()替换字符串

```
System.out.println("abac".replace("a", "\\a")); //\ab\ac 
```
