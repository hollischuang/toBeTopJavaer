Commons Collections增强了Java Collections Framework。 它提供了几个功能，使收集处理变得容易。 它提供了许多新的接口，实现和实用程序。 Commons Collections的主要功能如下

* Bag - Bag界面简化了每个对象具有多个副本的集合。

* BidiMap - BidiMap接口提供双向映射，可用于使用值使用键或键查找值。

* MapIterator - MapIterator接口提供简单而容易的迭代迭代。

* Transforming Decorators - 转换装饰器可以在将集合添加到集合时更改集合的每个对象。

* Composite Collections - 在需要统一处理多个集合的情况下使用复合集合。

* Ordered Map - 有序地图保留添加元素的顺序。

* Ordered Set - 有序集保留了添加元素的顺序。

* Reference map - 参考图允许在密切控制下对键/值进行垃圾收集。

* Comparator implementations - 可以使用许多Comparator实现。

* Iterator implementations - 许多Iterator实现都可用。

* Adapter Classes - 适配器类可用于将数组和枚举转换为集合。

* Utilities - 实用程序可用于测试测试或创建集合的典型集合论属性，例如union，intersection。 支持关闭。

### Commons Collections - Bag

Bag定义了一个集合，用于计算对象在集合中出现的次数。 例如，如果Bag包含{a，a，b，c}，则getCount（“a”）将返回2，而uniqueSet（）将返回唯一值。


```java

import org.apache.commons.collections4.Bag;
import org.apache.commons.collections4.bag.HashBag;
public class BagTester {
   public static void main(String[] args) {
      Bag<String> bag = new HashBag<>();
      //add "a" two times to the bag.
      bag.add("a" , 2);
      //add "b" one time to the bag.
      bag.add("b");
      //add "c" one time to the bag.
      bag.add("c");
      //add "d" three times to the bag.
      bag.add("d",3);
      //get the count of "d" present in bag.
      System.out.println("d is present " + bag.getCount("d") + " times.");
      System.out.println("bag: " +bag);
      //get the set of unique values from the bag
      System.out.println("Unique Set: " +bag.uniqueSet());
      //remove 2 occurrences of "d" from the bag
      bag.remove("d",2);
      System.out.println("2 occurences of d removed from bag: " +bag);
      System.out.println("d is present " + bag.getCount("d") + " times.");
      System.out.println("bag: " +bag);
      System.out.println("Unique Set: " +bag.uniqueSet());
   }
}

```

它将打印以下结果:

```

d is present 3 times.
bag: [2:a,1:b,1:c,3:d]
Unique Set: [a, b, c, d]
2 occurences of d removed from bag: [2:a,1:b,1:c,1:d]
d is present 1 times.
bag: [2:a,1:b,1:c,1:d]
Unique Set: [a, b, c, d]

```

### Commons Collections - BidiMap

使用双向映射，可以使用值查找键，并且可以使用键轻松查找值。

```java

import org.apache.commons.collections4.BidiMap;
import org.apache.commons.collections4.bidimap.TreeBidiMap;
public class BidiMapTester {
   public static void main(String[] args) {
      BidiMap<String, String> bidi = new TreeBidiMap<>();
      bidi.put("One", "1");
      bidi.put("Two", "2");
      bidi.put("Three", "3");
      System.out.println(bidi.get("One")); 
      System.out.println(bidi.getKey("1"));
      System.out.println("Original Map: " + bidi);
      bidi.removeValue("1"); 
      System.out.println("Modified Map: " + bidi);
      BidiMap<String, String> inversedMap = bidi.inverseBidiMap();  
      System.out.println("Inversed Map: " + inversedMap);
   }
}
```

它将打印以下结果。
```
1
One
Original Map: {One=1, Three=3, Two=2}
Modified Map: {Three=3, Two=2}
Inversed Map: {2=Two, 3=Three}
```

### Commons Collections - MapIterator

JDK Map接口很难迭代，因为迭代要在EntrySet或KeySet对象上完成。 MapIterator提供了对Map的简单迭代。

```java

import org.apache.commons.collections4.IterableMap;
import org.apache.commons.collections4.MapIterator;
import org.apache.commons.collections4.map.HashedMap;
public class MapIteratorTester {
   public static void main(String[] args) {
      IterableMap<String, String> map = new HashedMap<>();
      map.put("1", "One");
      map.put("2", "Two");
      map.put("3", "Three");
      map.put("4", "Four");
      map.put("5", "Five");
      MapIterator<String, String> iterator = map.mapIterator();
      while (iterator.hasNext()) {
         Object key = iterator.next();
         Object value = iterator.getValue();
         System.out.println("key: " + key);
         System.out.println("Value: " + value);
         iterator.setValue(value + "_");
      }
      System.out.println(map);
   }
}
```

它将打印以下结果。
```
key: 3
Value: Three
key: 5
Value: Five
key: 2
Value: Two
key: 4
Value: Four
key: 1
Value: One
{3=Three_, 5=Five_, 2=Two_, 4=Four_, 1=One_}
```

### Commons Collections - OrderedMap

OrderedMap是地图的新接口，用于保留添加元素的顺序。 LinkedMap和ListOrderedMap是两个可用的实现。 此接口支持Map的迭代器，并允许在Map中向前或向后迭代两个方向。

```java
import org.apache.commons.collections4.OrderedMap;
import org.apache.commons.collections4.map.LinkedMap;
public class OrderedMapTester {
   public static void main(String[] args) {
      OrderedMap<String, String> map = new LinkedMap<String, String>();
      map.put("One", "1");
      map.put("Two", "2");
      map.put("Three", "3");
      System.out.println(map.firstKey());
      System.out.println(map.nextKey("One"));
      System.out.println(map.nextKey("Two"));  
   }
}
```

它将打印以下结果。

```
One
Two
Three

```

### Commons Collections - Ignore NULL

Apache Commons Collections库的CollectionUtils类为常见操作提供了各种实用方法，涵盖了广泛的用例。 它有助于避免编写样板代码。 这个库在jdk 8之前非常有用，因为Java 8的Stream API现在提供了类似的功能。


```java
import java.util.LinkedList;
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class CollectionUtilsTester {
   public static void main(String[] args) {
      List<String> list = new LinkedList<String>();
      CollectionUtils.addIgnoreNull(list, null);
      CollectionUtils.addIgnoreNull(list, "a");
      System.out.println(list);
      if(list.contains(null)) {
         System.out.println("Null value is present");
      } else {
         System.out.println("Null value is not present");
      }
   }
}
```

它将打印以下结果。
```
[a]
Null value is not present
```

### Merge & Sort

Apache Commons Collections库的CollectionUtils类为常见操作提供了各种实用方法，涵盖了广泛的用例。 它有助于避免编写样板代码。 这个库在jdk 8之前非常有用，因为Java 8的Stream API现在提供了类似的功能。

```java

import java.util.Arrays;
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class CollectionUtilsTester {
   public static void main(String[] args) {
      List<String> sortedList1 = Arrays.asList("A","C","E");
      List<String> sortedList2 = Arrays.asList("B","D","F");
      List<String> mergedList = CollectionUtils.collate(sortedList1, sortedList2);
      System.out.println(mergedList); 
   }
}
```


它将打印以下结果。
```
[A, B, C, D, E, F]
```

### 安全空检查(Safe Empty Checks)

Apache Commons Collections库的CollectionUtils类为常见操作提供了各种实用方法，涵盖了广泛的用例。 它有助于避免编写样板代码。 这个库在jdk 8之前非常有用，因为Java 8的Stream API现在提供了类似的功能。


```java
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class CollectionUtilsTester {
   public static void main(String[] args) {
      List<String> list = getList();
      System.out.println("Non-Empty List Check: " + checkNotEmpty1(list));
      System.out.println("Non-Empty List Check: " + checkNotEmpty1(list));
   }
   static List<String> getList() {
      return null;
   } 
   static boolean checkNotEmpty1(List<String> list) {
      return !(list == null || list.isEmpty());
   }
   static boolean checkNotEmpty2(List<String> list) {
      return CollectionUtils.isNotEmpty(list);
   } 
}
```

它将打印以下结果。
```
Non-Empty List Check: false
Non-Empty List Check: false
```

### Commons Collections - Inclusion

检查列表是否是另一个列表的一部分

```java
import java.util.Arrays;
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class CollectionUtilsTester {
   public static void main(String[] args) {
      //checking inclusion
      List<String> list1 = Arrays.asList("A","A","A","C","B","B");
      List<String> list2 = Arrays.asList("A","A","B","B");
      System.out.println("List 1: " + list1);
      System.out.println("List 2: " + list2);
      System.out.println("Is List 2 contained in List 1: " 
         + CollectionUtils.isSubCollection(list2, list1));
   }
}
```

它将打印以下结果。

```
List 1: [A, A, A, C, B, B]
List 2: [A, A, B, B]
Is List 2 contained in List 1: true
```

### Commons Collections - Intersection

用于获取两个集合（交集）之间的公共对象

```java
import java.util.Arrays;
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class CollectionUtilsTester {
   public static void main(String[] args) {
      //checking inclusion
      List<String> list1 = Arrays.asList("A","A","A","C","B","B");
      List<String> list2 = Arrays.asList("A","A","B","B");
      System.out.println("List 1: " + list1);
      System.out.println("List 2: " + list2);
      System.out.println("Commons Objects of List 1 and List 2: " 
         + CollectionUtils.intersection(list1, list2));
   }
}
```

它将打印以下结果。


```
List 1: [A, A, A, C, B, B]
List 2: [A, A, B, B]
Commons Objects of List 1 and List 2: [A, A, B, B]

```

### Commons Collections - Subtraction
通过从其他集合中减去一个集合的对象来获取新集合

```java
import java.util.Arrays;
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class CollectionUtilsTester {
   public static void main(String[] args) {
      //checking inclusion
      List<String> list1 = Arrays.asList("A","A","A","C","B","B");
      List<String> list2 = Arrays.asList("A","A","B","B");
      System.out.println("List 1: " + list1);
      System.out.println("List 2: " + list2);
      System.out.println("List 1 - List 2: " 
         + CollectionUtils.subtract(list1, list2));
   }
}
```

它将打印以下结果。
```
List 1: [A, A, A, C, B, B]
List 2: [A, A, B, B]
List 1 - List 2: [A, C]
```

### Commons Collections - Union

用于获取两个集合的并集

```java
import java.util.Arrays;
import java.util.List;
import org.apache.commons.collections4.CollectionUtils;
public class CollectionUtilsTester {
   public static void main(String[] args) {
      //checking inclusion
      List<String> list1 = Arrays.asList("A","A","A","C","B","B");
      List<String> list2 = Arrays.asList("A","A","B","B");
      System.out.println("List 1: " + list1);
      System.out.println("List 2: " + list2);
      System.out.println("Union of List 1 and List 2: " 
         + CollectionUtils.union(list1, list2));
   }
}
```

它将打印以下结果。
``` 
List 1: [A, A, A, C, B, B]
List 2: [A, A, B, B]
Union of List 1 and List 2: [A, A, A, B, B, C]

```

原文地址：https://iowiki.com/commons_collections/commons_collections_index.html