## 概述

在本文中，我们将研究Java集合框架中的`ArrayList`类。我们将讨论它的属性、常用用例以及它的优缺点。

`ArrayList`包含在Java核心库中，因此不需要任何其他库。为了使用它，只需添加以下`import`语句:

```java
import java.util.ArrayList;
```

`List`表示一个有序的值序列，其中某些值可以重复。

`ArrayList`是构建在数组之上的列表实现之一，它能够在添加/删除元素时动态地增长和收缩。元素的索引可以很容易地从零开始访问。该实现具有以下特性:

* 随机访问需要O(1)
* 添加元素需要摊销固定时间O(1)
* 插入/删除需要O(n)
* 搜索未排序数组需要O(n)，排序数组需要O(log n)

## 创建

`ArrayList`有几个构造函数，我们将在本节中介绍它们。

首先，注意`ArrayList`是一个泛型类，因此可以使用任何类型对它进行参数化，编译器将确保，例如，不能将整数值放入字符串集合中。此外，从集合中检索元素时不需要强制转换元素。

其次，使用泛型接口列表作为变量类型是一个很好的实践，因为它将其与特定的实现解耦。

### 无参构造

```java
List<String> list = new ArrayList<>();
assertTrue(list.isEmpty());
```

我们只是创建一个空的ArrayList实例。

### 指定大小

```java
List<String> list = new ArrayList<>(20);
```

这里指定基础数组的初始长度。这可以帮助您在添加新项时避免不必要的调整大小。

### 集合构造

```java
Collection<Integer> numbers
  = IntStream.range(0, 10).boxed().collect(toSet());

List<Integer> list = new ArrayList<>(numbers);
assertEquals(10, list.size());
assertTrue(numbers.containsAll(list));
```

> 集合实例的元素用于填充基础数组。

## 添加元素

你可以在末尾或特定位置插入一个元素:

```java
List<Long> list = new ArrayList<>();

list.add(1L);
list.add(2L);
list.add(1, 3L);

assertThat(Arrays.asList(1L, 3L, 2L), equalTo(list));
```

你也可以同时插入一个集合或几个元素:

```java
List<Long> list = new ArrayList<>(Arrays.asList(1L, 2L, 3L));
LongStream.range(4, 10).boxed()
  .collect(collectingAndThen(toCollection(ArrayList::new), ys -> list.addAll(0, ys)));
assertThat(Arrays.asList(4L, 5L, 6L, 7L, 8L, 9L, 1L, 2L, 3L), equalTo(list));
```

## 遍历

有两种类型的迭代器可用:`Iterator`和`ListIterator`。

前者允许您在一个方向上遍历列表，而后者允许您在两个方向上遍历列表。

这里我们将只向您展示`ListIterator`:

```java
List<Integer> list = new ArrayList<>(
  IntStream.range(0, 10).boxed().collect(toCollection(ArrayList::new))
);
ListIterator<Integer> it = list.listIterator(list.size());
List<Integer> result = new ArrayList<>(list.size());
while (it.hasPrevious()) {
    result.add(it.previous());
}
 
Collections.reverse(list);
assertThat(result, equalTo(list));
```

您还可以使用迭代器搜索、添加或删除元素。

## 搜索

我们将演示如何使用集合进行搜索:

```java
List<String> list = LongStream.range(0, 16)
  .boxed()
  .map(Long::toHexString)
  .collect(toCollection(ArrayList::new));
List<String> stringsToSearch = new ArrayList<>(list);
stringsToSearch.addAll(list);
```

### 搜索未排序的列表

为了找到一个元素，您可以使用`indexOf()`或`lastIndexOf()`方法。它们都接受一个对象并返回`int`值:

```java
assertEquals(10, stringsToSearch.indexOf("a"));
assertEquals(26, stringsToSearch.lastIndexOf("a"));
```

你可以使用`Java 8 Stream API`过滤集合:

```java
Set<String> matchingStrings = new HashSet<>(Arrays.asList("a", "c", "9"));

List<String> result = stringsToSearch
  .stream()
  .filter(matchingStrings::contains)
  .collect(toCollection(ArrayList::new));

assertEquals(6, result.size());
```

也可以使用for循环或迭代器:

```java
Iterator<String> it = stringsToSearch.iterator();
Set<String> matchingStrings = new HashSet<>(Arrays.asList("a", "c", "9"));

List<String> result = new ArrayList<>();
while (it.hasNext()) {
    String s = it.next();
    if (matchingStrings.contains(s)) {
        result.add(s);
    }
}
```

## 搜索排序的列表

如果你有一个排序数组，那么你可以使用一个二进制搜索算法，它比线性搜索更快:

```java
List<String> copy = new ArrayList<>(stringsToSearch);
Collections.sort(copy);
int index = Collections.binarySearch(copy, "f");
assertThat(index, not(equalTo(-1)));
```

> 如果没有找到元素，那么将返回-1。

## 删除

为了删除一个元素，您应该找到它的索引，然后通过`remove()`方法执行删除。该方法的重载版本，它接受一个对象，搜索它，并执行删除第一个出现的相等元素:

```java
List<Integer> list = new ArrayList<>(
  IntStream.range(0, 10).boxed().collect(toCollection(ArrayList::new))
);
Collections.reverse(list);

list.remove(0);
assertThat(list.get(0), equalTo(8));

list.remove(Integer.valueOf(0));
assertFalse(list.contains(0));
```

但是在处理诸如`Integer`之类的装箱类型时要小心。为了删除一个特定的元素，你应该首先选择整型值，否则元素将被其索引删除。

您也可以使用前面提到的流api来删除几个项，但是我们这里不显示它。为此，我们将使用迭代器：

```java
Set<String> matchingStrings
 = HashSet<>(Arrays.asList("a", "b", "c", "d", "e", "f"));

Iterator<String> it = stringsToSearch.iterator();
while (it.hasNext()) {
    if (matchingStrings.contains(it.next())) {
        it.remove();
    }
}
```

## 总结

在这篇快速的文章中，我们了解了Java中的`ArrayList`。

我们展示了如何创建`ArrayList`实例，如何使用不同的方法添加、查找或删除元素。
