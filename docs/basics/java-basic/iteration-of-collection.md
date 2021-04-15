Collection的迭代有很多种方式：

1、通过普通for循环迭代

2、通过增强for循环迭代

3、使用Iterator迭代

4、使用Stream迭代


```
List<String> list = ImmutableList.of("Hollis", "hollischuang");

// 普通for循环遍历
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}

//增强for循环遍历
for (String s : list) {
    System.out.println(s);
}

//Iterator遍历
Iterator it = list.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

//Stream 遍历
list.forEach(System.out::println);

list.stream().forEach(System.out::println);
``` 
