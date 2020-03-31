java main 解释

## 概述

每个程序都需要一个开始执行的地方，说到Java程序，就是`main`方法。
我们已经习惯在编写代码时编写`main`方法，但是没有注意它的细节。在这篇简短的文章中，我们将分析这个方法，并展示其他一些编写方法。

## 方法签名

最常见的`main`法模板是:

```java
public static void main(String[] args) { }
```

这就是IDE自动为我们生成的代码。但这并不是这个方法唯一形式，我们可以使用一些有效的变体，并不是每个开发人员都注意到。

在研究这些方法签名之前，让我们先回顾一下常见签名的每个关键字的含义:

* public 访问修饰符，表示全局可见性
* static 方法可以直接从类中访问，我们不需要实例化一个对象来拥有引用并使用它
* void 表示此方法没有返回值
* main 方法名，这是JVM在执行Java程序时寻找的标识符

对于args参数，它表示方法接收到的值。这是我们第一次启动程序时传递参数的方式。

参数args是字符串数组。在下面的例子中:

```bash
java CommonMainMethodSignature foo bar
```

我们正在执行一个名为`CommonMainMethodSignature`的Java程序，并传递两个参数:`foo`和`bar`。这些值可以在主方法内部以`args[0]`(以foo为值)和args[1](以bar为值)的形式访问。

```java
public static void main(String[] args) {
    if (args.length > 0) {
        if (args[0].equals("test")) {
            // load test parameters
        } else if (args[0].equals("production")) {
            // load production parameters
        }
    }
}
```

还有一点就是ide也可以向程序传递参数。

## 其他形式

让我们看一下其他编写`main`方法的方式。虽然它们不是很常见，但它们是有效的签名。

注意，这些都不是`main`方法特有的，它们可以与任何Java方法一起使用，但是它们也是有效`main`方法。

```java
public static void main(String args[]) { }
```

```java
public static void main(String...args) { }
```

```java
public static void main(final String[] args) { }
```

```java
final static synchronized strictfp void main(final String[] args) { }
```

## 指定main方法

我们还可以在应用程序中定义多个`main`方法。

事实上，有些人使用它作为一种原始的测试技术来验证各个类(尽管像`JUnit`这样的测试框架更适合于这个技巧)。

要指定JVM应该执行哪个主方法作为应用程序的入口点，我们使用MF文件。在manifest里面，我们可以指明主类:

```java
Main-Class: mypackage.ClassWithMainMethod
```

这主要用于创建可执行的`.jar`文件。我们通过位于`META-INF/MANIFEST.MF`文件指出哪个类具有启动执行的主方法。

## 总结

本教程描述了`main`方法的细节，以及它可以采用的一些其他形式，甚至是对大多数开发人员来说不太常见的形式。

请记住，尽管我们展示的所有示例在语法上都是有效的，但它们只是用于学习目的，而且大多数时候我们将坚持使用公共签名来完成我们的工作。
