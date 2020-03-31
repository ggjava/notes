## 为什么要数组转List

在java中，我们经常需要将数组转换为List。这主要是由于以下原因:

* 数组的长度是固定的，数组的大小不能动态地增加，您需要提供在数组满时增加数组大小的方法，而列表满时自动增加数组大小。
* 数组删除元素比较复杂，而使用List则很容易做到这一点。
* 如果需要检查数组中元素的存在性，则需要遍历数组，效率较差。
* 您正在接收来自外部库的数据，并在处理之后将其发送到另一个服务。现在接收到的数据是数组的形式，而另一端所期望的是列表，您别无选择，只能将数组转换为列表。

在需要将数组转换为列表的代码中，可能会出现上述任何场景。这篇文章将提供一些可以使用的方法。

## 使用Arrays类

java.util.Arrays有一个asList方法，该方法以数组为参数，返回一个用数组内容填充的List。这是一个静态方法，可以在不使用java.util.Arrays的任何实例的情况下调用。

```java
import java.util.Arrays;
import java.util.List;

public class ArrayToListConverter {

   public static void main(String[] args) {
        // create string array
        String[] array = {"rainbow", "mountains", "ocean"};
        // convert it to a list
        List list = Arrays.asList(array);
        // print list
        System.out.println(list);
        // check if it is actually a list
        System.out.println(list instanceof List);
   }
}
```

输出：

```
[rainbow, mountains, ocean]
true
```

> 注意，当对java.util.List进行检查时，instanceof返回true。您不能向asList方法返回的List中添加新元素。试图添加新元素将引发java.lang.UnsupportedOperationException。这是因为asList方法返回的列表是一个固定大小的列表。

如果需要向新创建的List中添加元素，则需要创建一个新的List，如下所示：

```java
// convert array to list
List list = Arrays.asList(array);
// create a new list with the contents of existing list
List newList = new ArrayList(list);
// add a new element to list
newList.add("river");
```

> 此方法只能用于对象数组，而不能用于诸如int、float、char或double等基本类型数组。应该使用Integer, Float, Double and Character等包装器类型的数组。

## 使用Collections方法

java.util.Collections类具有一个addAll方法，该方法接受另一个集合和一个数组作为参数，并将数组的元素添加到提供的集合中。

创建一个新的空List，并将其连同数组一起传递给addAll方法。数组的内容被复制到List

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ArrayToListConverter {

   public static void main(String[] args) {
      // create an array
      String[] array = {"rainbow", "mountains", "ocean"};
      // create an empty list
      List list = new ArrayList();
      // add array elements to list
      Collections.addAll(list, array);
      System.out.println(list);
   }
}
```

输出：

```
[rainbow, mountains, ocean]
```

> 调用addAll方法时List和数组一个为null，将抛出java.lang.NullPointerException.

## 数组遍历

这是将数组转换为列表的传统方法。在此方法中，创建一个空列表。使用循环遍历数组。

在每次迭代中，使用数组元素的add方法将数组元素添加到列表中。循环完成后，list将包含数组的所有元素。

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayToListConverter {

    public static void main(String[] args) {
        String[] array = { "rainbow", "mountains", "ocean" };
        // create an empty list
        List list = new ArrayList();
        // iterate over array
        for (String element : array) {
            // add array element to list
            list.add(element);
        }
        System.out.println(list);
   }
}
```

输出：

```
[rainbow, mountains, ocean]
```

## 使用stream

Java 8引入了流的概念

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ArrayToListConverter {

    public static void main(String[] args) {
        String[] array = { "rainbow", "mountains", "ocean" };
        // get a stream of array elements
        Stream stream = Arrays.stream(array);
        // get the array as list
        List list = stream.collect(Collectors.toList());
        System.out.println(list);
    }
}
```

输出：

```
[rainbow, mountains, ocean]
```

上面代码还可以写成：

```java
List list = Arrays.stream(array).collect(Collectors.toList());
```

> 请记住List的泛型类型应该与数组的类型相同。

希望本文列出的方法对您有所帮助。