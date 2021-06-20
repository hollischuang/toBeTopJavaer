replace、replaceAll和replaceFirst是Java中常用的替换字符的方法,它们的方法定义是：

replace(CharSequence target, CharSequence replacement) ，用replacement替换所有的target，两个参数都是字符串。

replaceAll(String regex, String replacement) ，用replacement替换所有的regex匹配项，regex很明显是个正则表达式，replacement是字符串。

replaceFirst(String regex, String replacement) ，基本和replaceAll相同，区别是只替换第一个匹配项。

可以看到，其中replaceAll以及replaceFirst是和正则表达式有关的，而replace和正则表达式无关。

replaceAll和replaceFirst的区别主要是替换的内容不同，replaceAll是替换所有匹配的字符，而replaceFirst()仅替换第一次出现的字符

### 用法例子

    String string = "abc123adb23456aa";
    System.out.println(string);//abc123adb23456aa

    //使用replace将a替换成H
    System.out.println(string.replace("a","H"));//Hbc123Hdb23456HH
    //使用replaceFirst将第一个a替换成H
    System.out.println(string.replaceFirst("a","H"));//Hbc123adb23456aa
    //使用replace将a替换成H
    System.out.println(string.replaceAll("a","H"));//Hbc123Hdb23456HH

    //使用replaceFirst将第一个数字替换成H
    System.out.println(string.replaceFirst("\\d","H"));//abcH23adb23456aa
    //使用replaceAll将所有数字替换成H
    System.out.println(string.replaceAll("\\d","H"));//abcHHHadbHHHHHaa