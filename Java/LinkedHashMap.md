## 概述

在本文中，我们将探讨`LinkedHashMap`类的内部实现。`LinkedHashMap`是`Map`接口的一种常见实现。

这个特定的实现是`HashMap`的子类，因此共享`HashMap`实现的核心构建块。因此，强烈建议您在阅读本文之前复习一下这方面的知识。

## LinkedHashMap vs HashMap

`LinkedHashMap`类在大多数方面与`HashMap`非常相似。但是，`LinkedHashMap`同时基于哈希表和链表，以增强哈希映射的功能。

它维护一个双链表，除了默认大小为16的底层数组之外，还运行所有条目。

为了维护元素的顺序，`LinkedHashMap`通过修改`HashMap`的`Map.Entry `添加指向下一个和前一个条目的指针:

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

注意，`Entry`类只添加了两个指针;在此之前和之后，它可以将自己挂接到链表上。除此之外，它还使用`HashMap`的入口类实现。

最后，请记住这个链表定义了迭代的顺序，默认情况下是元素插入的顺序(插入顺序)。

## 插入顺序

让我们来看看一个`LinkedHashMap`实例，它根据条目如何插入到映射中来对其排序。它还保证在实例的整个生命周期内维持这一秩序:

```java
@Test
public void givenLinkedHashMap_whenGetsOrderedKeyset_thenCorrect() {
    LinkedHashMap<Integer, String> map = new LinkedHashMap<>();
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);

    Set<Integer> keys = map.keySet();
    Integer[] arr = keys.toArray(new Integer[0]);

    for (int i = 0; i < arr.length; i++) {
        assertEquals(new Integer(i + 1), arr[i]);
    }
}
```

在这里，我们只是对`LinkedHashMap`中的元素排序进行了初步的、非结论性的测试。

我们可以保证这个测试将始终通过，因为插入顺序将始终保持不变。但`HashMap`则不行。

如果客户需要在调用API之前以相同的方式对返回的map进行排序，那么可以使用`LinkedHashMap`。

如果一个键被重新插入到map中，则插入顺序不受影响。

## 若不想使用默认顺序

`LinkedHashMap`提供了一个特殊的构造函数，使我们能够在自定义负载因子(LF)和初始容量之间指定一种称为访问顺序的不同排序机制/策略:

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>(16, .75f, true);
```

第一个参数是初始容量，然后是负载系数，最后一个参数是排序模式。所以，通过传入`true`，我们得到了访问顺序，而默认的是插入顺序。

此机制确保元素的迭代顺序是上次访问元素的顺序，从最近最少访问到最近最多访问。

因此，构建一个最近最少使用的(LRU)缓存是非常简单和实用:

```java
@Test
public void givenLinkedHashMap_whenAccessOrderWorks_thenCorrect() {
    LinkedHashMap<Integer, String> map 
      = new LinkedHashMap<>(16, .75f, true);
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);

    Set<Integer> keys = map.keySet();
    assertEquals("[1, 2, 3, 4, 5]", keys.toString());
  
    map.get(4);
    assertEquals("[1, 2, 3, 5, 4]", keys.toString());
  
    map.get(1);
    assertEquals("[2, 3, 5, 4, 1]", keys.toString());
  
    map.get(3);
    assertEquals("[2, 5, 4, 1, 3]", keys.toString());
}
```

请注意，当我们在map上执行访问操作时，如何转换map中元素的顺序， map上的任何访问操作，则被访问的元素将最后出现。

迭代map不会影响顺序；只有访问操作才会影响顺序。

`LinkedHashMap`还提供了一种机制，用于维护固定数量的元素，并在需要添加新元素时不断删除最旧的元素。

可以重写`removeeldestentry`方法以强制此策略自动删除过时的元素。

为了在实践中看到这一点，让我们创建自己的`LinkedHashMap`类，通过扩展`LinkedHashMap`强制删除过时的元素：

```java
public class MyLinkedHashMap<K, V> extends LinkedHashMap<K, V> {

    private static final int MAX_ENTRIES = 5;

    public MyLinkedHashMap(
      int initialCapacity, float loadFactor, boolean accessOrder) {
        super(initialCapacity, loadFactor, accessOrder);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

}
```

上面的覆盖将允许map增长到最大的5个条目。当大小超过这个值时，每个新元素将被插入，代价是丢失map中最老的元素，即最后访问时间先于所有其他元素的元素:

```java
@Test
public void givenLinkedHashMap_whenRemovesEldestEntry_thenCorrect() {
    LinkedHashMap<Integer, String> map
      = new MyLinkedHashMap<>(16, .75f, true);
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);
    Set<Integer> keys = map.keySet();
    assertEquals("[1, 2, 3, 4, 5]", keys.toString());
  
    map.put(6, null);
    assertEquals("[2, 3, 4, 5, 6]", keys.toString());
  
    map.put(7, null);
    assertEquals("[3, 4, 5, 6, 7]", keys.toString());
  
    map.put(8, null);
    assertEquals("[4, 5, 6, 7, 8]", keys.toString());
}
```

## 性能

与`HashMap`一样，只要哈希函数良好，`LinkedHashMap`就可以在恒定的时间内执行添加、删除和包含基本操作。它还接受空键和空值。

但是，`LinkedHashMap`的这种常量时间性能可能比`HashMap`的常量时间性能稍差，这是由于维护一个双链表增加了开销。

对`LinkedHashMap`集合视图的迭代也花费与`HashMap`类似的线性时间`O(n)`。另一方面，`LinkedHashMap`在迭代期间的线性时间性能优于`HashMap`的线性时间。

这是因为，对于`LinkedHashMap`, `O(n)`中的`n`只是映射中条目的数量，而与容量无关。而对于`HashMap`, `n`是容量，大小相加为`O(大小+容量)`。

负载因子和初始容量被精确地定义为`HashMap`。但是，请注意，对于初始容量选择过高的惩罚对于`LinkedHashMap`来说没有`HashMap`那么严重，因为该类的迭代时间不受容量的影响。

## 并发

与`HashMap`一样`LinkedHashMap`也是不同步的。因此，如果您要从多个线程访问它，并且其中至少有一个线程可能会从结构上更改它，那么它必须是外部同步的。

最好在创建时这样做:

```java
Map m = Collections.synchronizedMap(new LinkedHashMap());
```

与`HashMap`的区别在`LinkedHashMap`仅仅调用get就会导致结构修改。除此之外，还有put和remove之类的操作。

## 总结

在本文中，我们探讨了Java `LinkedHashMap`类作为`Map`接口的最重要实现之一的用法。我们还探讨了它与`HashMap`的区别，`HashMap`是它的超类。

希望在阅读完这篇文章后，您可以做出更明智和有效的决定，在您的用例中使用哪种map实现。
