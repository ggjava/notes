## 概述

在本文中，我们将了解如何在Java中使用`HashMap`，以及它的一些内部实现。

首先看看`HashMap`是一个映射的含义。映射是`key-value`映射，这意味着每个键都映射到一个值，我们可以使用该键从映射检索对应的值。

有人可能会问，为什么不简单地将值添加到列表中。为什么需要`HashMap`?原因很简单，就是性能。如果我们想在一个列表中找到一个特定的元素，时间复杂度是`O(n)`，如果这个列表是排序的，它将是`O(log n)`，例如，使用二分查找。

`HashMap`的优点是插入和检索值的时间复杂度平均为`O(1)`。稍后我们将讨论如何实现这一点。让我们首先看看如何使用`HashMap`。

## 如何使用

让我们创建一个简单的类，我们将在整篇文章中使用:

```java
public class Product {

    private String name;
    private String description;
    private List<String> tags;

    // standard getters/setters/constructors

    public Product addTagsOfOtherProdcut(Product product) {
        this.tags.addAll(product.getTags());
        return this;
    }
}
```

### put

我们现在可以创建一个`HashMap`类型的关键字字符串和类型为`Product`的元素:

```java
Map<String, Product> productsByName = new HashMap<>();
```

并添加产品到我们的`HashMap`:

```java
Product eBike = new Product("E-Bike", "A bike with a battery");
Product roadBike = new Product("Road bike", "A bike for competition");
productsByName.put(eBike.getName(), eBike);
productsByName.put(roadBike.getName(), roadBike);
```

### get

我们可以通过键获取值:

```java
Product nextPurchase = productsByName.get("E-Bike");
assertEquals("A bike with a battery", nextPurchase.getDescription());
```

如果键对应的值不存在，我们会得到一个空值:

```java
Product nextPurchase = productsByName.get("Car");
assertNull(nextPurchase);
```

如果我们用相同的键插入第二个值，我们只会得到该键最后插入的值:

```java
Product newEBike = new Product("E-Bike", "A bike with a better battery");
productsByName.put(newEBike.getName(), newEBike);
assertEquals("A bike with a better battery", productsByName.get("E-Bike"));
```

### null为键

`HashMap`允许键为null:

```java
Product defaultProduct = new Product("Chocolate", "At least buy chocolate");
productsByName.put(null, defaultProduct);
 
Product nextPurchase = productsByName.get(null);
assertEquals("At least buy chocolate", nextPurchase.getDescription());
```

### 相同的值对应不同的键

```java
productsByName.put(defaultProduct.getName(), defaultProduct);
assertSame(productsByName.get(null), productsByName.get("Chocolate"));
```

### Remove

我们可以从`HashMap`中删除键-值映射:

```java
productsByName.remove("E-Bike");
assertNull(productsByName.get("E-Bike"));
```

### 检查key/value是否存在

要检查映射中是否存在键，可以使用`containsKey()`方法:

```java
productsByName.containsKey("E-Bike");
```

要检查映射中是否存在值，可以使用`containsValue()`方法:

```java
productsByName.containsValue(eBike);
```

在我们的示例中，这两个方法调用都将返回`true`。尽管它们看起来非常相似，但是这两个方法调用之间在性能上有一个重要的区别。检查键是否存在的复杂度是`O(1)`，而检查元素的复杂度是`O(n)`，因为需要遍历映射中的所有元素。

### 遍历

有三种基本方法可以遍历`HashMap`中的所有键-值对。

我们可以遍历所有键的集合:

```java
for(String key : productsByName.keySet()) {
    Product product = productsByName.get(key);
}
```

我们可以遍历所有值的集合:

```java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

最后，我们可以遍历所有值:

```java
List<Product> products = new ArrayList<>(productsByName.values());
```

## 关键点

我们可以使用任何类作为`HashMap`中的键。但是，为了使映射正常工作，我们需要提供`equals()`和`hashCode()`的实现。假设我们想要一个以产品为键，价格为值的`HashMap`:

```java
HashMap<Product, Integer> priceByProduct = new HashMap<>();
priceByProduct.put(eBike, 900);
```

让我们实现`equals()`和`hashCode()`方法:

```java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null || getClass() != o.getClass()) {
        return false;
    }

    Product product = (Product) o;
    return Objects.equals(name, product.name) &&
      Objects.equals(description, product.description);
}

@Override
public int hashCode() {
    return Objects.hash(name, description);
}
```

注意，`hashCode()`和`equals()`只需要为我们想要用作映射键的类而重写，而不需要为仅用作映射中的值的类重写。

## Java 8中的其他方法

`Java 8`向`HashMap`添加了几个函数风格的方法。在本节中，我们将研究其中的一些方法。

对于每种方法，我们将查看两个示例。第一个示例展示了如何使用新方法，第二个示例展示了如何在Java的早期版本中实现相同的功能。

由于这些方法非常简单，我们将不会看到更详细的示例。

### forEach()

`forEach`方法是遍历`map`中所有元素的函数式方法:

```java
productsByName.forEach( (key, product) -> {
    System.out.println("Key: " + key + " Product:" + product.getDescription());
    //do something with the key and value
});
```

Java 8之前:

```java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

### getOrDefault()

使用`getOrDefault()`方法，我们可以从映射中获取一个值，或者在给定键没有映射的情况下返回一个默认元素:

```java
Product chocolate = new Product("chocolate", "something sweet");
Product defaultProduct = productsByName.getOrDefault("horse carriage", chocolate); 
Product bike = productsByName.getOrDefault("E-Bike", chocolate);
```

Java 8之前:

```java
Product bike2 = productsByName.containsKey("E-Bike")
    ? productsByName.get("E-Bike")
    : chocolate;
Product defaultProduct2 = productsByName.containsKey("horse carriage") 
    ? productsByName.get("horse carriage")
    : chocolate;
```

### putIfAbsent()

使用这种方法，我们可以添加一个新的映射，但只有在给定键还没有映射时:

```java
productsByName.putIfAbsent("E-Bike", chocolate);
```

Java 8之前:

```java
if(productsByName.containsKey("E-Bike")) {
    productsByName.put("E-Bike", chocolate);
}
```

### merge()

使用`merge()`，如果存在映射，我们可以修改给定键的值，或者添加新值:

```java
Product eBike2 = new Product("E-Bike", "A bike with a battery");
eBike2.getTags().add("sport");
productsByName.merge("E-Bike", eBike2, Product::addTagsOfOtherProdcut);
```

Java 8之前:

```java
if(productsByName.containsKey("E-Bike")) {
    productsByName.get("E-Bike").addTagsOfOtherProdcut(eBike2);
} else {
    productsByName.put("E-Bike", eBike2);
}
```

### compute()

使用`compute()`方法，我们可以计算给定键的值:

```java
productsByName.compute("E-Bike", (k,v) -> {
    if(v != null) {
        return v.addTagsOfOtherProdcut(eBike2);
    } else {
        return eBike2;
    }
});
```

Java 8之前:

```java
if(productsByName.containsKey("E-Bike")) {
    productsByName.get("E-Bike").addTagsOfOtherProdcut(eBike2);
} else {
    productsByName.put("E-Bike", eBike2);
}
```

值得注意的是，`merge()`和`compute()`方法非常相似。`compute()`方法接受两个参数:用于重新映射的键和双函数。`merge()`接受三个参数:键、在键不存在时添加到映射的默认值，以及用于重新映射的双函数。

## 内部实现

在本节中，我们将研究`HashMap`如何在内部工作，以及使用`HashMap`而不是简单列表的好处。

正如我们所看到的，我们可以通过它的键从`HashMap`中检索元素。一种方法是使用列表，遍历所有元素，并在找到与键匹配的元素时返回。这种方法的时间和空间复杂度都是`O(n)`。

使用`HashMap`，我们可以实现`put`和`get`操作的平均时间复杂度`O(1)`和空间复杂度`O(n)`。让我们看看它是如何工作的。

### `equals()`和`hashCode()`

`HashMap`不是遍历所有元素，而是尝试根据键计算值的位置。

简单的方法是拥有一个包含尽可能多的键的元素的列表。例如，假设我们的键是小写字符。然后，有一个大小为26的列表就足够了，如果我们想要访问键为‘c’的元素，我们就知道它是位置3的元素，我们可以直接检索它。

但是，如果我们有更大的键空间，这种方法就不是很有效。例如，假设我们的键是整数。在这种情况下，列表的大小必须是2147,483,647。在大多数情况下，我们还会有更少的元素，所以大部分分配的内存仍然是未使用的。

`HashMap`将元素存储在所谓的`bucket`中，而`bucket`的数量称为容量。当我们想在映射中放入一个值时，`HashMap`根据键计算`bucket`并将值存储在那个`bucket`中。要检索该值，`HashMap`以完全相同的方式计算`bucket`。

### 碰撞

相同的键必须具有相同的散列，然而，不同的键可以具有相同的散列。如果两个不同的键具有相同的散列，属于它们的两个值将存储在同一个`bucket`中。在`bucket`中，值存储在一个列表中，并通过遍历所有元素来检索。这个的代价是`O(n)`

Java 8数据结构的值存储在一个桶从列表中改变平衡树如果一个`bucket`包含8个或多个值,它改变了回如果列表,在某种程度上,只剩下6个值是在`bucket`里。这样可以将性能提高到`O(log n)`。

### 容量和负载因子

为了避免多个`bucket`具有多个值，如果`bucket`的75%(负载因子)变为非空，则容量将增加一倍。负载因子的缺省值为75%，缺省初始容量为16。两者都可以在构造函数中设置。

### put和get操作的总结

让我们总结一下`put`和`get`操作是如何工作的。

当我们向映射添加一个元素时，`HashMap`将计算`bucket`。如果`bucket`已经包含了一个值，那么该值将被添加到属于该`bucket`的列表(或树)中。如果负载因子大于`map`的最大负载因子，则容量将增加一倍。

## 总结

在本文中，我们了解了如何使用`HashMap`及其内部工作方式。与`ArrayList`一样，`HashMap`是`Java`中最常用的数据结构之一，因此了解如何使用它以及它是如何工作的非常方便。