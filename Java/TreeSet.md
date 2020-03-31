##  概述

在本文中，我们将了解Java集合框架的组成部分和最流行的集合实现之一——`TreeSet`。

简单地说，`TreeSet`是一个排序的集合，它扩展了`AbstractSet`类并实现了`NavigableSet`接口。

以下是该实现最重要方面的快速总结:

* 它存储独特的元素
* 它不保留元素的插入顺序
* 它按升序排列元素
* 不是线程安全的

在这个实现中，对象按照它们的自然顺序升序排序和存储。`TreeSet`使用自平衡二叉搜索树，更确切地说是红黑树。

简单地说，作为一个自平衡的二叉搜索树，二叉树的每个节点都包含一个额外的位，用来识别节点的颜色，该颜色可以是红色，也可以是黑色。在随后的插入和删除过程中，这些“颜色”位有助于确保树保持或多或少的平衡。

那么，让我们创建一个`TreeSet`实例:

```java
Set<String> treeSet = new TreeSet<>();
```

## 构造函数

我们可以用构造函数构造一个`TreeSet`，让我们定义元素排序的顺序，通过使用`Comparable`或`Comparator`:

```java
Set<String> treeSet = new TreeSet<>(Comparator.comparing(String::length));
```

虽然`TreeSet`不是线程安全的，但它可以在外部使用`collections`进行同步。

```java
Set<String> syncTreeSet = Collections.synchronizedSet(treeSet);
```

好了，现在我们已经对如何创建`TreeSet`实例有了一个清晰的概念，接下来让我们看看可用的公共操作。

## API

### add()

`add()`方法可用于向`TreeSet`添加元素。如果添加了一个元素，该方法将返回`true`，否则返回`false`。

该方法的契约规定，只有在集合中没有元素时才添加元素。

让我们添加一个元素到`TreeSet`:

```java
@Test
public void whenAddingElement_shouldAddElement() {
    Set<String> treeSet = new TreeSet<>();

    assertTrue(treeSet.add("String Added"));
 }
```

`add`方法非常重要，因为该方法的实现细节说明了`TreeSet`如何在内部工作，它如何利用`TreeMap`的`put`方法来存储元素:

```java
public boolean add(E e) {
    return m.put(e, PRESENT) == null;
}
```

变量m指的是一个内部支持的`TreeMap`(注意`TreeMap`实现了`NavigateableMap`):

```java
private transient NavigableMap<E, Object> m;
```

因此，`TreeSet`内部依赖于一个支持的`NavigableMap`，当`TreeSet`的一个实例被创建时，它会被TreeMap的一个实例初始化:

```java
public TreeSet() {
    this(new TreeMap<E,Object>());
}
```

### contains()

`contains()`方法用于检查给定`TreeSet`中是否存在给定元素。如果找到该元素，则返回`true`，否则返回`false`。

让我们看看`contains()`的使用:

```java
@Test
public void whenCheckingForElement_shouldSearchForElement() {
    Set<String> treeSetContains = new TreeSet<>();
    treeSetContains.add("String Added");

    assertTrue(treeSetContains.contains("String Added"));
}
```

### remove()

如果指定的元素存在，则使用`remove()`方法从集合中删除它。

如果集合包含指定的元素，则此方法返回`true`。

让我们看看它的使用:

```java
@Test
public void whenRemovingElement_shouldRemoveElement() {
    Set<String> removeFromTreeSet = new TreeSet<>();
    removeFromTreeSet.add("String Added");
 
    assertTrue(removeFromTreeSet.remove("String Added"));
}
```

### clear()

如果我们想从一个集合中删除所有的元素，我们可以使用`clear()`方法:

```java
@Test
public void whenClearingTreeSet_shouldClearTreeSet() {
    Set<String> clearTreeSet = new TreeSet<>();
    clearTreeSet.add("String Added");
    clearTreeSet.clear();
  
    assertTrue(clearTreeSet.isEmpty());
}
```

### size()

`size()`方法用于集合中元素的数量。这是API中的基本方法之一:

```java
@Test
public void whenCheckingTheSizeOfTreeSet_shouldReturnThesize() {
    Set<String> treeSetSize = new TreeSet<>();
    treeSetSize.add("String Added");
  
    assertEquals(1, treeSetSize.size());
}
```

### isEmpty()

`isEmpty()`方法可以用来判断给定的集合实例是否为空:

```java
@Test
public void whenCheckingForEmptyTreeSet_shouldCheckForEmpty() {
    Set<String> emptyTreeSet = new TreeSet<>();
     
    assertTrue(emptyTreeSet.isEmpty());
}
```

### iterator()

`iterator()`方法返回一个迭代器，以升序对集合中的元素进行迭代。

我们可以在这里观察到升序迭代顺序:

```java
@Test
public void whenIteratingTreeSet_shouldIterateTreeSetInAscendingOrder() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        System.out.println(itr.next());
    }
}
```

此外，`TreeSet`允许我们按降序遍历集合。

让我们看看实际情况:

```java
@Test
public void whenIteratingTreeSet_shouldIterateTreeSetInDescendingOrder() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.descendingIterator();
    while (itr.hasNext()) {
        System.out.println(itr.next());
    }
}
```

如果在以任何方式(除了通过迭代器的`remove()`方法)创建迭代器之后的任何时间修改集合，迭代器将抛出`ConcurrentModificationException`。

让我们为此创建一个测试:

```java
@Test(expected = ConcurrentModificationException.class)
public void whenModifyingTreeSetWhileIterating_shouldThrowException() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        itr.next();
        treeSet.remove("Second");
    }
}
```

或者，如果我们使用了迭代器的删除方法，那么我们就不会遇到异常:

```java
@Test
public void whenRemovingElementUsingIterator_shouldRemoveElement() {
  
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        String element = itr.next();
        if (element.equals("Second"))
           itr.remove();
    }
  
    assertEquals(2, treeSet.size());
}
```

对于迭代器的快速故障行为没有保证，因为在非同步并发修改的情况下不可能做出任何严格的保证。

### first()

如果树集的第一个元素不是空的，这个方法就返回它。否则，它将抛出`NoSuchElementException`。

我们来看一个例子:

```java
@Test
public void whenCheckingFirstElement_shouldReturnFirstElement() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");

    assertEquals("First", treeSet.first());
}
```

### last()

与上面的例子类似，如果集合不是空的，这个方法将返回最后一个元素:

```java
@Test
public void whenCheckingLastElement_shouldReturnLastElement() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Last");

    assertEquals("Last", treeSet.last());
}
```

### subSet()

这个方法将返回从`fromElement`到`toElement`的元素。注意`fromElement`是包含性的，`toElement`是排他性的:

```java
@Test
public void whenUsingSubSet_shouldReturnSubSetElements() {
    SortedSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> expectedSet = new TreeSet<>();
    expectedSet.add(2);
    expectedSet.add(3);
    expectedSet.add(4);
    expectedSet.add(5);

    Set<Integer> subSet = treeSet.subSet(2, 6);
  
    assertEquals(expectedSet, subSet);
}
```

### headSet()

该方法将返回小于指定元素的`TreeSet`元素:

```java
@Test
public void whenUsingHeadSet_shouldReturnHeadSetElements() {
    SortedSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> subSet = treeSet.headSet(6);
  
    assertEquals(subSet, treeSet.subSet(1, 6));
}
```

### tailSet()

该方法将返回集合中大于或等于指定元素的元素:

```java
@Test
public void whenUsingTailSet_shouldReturnTailSetElements() {
    NavigableSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> subSet = treeSet.tailSet(3);
  
    assertEquals(subSet, treeSet.subSet(3, true, 6, true));
}
```

## 关于null

在Java 7之前，可以向空树集添加空元素。

然而，这被认为是一个错误。因此，`TreeSet`不再支持添加`null`。

当我们向树集添加元素时，元素将根据它们的自然顺序或由比较器指定的顺序进行排序。因此，当与现有元素比较时，添加`null`将导致`NullPointerException`，因为`null`不能与任何值进行比较:

```java
@Test(expected = NullPointerException.class)
public void whenAddingNullToNonEmptyTreeSet_shouldThrowException() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add(null);
}
```

插入`TreeSet`的元素必须实现`Comparable`接口，或者至少被指定的比较器接受。所有这些元素必须是相互可比的，即`e1. compareto (e2)`或`comparator.compare(e1, e2)`不能抛出`ClassCastException`。

我们来看一个例子:

```java
class Element {
    private Integer id;

    // Other methods...
}

Comparator<Element> comparator = (ele1, ele2) -> {
    return ele1.getId().compareTo(ele2.getId());
};

@Test
public void whenUsingComparator_shouldSortAndInsertElements() {
    Set<Element> treeSet = new TreeSet<>(comparator);
    Element ele1 = new Element();
    ele1.setId(100);
    Element ele2 = new Element();
    ele2.setId(200);

    treeSet.add(ele1);
    treeSet.add(ele2);

    System.out.println(treeSet);
}
```

## 性能

与`HashSet`相比，`TreeSet`的性能较低。像添加、删除和搜索这样的操作需要`O(log n)`的时间，而像按顺序打印n个元素这样的操作需要`O(n)`的时间。

如果我们希望保持元素的排序，那么`TreeSet`应该是我们的主要选择，因为`TreeSet`可以按升序或降序访问和遍历，而且升序操作和视图的性能可能比降序操作和视图的性能更快。

局部性原则——是一个术语，指的是根据内存访问模式频繁访问相同的值或相关的存储位置。

* 相似的数据经常被具有相似频率的应用程序访问
* 如果给定一个顺序，两个条目相邻，则`TreeSet`将它们放在数据结构中彼此相邻的位置，因此在内存中也是如此

`TreeSet`作为一个数据结构与更大的位置,因此,根据位置的原则,得出我们应当优先`TreeSet`如果我们缺少内存如果我们想访问元素相对接近对方根据他们的自然排序。

如果需要从硬盘读取数据(其延迟比从缓存或内存读取的数据更大)，那么最好选择`TreeSet`，因为它具有更大的局部性。

## 总结

在本文中，我们将重点了解如何在`Java`中使用标准的`TreeSet`实现。我们看到了它的目的和它在可用性方面的效率，因为它能够避免重复和对元素排序。

