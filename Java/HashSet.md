## 概述

在本文中，我们将深入讨论`HashSet`。它是最流行的`Set`实现之一，也是Java集合框架的一个组成部分。

让我们回顾一下这个实现最重要的方面:

* 它存储的元素是唯一的并允许为空
* 它由`HashMap`支持
* 元素是无序的
* 不是线程安全的

> 注意，这个内部`HashMap`是在创建`HashSet`的实例时初始化的:

```java
public HashSet() {
    map = new HashMap<>();
}
```

如果您想更深入地了解HashMap的工作方式，可以在这里阅读关于它的文章。

## API

在本节中，我们将回顾最常用的方法，并查看一些简单的示例。

### add()

`add()`方法可用于向集合中添加元素。该方法契约声明，只有在集合中没有元素时才会添加元素。如果添加了元素，该方法将返回`true`，否则返回 `false`。

我们可以向`HashSet`添加一个元素，比如:

```java
@Test
public void whenAddingElement_shouldAddElement() {
    Set<String> hashset = new HashSet<>();
  
    assertTrue(hashset.add("String Added"));
}
```

从实现的角度来看，`add`方法非常重要。实现细节说明了`HashSet`如何在内部工作并利用`HashMap`的`put`方法:

```java
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

`map`变量是对内部支持`HashMap`的引用:

```java
private transient HashMap<E, Object> map;
```

最好先熟悉`hashcode`，以便详细了解元素在基于`hashcode`的数据结构中是如何组织的。

### contains()

`contains`方法的目的是检查给定`HashSet`中是否存在元素。如果找到元素，则返回`true`，否则返回`false`。

我们可以检查`HashSet`中的一个元素:

```java
@Test
public void whenCheckingForElement_shouldSearchForElement() {
    Set<String> hashsetContains = new HashSet<>();
    hashsetContains.add("String Added");
  
    assertTrue(hashsetContains.contains("String Added"));
}
```

无论何时将对象传递给此方法，都会计算散列值。然后，解析并遍历相应的`bucket`位置。

### remove()

如果指定的元素存在，该方法将从集合中删除它。如果集合包含指定的元素，则此方法返回true。

让我们来看一个实际的例子:

```java
@Test
public void whenRemovingElement_shouldRemoveElement() {
    Set<String> removeFromHashSet = new HashSet<>();
    removeFromHashSet.add("String Added");
  
    assertTrue(removeFromHashSet.remove("String Added"));
}
```

### clear()

当我们打算从一个集合中删除所有项时，我们使用这个方法。

```java
@Test
public void whenClearingHashSet_shouldClearHashSet() {
    Set<String> clearHashSet = new HashSet<>();
    clearHashSet.add("String Added");
    clearHashSet.clear();

    assertTrue(clearHashSet.isEmpty());
}
```

### size()

这是API中的基本方法之一。它被大量使用，因为它有助于确定`HashSet`中元素的数量。底层实现只是将计算委托给`HashMap`的`size()`方法。

让我们看看实际情况:

```java
@Test
public void whenCheckingTheSizeOfHashSet_shouldReturnThesize() {
    Set<String> hashSetSize = new HashSet<>();
    hashSetSize.add("String Added");

    assertEquals(1, hashSetSize.size());
}
```

### isEmpty()

我们可以使用此方法来确定`HashSet`的给定实例是否为空。如果集合不包含元素，则该方法返回`true`:

```java
@Test
public void whenCheckingForEmptyHashSet_shouldCheckForEmpty() {
    Set<String> emptyHashSet = new HashSet<>();

    assertTrue(emptyHashSet.isEmpty());
}
```

### iterator()

该方法对集合中的元素返回一个迭代器。元素没有特定的顺序被访问，迭代器是快速失效的。

我们可以在这里观察到随机迭代的顺序:

```java
@Test
public void whenIteratingHashSet_shouldIterateHashSet() {
    Set<String> hashset = new HashSet<>();
    hashset.add("First");
    hashset.add("Second");
    hashset.add("Third");
    Iterator<String> itr = hashset.iterator();
    while(itr.hasNext()){
        System.out.println(itr.next());
    }
}
```

如果在以任何方式(除了通过迭代器自己的`remove`方法之外)创建迭代器之后的任何时候修改集合，迭代器将抛出`ConcurrentModificationException`。

让我们看看实际情况:

```java
@Test(expected = ConcurrentModificationException.class)
public void whenModifyingHashSetWhileIterating_shouldThrowException() {
  
    Set<String> hashset = new HashSet<>();
    hashset.add("First");
    hashset.add("Second");
    hashset.add("Third");
    Iterator<String> itr = hashset.iterator();
    while (itr.hasNext()) {
        itr.next();
        hashset.remove("Second");
    }
}
```
或者，如果我们使用迭代器的删除方法，那么我们就不会遇到异常:

```java
@Test
public void whenRemovingElementUsingIterator_shouldRemoveElement() {
  
    Set<String> hashset = new HashSet<>();
    hashset.add("First");
    hashset.add("Second");
    hashset.add("Third");
    Iterator<String> itr = hashset.iterator();
    while (itr.hasNext()) {
        String element = itr.next();
        if (element.equals("Second"))
            itr.remove();
    }
  
    assertEquals(2, hashset.size());
}
```

迭代器的快速故障行为无法得到保证，因为在存在非同步并发修改的情况下，不可能做出任何严格的保证。

故障快速迭代器在最大努力的基础上抛出`ConcurrentModificationException`。因此，编写依赖于此异常的正确性的程序是错误的。

## HashSet如何保持唯一性?

当我们将一个对象放入`HashSet`时，它使用对象的`hashcode`值来确定一个元素是否已经不在集合中。

每个哈希码值对应于一个可以包含各种元素的桶位置，对于这些元素，计算出的哈希值是相同的。但是具有相同`hashCode`的两个对象可能不相等。

因此，将使用`equals()`方法比较同一`bucket`中的对象。

## HashSet的性能

`HashSet`的性能主要受两个参数的影响:初始容量和负载因子。

向一个集合添加一个元素的预期时间复杂度是`O(1)`，在最坏的情况下(只有一个桶)，它可以下降到`O(n)`——因此，维护正确的`HashSet`容量是必要的。

重要提示: 自`JDK 8`以来，最坏情况下的时间复杂度是`O(log*n)`。

负载因子描述了最大的填充级别，超过这个级别，就需要调整集合的大小。

我们还可以创建一个`HashSet`与自定义值的初始容量和负载因素:

```java
Set<String> hashset = new HashSet<>();
Set<String> hashset = new HashSet<>(20);
Set<String> hashset = new HashSet<>(20, 0.5f);
```

在第一种情况下，使用默认值——初始容量为16，负载系数为0.75。在第二种情况下，我们覆盖默认容量，在第三种情况下，我们覆盖两者。

低初始容量降低了空间复杂度，但增加了重哈希的频率，这是一个昂贵的过程。

另一方面，高初始容量增加了迭代的成本和初始内存消耗。

作为一个经验法则:

* 高初始容量对于大量条目和很少或没有迭代的情况是很好的
* 较低的初始容量对于具有大量迭代的少量条目是有好处的

因此，在两者之间取得正确的平衡是非常重要的。通常，默认实现是经过优化的，工作得很好，如果我们觉得需要调优这些参数以适应需求，那么我们需要明智地进行调整。

## 总结

在本文中，我们概述了`HashSet`的实用程序、用途及其底层工作。考虑到它的持续时间性能和避免重复的能力，我们看到了它在可用性方面的效率。

我们研究了一些API的重要方法，它们如何帮助我们作为开发人员使用`HashSet`来发挥其潜力。
