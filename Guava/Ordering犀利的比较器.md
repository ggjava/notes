在本文中，我们将介绍Guava库中Ordering类。

Ordering类实现了Comparator接口，它可以用来为构建复杂的比较器，以完成集合排序的功能。

从实现上说，Ordering实例就是一个特殊的Comparator实例。Ordering把很多基于Comparator的静态方法（如Collections.max）包装为自己的实例方法（非静态方法），并且提供了链式调用方法，来定制和增强现有的比较器。

## 创建

## natural()

使用自然顺序排序

```java
List<Integer> list = Arrays.asList(1, 5, 3, 8, 2);
Collections.sort(list, Ordering.natural());
```

### usingToString()

通过对象字符串（toString()返回）表示形式的字典顺序来比较对象。

```java
List<String> listString = Lists.newArrayList("wang", "li", "chen");
Collections.sort(listString, Ordering.usingToString());
```

### from(Comparator)

通过比较器构建

```java
List<User> users = Lists.newArrayList(usera, userb, userc);
Ordering<User> orderUser = Ordering.from(new UserIdComparator());
Collections.sort(users, orderUser);
```

### 使用抽象类

```java
Ordering<String> byLengthOrdering = new Ordering<String>() {
  public int compare(String left, String right) {
    return Ints.compare(left.length(), right.length());
  }
};
```

## 链式调用

### reverse()

获取语义相反的排序器

```java
@Test
public void testOrderReverse() {
    List<Integer> list = Arrays.asList(1, 5, 3, 8, 2);
    Collections.sort(list, Ordering.natural().reverse());
    System.out.println("获取最大的元素" + Ordering.natural().max(list).toString());
    System.out.println("排序后" + list.toString());
}
```

### nullsFirst()

使用当前排序器，但额外把null值排到最前面。

```java
@Test(expected = NullPointerException.class)
public void testJDKOrderIssue() {
    List<Integer> list = Arrays.asList(1, 5, null, 3, 8, 2);
    System.out.println("排序前" + list.toString());
    Collections.sort(list); // 出现异常...
}
```

### nullsLast()

使用当前排序器，但额外把null值排到最后面。

### compound(Comparator)

合成另一个比较器，以处理当前排序器中的相等情况。

### lexicographical()

基于处理类型T的排序器，返回该类型的可迭代对象Iterable<T>的排序器。

### onResultOf(Function)

对集合中元素调用Function，再按返回值用当前排序器排序。  

例如，你需要下面这个类的排序器。

```java
class Foo {
    @Nullable String sortedBy;
    int notSortedBy;
}
```

考虑到排序器应该能处理sortedBy为null的情况，我们可以使用下面的链式调用来合成排序器：

```java
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(new Function<Foo, String>() {
  public String apply(Foo foo) {
    return foo.sortedBy;
  }
});
```

当阅读链式调用产生的排序器时，应该从后往前读。上面的例子中，排序器首先调用apply方法获取sortedBy值，并把sortedBy为null的元素都放到最前面，然后把剩下的元素按sortedBy进行自然排序。之所以要从后往前读，是因为每次链式调用都是用后面的方法包装了前面的排序器。

> 注：用compound方法包装排序器时，就不应遵循从后往前读的原则。为了避免理解上的混乱，请不要把compound写在一长串链式调用的中间，你可以另起一行，在链中最先或最后调用compound。

超过一定长度的链式调用，也可能会带来阅读和理解上的难度。我们建议按下面的代码这样，在一个链中最多使用三个方法。此外，你也可以把Function分离成中间对象，让链式调用更简洁紧凑。

```java
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(sortKeyFunction)
```

## 运用排序器

Guava的排序器实现有若干操纵集合或元素值的方法:

### greatestOf(Iterable iterable, int k)

获取可迭代对象中最大的k个元素。

```java
@Test
public void testGreaTestOf() {
    List<Integer> list = Arrays.asList(1, 5, 3, 8, 2);
    //获取可迭代对象中最大的k个元素。
    List<Integer> listMaxtOfK = Ordering.natural().greatestOf(list, 3);
    System.out.println("获取最大的k个元素：" + listMaxtOfK.toString());
    List<Integer> listMaxtOfMinik = Ordering.natural().reverse().greatestOf(list, 3);
    // listMaxtOfK.add(1); UnmodifiableCollection 返回的是不可变对象，不可以进行操作
    System.out.println("获取最大的Minik个元素：" + listMaxtOfMinik.toString());
}
```

### isOrdered(Iterable)

判断可迭代对象是否已按排序器排序：允许有排序值相等的元素。

```java
@Test
public void testOrderNatural() {
    List<Integer> list = Arrays.asList(1, 5, 3, 8, 2);
    Collections.sort(list);
    //是否为按照这样的顺序排好序的！自然的排序
    boolean order = Ordering.natural().isOrdered(list);
    System.out.println("排好序的：" + (order == true ? "是的" : "不是"));
}
```

### sortedCopy(Iterable)

判断可迭代对象是否已严格按排序器排序：不允许排序值相等的元素。

### min(E, E)

返回两个参数中最小的那个。如果相等，则返回第一个参数。

### min(E, E, E, E...)

返回多个参数中最小的那个。如果有超过一个参数都最小，则返回第一个最小的参数。

```java
@Test
public void testMax() {
    List<Integer> list = Arrays.asList(1, 5, 3, 8, 2);
    //获取最大的元素
    System.out.println("获取最大的元素" + Ordering.natural().max(list).toString());
    System.out.println("获取最大的元素" + Ordering.natural().reverse().max(list).toString());
}
```

### min(Iterable)

返回迭代器中最小的元素。如果可迭代对象中没有元素，则抛出NoSuchElementException。
