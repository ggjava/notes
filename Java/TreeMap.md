## 概述

在本文中，我们将探讨来自`Java`集合框架`Map`接口的`TreeMap`实现。

`TreeMap`它根据键的自然顺序对其元素进行排序，也可以根据用户提供了比较器排序。

在此之前，我们已经介绍了`HashMap`的实现，我们将认识到关于这些类如何工作的相当多的信息是类似的。

在继续阅读本文之前，强烈推荐阅读上面提到的文章。

## 默认排序

默认情况下，`TreeMap`根据其自然顺序对所有元素进行排序。对于整数，这意味着升序，对于字符串，这意味着字母顺序。

让我们看看测试中的自然顺序:

```java
@Test
public void givenTreeMap_whenOrdersEntriesNaturally_thenCorrect() {
    TreeMap<Integer, String> map = new TreeMap<>();
    map.put(3, "val");
    map.put(2, "val");
    map.put(1, "val");
    map.put(5, "val");
    map.put(4, "val");

    assertEquals("[1, 2, 3, 4, 5]", map.keySet().toString());
}
```

注意，我们以非有序的方式放置了整数键，但是在检索键集时，我们确认它们确实是按升序维护的。这是整数的自然顺序。

同样的，当我们使用字符串时，它们会按自然顺序排序，即按字母顺序:

```java
@Test
public void givenTreeMap_whenOrdersEntriesNaturally_thenCorrect2() {
    TreeMap<String, String> map = new TreeMap<>();
    map.put("c", "val");
    map.put("b", "val");
    map.put("a", "val");
    map.put("e", "val");
    map.put("d", "val");

    assertEquals("[a, b, c, d, e]", map.keySet().toString());
}
```

## 自定义排序

如果我们对`TreeMap`的自然顺序不满意，我们还可以在构建时通过比较器定义自己的排序规则。

在下面的例子中，我们希望整数键按降序排列:

```java
@Test
public void givenTreeMap_whenOrdersEntriesByComparator_thenCorrect() {
    TreeMap<Integer, String> map = 
      new TreeMap<>(Comparator.reverseOrder());
    map.put(3, "val");
    map.put(2, "val");
    map.put(1, "val");
    map.put(5, "val");
    map.put(4, "val");

    assertEquals("[5, 4, 3, 2, 1]", map.keySet().toString());
}
```

`HashMap`不保证存储的键的顺序，也不保证这个顺序在一段时间内保持不变，但是`TreeMap`保证键总是按照指定的顺序排序。

## 排序的重要性

我们现在知道`TreeMap`以排序的顺序存储它的所有元素。我们可以执行以下查询;查找“最大”，查找“最小”，查找所有小于或大于某个值的键，等等。

以下守则只涵盖其中一小部分个案:

```java
@Test
public void givenTreeMap_whenPerformsQueries_thenCorrect() {
    TreeMap<Integer, String> map = new TreeMap<>();
    map.put(3, "val");
    map.put(2, "val");
    map.put(1, "val");
    map.put(5, "val");
    map.put(4, "val");

    Integer highestKey = map.lastKey();
    Integer lowestKey = map.firstKey();
    Set<Integer> keysLessThan3 = map.headMap(3).keySet();
    Set<Integer> keysGreaterThanEqTo3 = map.tailMap(3).keySet();

    assertEquals(new Integer(5), highestKey);
    assertEquals(new Integer(1), lowestKey);
    assertEquals("[1, 2]", keysLessThan3.toString());
    assertEquals("[3, 4, 5]", keysGreaterThanEqTo3.toString());
}
```

## TreeMap的内部实现

`TreeMap`实现了`NavigableMap`接口，内部基于红黑树:

```java
public class TreeMap<K,V> extends AbstractMap<K,V>
  implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

红黑树的原则超出了本文的范围，但是，为了理解它们如何适应`TreeMap`，需要记住一些关键的事情。

首先，红黑树是由节点组成的数据结构;想象一棵倒立的芒果树，它的根在天空中，树枝向下生长。根将包含添加到树中的第一个元素。

规则是，从根节点开始，任何节点的左分支中的任何元素总是小于节点本身中的元素。右边的总是更大。如前所述，定义大于或小于的是由元素的自然顺序或构造时定义的比较器决定的。

这个规则保证了`TreeMap`的元素始终是有序和可预测的。

其次，红黑树是一种自平衡二叉搜索树。该属性和上述属性保证了搜索、获取、放置和删除等基本操作花费对数时间`O(log n)`。

自我平衡是关键。当我们不断地插入和删除条目时，想象一下这棵树一边长了，另一边短了。

这意味着一个操作在较短的分支上花费的时间较短而在离根最远的分支上花费的时间较长，这是我们不希望发生的。

因此，在红黑树的设计中要注意这一点。对于每一次插入和删除操作，树在任何边的最大高度都保持在`O(log n)`处，即树本身保持连续平衡。

## 总结

在本文中，我们探讨了Java `TreeMap`类的使用及其内部实现。
