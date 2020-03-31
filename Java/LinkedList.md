## 概述

`LinkedList`是`list`和`Deque`接口的双链列表实现。它实现所有可选的列表操作，并允许所有元素(包括`null`)。

## 特点

下面你可以找到`LinkedList`最重要的属性:

* 索引到列表中的操作将从头到尾遍历列表，以更接近指定索引的操作为准
* 它不是同步的
* 它的迭代器和`ListIterator`迭代器是快速失败的(这意味着在迭代器创建之后，如果列表被修改，将抛出`ConcurrentModificationException`)
* 每个元素都是一个节点，它保存对下一个和上一个元素的引用
* 它保持插入顺序

尽管`LinkedList`未同步，但我们可以通过调用`Collections.SynchronizedList`方法检索他的同步版本，如：

```java
List list = Collections.synchronizedList(new LinkedList(...));
```

## 与ArrayList对比

尽管它们都实现了`List`接口，但它们具有不同的含义——这肯定会影响使用哪个接口。

### 结构

`ArrayList`是由数组支持的基于索引的数据结构。它提供对其性能等于`O(1)`的元素的随机访问。

另一方面，`LinkedList`将其数据存储为元素列表，每个元素都链接到它的上一个和下一个元素。在本例中，对元素的搜索操作的执行时间等于`O(n)`。

### 操作

在`LinkedList`中，元素的插入、添加和删除操作速度更快，因为当元素被添加到集合中的任意位置时，不需要调整数组的大小或更新索引，只需要更改周围元素中的引用。

### 内存占用

`LinkedList`比`ArrayList`消耗更多的内存，因为`LinkedList`中的每个节点都存储两个引用，一个用于前一个元素，一个用于下一个元素，而`ArrayList`只保存数据和索引。

## 使用

下面是一些代码示例，展示了如何使用`LinkedList`:

### 创建

```java
LinkedList<Object> linkedList = new LinkedList<>();
```

### 添加元素

`LinkedList`实现了`List`和`Deque`接口，除了标准的`add()`和`addAll()`方法之外，您还可以找到`addFirst()`和`addLast()`，它们分别在开头和结尾添加了一个元素。

### 删除元素

与元素添加类似，此列表实现提供了`removeFirst()`和`removeLast()`。

此外，还有方便的方法`removeFirstOccurence()`和`removelastoccurs()`，它们返回布尔值(如果集合包含指定的元素，则为`true`)。

### 搜索操作

`Deque`接口提供了类似队列的行为(实际上`Deque`扩展了队列接口):

```java
linkedList.poll();
linkedList.pop();
```

这些方法检索第一个元素并将其从列表中删除。

`poll()`和`pop()`的区别在于，`pop`将抛出`NoSuchElementException()`到空列表，而`poll`返回`null`。api `pollFirst()`和`pollLast()`也可用。

```java
linkedList.push(Object o);
```

它将元素插入集合的头部。

`LinkedList`还有很多其他的方法，其中大部分对于已经使用过`list`的用户应该很熟悉。`Deque`提供的其他方法可能是“标准”方法的一个方便的替代方法。

## 总结

`ArrayList`通常是默认的列表实现。

然而，在某些特定的用例中，使用`LinkedList`会更合适，比如对固定插入/删除时间(例如频繁插入/删除/更新)、固定访问时间和有效内存使用。
