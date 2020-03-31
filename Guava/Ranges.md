
## 示例

```java
List<Double> scores;
Iterable<Double> belowMedianScores = Iterables.filter(scores, Range.lessThan(median));
...
Range<Integer> validGrades = Range.closed(1, 12);
for(int grade : ContiguousSet.create(validGrades, DiscreteDomain.integers())) {
  ...
}
```

## 简介

区间，有时也称为范围，是特定域中的凸性（非正式说法为连续的或不中断的）部分。在形式上，凸性表示对 `a<=b<=c`, `range.contains(a)`且 `range.contains(c)`意味着 `range.contains(b)`。

区间可以延伸至无限——例如，范围”x>3″包括任意大于3的值——也可以被限制为有限，如`”2<=x<5″`。Guava 用更紧凑的方法表示范围，有数学背景的程序员对此是耳熟能详的：

* (a..b) = {x | a < x < b}
* [a..b] = {x | a <= x <= b}
* [a..b) = {x | a <= x < b}
* (a..b] = {x | a < x <= b}
* (a..+∞) = {x | x > a}
* [a..+∞) = {x | x >= a}
* (-∞..b) = {x | x < b}
* (-∞..b] = {x | x <= b}
* (-∞..+∞) = all values

上面的 a、b 称为端点 。为了提高一致性，Guava 中的 Range 要求上端点不能小于下端点。上下端点有可能是相等的，但要求区间是闭区间或半开半闭区间（至少有一个端点是包含在区间中的）：

* [a..a]：单元素区间
* [a..a); (a..a]：空区间，但它们是有效的
* (a..a)：无效区间

Guava用[Range<C>](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html)表示区间。所有区间实现都是不可变类型。

## 构建区间

区间实例可以由`Range`类的静态方法获取：

| Range type | Method |
| - | - |
| (a..b) | open(C, C) |
| [a..b] | closed(C, C) |
| [a..b) | closedOpen(C, C) |
| (a..b] | openClosed(C, C) |
| (a..+∞) | greaterThan(C) |
| [a..+∞) | atLeast(C) |
| (-∞..b) | lessThan(C) |
| (-∞..b] | atMost(C) |
| (-∞..+∞) | all() |

```java
Range.closed("left", "right"); // all strings lexographically between "left" and "right" inclusive
Range.lessThan(4.0); // double values strictly less than 4
```

此外，也可以明确地指定边界类型来构造区间：

| Range type | Method |
| - | - |
| Bounded on both ends | range(C, BoundType, C, BoundType) |
| Unbounded on top ((a..+∞) or [a..+∞)) | downTo(C, BoundType) |
| Unbounded on bottom ((-∞..b) or (-∞..b]) | upTo(C, BoundType) |

这里的[BoundType](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BoundType.html)是一个枚举类型，包含`CLOSED`和`OPEN`两个值。

```java
Range.downTo(4, boundType); // allows you to decide whether or not you want to include 4
Range.range(1, CLOSED, 4, OPEN); // another way of writing Range.closedOpen(1, 4)
```

### 区间运算

`Range`的基本运算是它的[contains(C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#contains-C-)方法，和你期望的一样，它用来区间判断是否包含某个值。此外，`Range`实例也可以当作`Predicate`，并且在函数式编程中使用。任何`Range` 实例也都支持`containsAll(Iterable<? extends C>)`方法：

```java
Range.closed(1, 3).contains(2); // returns true
Range.closed(1, 3).contains(4); // returns false
Range.lessThan(5).contains(5); // returns false
Range.closed(1, 4).containsAll(Ints.asList(1, 2, 3)); // returns true
```

### 查询运算

`Range`类提供了以下方法来查看区间的端点：

* [hasLowerBound()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#hasLowerBound--) 和 [hasUpperBound()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#hasUpperBound--)：判断区间是否有特定边界，或是无限的；
* [lowerBoundType()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#lowerBoundType--) 和 [upperBoundType()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#upperBoundType--)：返回区间边界类型，`CLOSED`或`OPEN`；如果区间没有对应的边界，抛出 `IllegalStateException`；
* [lowerEndpoint()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#lowerEndpoint--) 和 [upperEndpoint()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#upperEndpoint--)：返回区间的端点值；如果区间没有对应的边界，抛出`IllegalStateException`
* [isEmpty()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#isEmpty--)：判断是否为空区间。

```java
Range.closedOpen(4, 4).isEmpty(); // returns true
Range.openClosed(4, 4).isEmpty(); // returns true
Range.closed(4, 4).isEmpty(); // returns false
Range.open(4, 4).isEmpty(); // Range.open throws IllegalArgumentException

Range.closed(3, 10).lowerEndpoint(); // returns 3
Range.open(3, 10).lowerEndpoint(); // returns 3
Range.closed(3, 10).lowerBoundType(); // returns CLOSED
Range.open(3, 10).upperBoundType(); // returns OPEN
```

### 区间运算

#### 包含[enclose]

区间之间的最基本关系就是包含[encloses(Range)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#encloses-com.google.common.collect.Range-)：如果内区间的边界没有超出外区间的边界，则外区间包含内区间。包含判断的结果完全取决于区间端点的比较！

* [3..6] 包含[4..5] ；
* (3..6) 包含(3..6) ；
* [3..6] 包含[4..4)，虽然后者是空区间；
* (3..6]不 包含[3..6] ；
* [4..5]不 包含(3..6)，虽然前者包含了后者的所有值，离散域[discrete domains]可以解决这个问题（见 8.5节）；
* [3..6]不 包含(1..1]，虽然前者包含了后者的所有值。

包含是一种偏序关系[partial ordering](https://github.com/google/guava/wiki/GuavaTermsExplained#partial-ordering)。基于包含关系的概念，`Range`还提供了以下运算方法。

#### 相连[isConnected]

[Range.isConnected(Range)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#isConnected-com.google.common.collect.Range-)判断区间是否是相连的。具体来说，`isConnected`测试是否有区间同时包含于这两个区间，这等同于数学上的定义”两个区间的并集是连续集合的形式”（空区间的特殊情况除外）。

相连是一种自反的[reflexive](https://github.com/google/guava/wiki/GuavaTermsExplained#reflexive)、对称的[symmetric](https://github.com/google/guava/wiki/GuavaTermsExplained#symmetric)关系。

```java
Range.closed(3, 5).isConnected(Range.open(5, 10)); // returns true
Range.closed(0, 9).isConnected(Range.closed(3, 4)); // returns true
Range.closed(0, 5).isConnected(Range.closed(3, 9)); // returns true
Range.open(3, 5).isConnected(Range.open(5, 10)); // returns false
Range.closed(1, 5).isConnected(Range.closed(6, 10)); // returns false
```

#### 交集[intersection]

[Range.intersection(Range)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Range.html#intersection(com.google.common.collect.Range))返回两个区间的交集：既包含于第一个区间，又包含于另一个区间的最大区间。当且仅当两个区间是相连的，它们才有交集。如果两个区间没有交集，该方法将抛出`IllegalArgumentException`。

交集是可互换的[commutative](https://github.com/google/guava/wiki/GuavaTermsExplained#commutative)、关联的[associative](https://github.com/google/guava/wiki/GuavaTermsExplained#associative)运算[operation][binary-operation]。

```java
Range.closed(3, 5).intersection(Range.open(5, 10)); // returns (5, 5]
Range.closed(0, 9).intersection(Range.closed(3, 4)); // returns [3, 4]
Range.closed(0, 5).intersection(Range.closed(3, 9)); // returns [3, 5]
Range.open(3, 5).intersection(Range.open(5, 10)); // throws IAE
Range.closed(1, 5).intersection(Range.closed(6, 10)); // throws IAE
```

#### 跨区间[span]

[Range.span(Range)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#span-com.google.common.collect.Range-)返回”同时包括两个区间的最小区间”，如果两个区间相连，那就是它们的并集。

`span`是可互换的[commutativ](https://github.com/google/guava/wiki/GuavaTermsExplained#commutative)、关联的[associative](https://github.com/google/guava/wiki/GuavaTermsExplained#associative)、闭合的[closed](https://github.com/google/guava/wiki/GuavaTermsExplained#closed)运算[operation][binary-operation]。

```java
Range.closed(3, 5).span(Range.open(5, 10)); // returns [3, 10)
Range.closed(0, 9).span(Range.closed(3, 4)); // returns [0, 9]
Range.closed(0, 5).span(Range.closed(3, 9)); // returns [0, 9]
Range.open(3, 5).span(Range.open(5, 10)); // returns (3, 10)
Range.closed(1, 5).span(Range.closed(6, 10)); // returns [1, 10]
```

## 离散域

部分（但不是全部）可比较类型是离散的，即区间的上下边界都是可枚举的。

在`Guava`中，用[DiscreteDomain<C\>](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/DiscreteDomain.html)实现类型`C`的离散形式操作。一个离散域总是代表某种类型值的全集；它不能代表类似”素数”、”长度为5的字符串”或”午夜的时间戳”这样的局部域。

`DiscreteDomain`提供的离散域实例包括：

| Type | DiscreteDomain |
| - | - |
| Integer | integers() |
| Long | longs() |

一旦获取了 DiscreteDomain 实例，你就可以使用下面的 Range 运算方法：

* [ContiguousSet.create(range, domain)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ContiguousSet.html#create-com.google.common.collect.Range-com.google.common.collect.DiscreteDomain-)：用 `ImmutableSortedSet`形式表示`Range`中符合离散域定义的元素，并增加一些额外操作。
* [canonical(domain)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#canonical-com.google.common.collect.DiscreteDomain-)：把离散域转为区间的”规范形式”。如果`ContiguousSet.create(a, domain).equals(ContiguousSet.create(b, domain))`并且`!a.isEmpty()`，则有`a.canonical(domain).equals(b.canonical(domain))`。（这并不意味着`a.equals(b)`）

```java
ImmutableSortedSet<Integer> set = ContiguousSet.create(Range.open(1, 5), DiscreteDomain.integers());
// set contains [2, 3, 4]

ContiguousSet.create(Range.greaterThan(0), DiscreteDomain.integers());
// set contains [1, 2, ..., Integer.MAX_VALUE]

```

> 注意，`ContiguousSet.create`并没有真的构造了整个集合，而是返回了`set`形式的区间视图。

### 你自己的离散域

你可以创建自己的离散域，但必须记住`DiscreteDomain`契约的几个重要方面。

* 一个离散域总是代表某种类型值的全集；它不能代表类似”素数”或”长度为5的字符串”这样的局部域。所以举例来说，你无法构造一个`DiscreteDomain`以表示精确到秒的`JODA DateTime`日期集合：因为那将无法包含`JODA DateTime`的所有值。
* DiscreteDomain 可能是无限的——比如`BigInteger DiscreteDomain`。这种情况下，你应当用`minValue()`和`maxValue()`的默认实现，它们会抛出`NoSuchElementException`。但`Guava`禁止把无限区间传入`ContiguousSet.create`。

## 如果我需要一个Comparator呢？

我们想要在`Range`的可用性与`API`复杂性之间找到特定的平衡，这部分导致了我们没有提供基于`Comparator`的接口：我们不需要操心区间是怎样基于不同`Comparator`互动的；所有`API`签名都是简单明确的；这样更好。

另一方面，如果你需要任意`Comparator`，可以按下列其中一项来做：

使用通用的`Predicate`接口，而不是`Range`类。（`Range`现了`Predicate`接口，因此可以用`Predicates.compose(range, function)`获取`Predicate`实例）

使用包装类以定义期望的排序。
