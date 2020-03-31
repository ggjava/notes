## 概述

在本教程中，我们将研究`java.util.Arrays`，一个实用工具类，我们可以使用`Arrays`创建、比较、排序、搜索、流和转换数组。

## 创建

我们来看看创建数组的一些方法:`copyOf`、`copyOfRange`和`fill`。

### copyOf copyOfRange

要使用`copyOfRange`，我们需要原始数组和要复制的起始索引(包括)和结束索引(排除):

```java
String[] intro = new String[] { "once", "upon", "a", "time" };
String[] abridgement = Arrays.copyOfRange(storyIntro, 0, 3);

assertArrayEquals(new String[] { "once", "upon", "a" }, abridgement);
assertFalse(Arrays.equals(intro, abridgement));
```

使用`copyof`，我们将使用`intro`和一个目标数组大小，然后我们将得到一个该长度的新数组：

```java
String[] revised = Arrays.copyOf(intro, 3);
String[] expanded = Arrays.copyOf(intro, 5);

assertArrayEquals(Arrays.copyOfRange(intro, 0, 3), revised);
assertNull(expanded[4]);
```

> 请注意，如果目标大小大于原始大小，`copyof`将用空填充数组。

### fill

我们可以创建一个固定长度的数组，当我们需要一个所有元素都相同的数组时，`fill`非常有用：

```java
String[] stutter = new String[3];
Arrays.fill(stutter, "once");

assertTrue(Stream.of(stutter)
  .allMatch(el -> "once".equals(el));
```

> 通过`setAll`可以创建元素不同的数组。

注意，我们需要预先自己实例化数组，而不是`String[] filled = Arrays.fill(“once”, 3)`。

## 比较

现在让我们切换到比较数组的方法。

### equals/deepEquals

我们可以使用`equals`对数组的大小和内容进行简单的比较。如果我们添加一个null作为元素之一，将会失败:

```java
assertTrue(
  Arrays.equals(new String[] { "once", "upon", "a", "time" }, intro));
assertFalse(
  Arrays.equals(new String[] { "once", "upon", "a", null }, intro));
```

当我们嵌套或多维数组时，我们不仅可以使用`deepEquals`检查顶层元素，还可以递归执行检查:

```java
Object[] story = new Object[]
  { intro, new String[] { "chapter one", "chapter two" }, end };
Object[] copy = new Object[]
  { intro, new String[] { "chapter one", "chapter two" }, end };

assertTrue(Arrays.deepEquals(story, copy));
assertFalse(Arrays.equals(story, copy));
```

注意`deepEquals`是通过而`equals`失败的。

这是因为`deepEquals`每次遇到数组时最终都会调用自己，而equals只会比较子数组的引用。

### hashCode/deepHashCode

`hashCode`的实现将为我们提供Java对象推荐的`equals/hashCode`契约的另一部分。我们使用`hashCode`根据数组的内容计算:

```java
Object[] looping = new Object[]{ intro, intro };
int hashBefore = Arrays.hashCode(looping);
int deepHashBefore = Arrays.deepHashCode(looping);
```

现在，我们将原始数组的一个元素设置为null，并重新计算散列值:

```java
intro[3] = null;
int hashAfter = Arrays.hashCode(looping);
```

或者，`deepHashCode`检查嵌套数组以匹配元素和内容的数量。如果我们用`deepHashCode`重新计算:

```java
int deepHashAfter = Arrays.deepHashCode(looping);
```

现在，我们可以看到这两种方法的区别:

```java
assertEquals(hashAfter, hashBefore);
assertNotEquals(deepHashAfter, deepHashBefore);
```

`deepHashCode`与`HashMap`和`HashSet`等处理数据结构时使用算法一样。

## 排序

如果我们的元素或者它们实现了`Comparable`，我们可以使用sort来执行排序:

```java
String[] sorted = Arrays.copyOf(intro, 4);
Arrays.sort(sorted);

assertArrayEquals(
  new String[]{ "a", "once", "time", "upon" }, 
  sorted);
```

> 注意，排序会改变原始引用，这就是我们在这里执行副本的原因。

`sort`将对不同的数组元素类型使用不同的算法。基本类型使用双主快速排序，对象类型使用`Timsort`。对于随机排序的数组，它们都有O(n log(n))的平均情况。

从Java8开始，`ParallelSort`可用于并行归并排序。它提供了一种使用多个数组的并发排序方法。

## 搜索

在一个无序数组中搜索是线性的，但是如果我们有一个有序数组，我们可以用binarySearch, 复杂度O(log n):

```java
int exact = Arrays.binarySearch(sorted, "time");
int caseInsensitive = Arrays.binarySearch(sorted, "TiMe", String::compareToIgnoreCase);

assertEquals("time", sorted[exact]);
assertEquals(2, exact);
assertEquals(exact, caseInsensitive);
```

如果我们不提供比较器作为第三个参数，那么`binarySearch`就依赖于我们的元素类型是`Comparable`类型。

> 请注意，如果我们的数组没有首先排序，那么`binarySearch`将不能像我们预期的那样工作!

## 流

正如我们前面看到的，在Java 8中更新了`Arrays`，以包含使用流API的方法，如`parallelSort`、`Stream`和`setAll`。

```java
Assert.assertEquals(Arrays.stream(intro).count(), 4);

exception.expect(ArrayIndexOutOfBoundsException.class);
Arrays.stream(intro, 2, 1).count();
```

我们可以为流提供包含的和排他的索引，但是，如果索引出现顺序错误、负数或超出范围，则会抛出`ArrayIndexOutOfBoundsException`异常。

## 转换

最后，`toString`、`asList`和`setAll`给出了转换数组的几种不同方法。

### toString/deepToString

获得原始数组可读版本的一个很好的方法是使用`toString`：

```java
assertEquals("[once, upon, a, time]", Arrays.toString(storyIntro));
```

同样，我们必须使用深度版本打印嵌套数组的内容:

```java
assertEquals(
  "[[once, upon, a, time], [chapter one, chapter two], [the, end]]",
  Arrays.deepToString(story));
```

### asList

我们使用的所有数组方法中最方便的是`asList`。一个简单的方法把数组变成列表:

```java
List<String> rets = Arrays.asList(storyIntro);

assertTrue(rets.contains("upon"));
assertTrue(rets.contains("time"));
assertEquals(rets.size(), 4);
```

> 返回的列表将是固定长度，因此我们将无法添加或删除元素。

### setAll

使用`setAll`，我们可以使用函数接口设置数组的所有元素。生成器实现将位置索引作为参数:

```java
String[] longAgo = new String[4];
Arrays.setAll(longAgo, i -> this.getWord(i));
assertArrayEquals(longAgo, new String[]{"a", "long", "time", "ago"});
```

> 当然，异常处理是使用`lambda`时比较危险的部分之一。记住，如果`lambda`抛出异常，那么Java不会定义数组的最终状态。

## 总结

在本文中，我们学习了如何使用`java.util.Arrays`创建、搜索、排序和转换数组。
