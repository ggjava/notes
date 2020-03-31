## 概述

在我们的日常开发中，经常要对入参进行一定的参数校验，比如是否为空，参数的取值范围是否符合要求等等。这种参数校验如果我们单独进行校验的话，代码的重复率比较高，也不是很优雅。Guava提供了一个类Preconditions来统一校验我们的参数，同时可以抛出对应的异常信息，将参数校验的工作进行了统一。

## Preconditions

Preconditions类中的每个静态方法都支持三种方式:

* 无额外参数， 抛出的异常中没有错误消息
* 指定error message 抛出指定error message的异常
* 额外的字符串参数 替换带有占位符的error message消息。这个变种处理异常消息的方式有点类似printf，但考虑GWT的兼容性和效率，只支持%s指示符。

> 注意:checkNotNull、checkArgument和checkState有大量的重载，它们采用可变参数方式。

```java
checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
checkArgument(i < j, "Expected i < j, but %s > %s", i, j);
```

以下是com.google.common.base.Preconditions类的声明：

```java
@GwtCompatible
public final class Preconditions {}
```

### checkArgument

功能描述：检查boolean是否为真。用作方法中检查参数  
Throw Exception: IllegalArgumentException

```java
// Without an Error Message
@Test
public void whenCheckArgumentEvaluatesFalse_throwsException() {
    int age = -18;
    assertThatThrownBy(() -> Preconditions.checkArgument(age > 0))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessage(null).hasNoCause();
}

// With an Error Message
@Test
public void givenErrorMsg_whenCheckArgEvalsFalse_throwsException() {
    int age = -18;
    String message = "Age can't be zero or less than zero.";
  
    assertThatThrownBy(() -> Preconditions.checkArgument(age > 0, message))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessage(message).hasNoCause();
}

// With a Template Error Message
@Test
public void givenTemplateMsg_whenCheckArgEvalsFalse_throwsException() {
    int age = -18;
    String message = "Age should be positive number, you supplied %s.";
    assertThatThrownBy(
      () -> Preconditions.checkArgument(age > 0, message, age))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessage(message, age).hasNoCause();
}
```

### checkNotNull

功能描述：检查value不为null，直接返回value。  
Throw Exception：NullPointerException

```java
@Test
public void givenNullString_whenCheckNotNullWithMessage_throwsException () {
    String nullObject = null;
    String message = "Please check the Object supplied, its null!";
  
    assertThatThrownBy(() -> Preconditions.checkNotNull(nullObject, message))
      .isInstanceOf(NullPointerException.class)
      .hasMessage(message).hasNoCause();
}

@Test
public void whenCheckNotNullWithTemplateMessage_throwsException() {
    String nullObject = null;
    String message = "Please check the Object supplied, its %s!";
  
    assertThatThrownBy(
      () -> Preconditions.checkNotNull(nullObject, message,
        new Object[] { null }))
      .isInstanceOf(NullPointerException.class)
      .hasMessage(message, nullObject).hasNoCause();
}
```

### checkState

功能描述：检查对象的一些状态，不依赖方法参数。例如，Iterator可以用来next是否在remove之前被调用。  
Throw Exception：IllegalStateException

```java
@Test
public void checkState_throwsException() {
    int[] validStates = { -1, 0, 1 };
    int givenState = 10;
    String message = "You have entered an invalid state";
  
    assertThatThrownBy(
      () -> Preconditions.checkState(
        Arrays.binarySearch(validStates, givenState) > 0, message))
      .isInstanceOf(IllegalStateException.class)
      .hasMessageStartingWith(message).hasNoCause();
}
```

### checkElementIndex

功能描述：检查index是否为在一个长度为size的list， string或array合法的范围。 index的范围区间是[0, size)(包含0不包含size)。无需直接传入list， string或array， 只需传入大小。返回index。  
Throw Exception：IndexOutOfBoundsException

```java
@Test
public void checkElementIndex_throwsException() {
    int[] numbers = { 1, 2, 3, 4, 5 };
    String message = "Please check the bound of an array and retry";
  
    assertThatThrownBy(() -> 
      Preconditions.checkElementIndex(6, numbers.length - 1, message))
      .isInstanceOf(IndexOutOfBoundsException.class)
      .hasMessageStartingWith(message).hasNoCause();
}
```

### checkPositionIndex

功能描述：检查位置index是否为在一个长度为size的list， string或array合法的范围。 index的范围区间是[0， size)(包含0不包含size)。无需直接传入list， string或array， 只需传入大小。返回index。  
Throw Exception：IndexOutOfBoundsException

```java
@Test
public void checkPositionIndex_throwsException() {
    int[] numbers = { 1, 2, 3, 4, 5 };
    String message = "Please check the bound of an array and retry";
  
    assertThatThrownBy(
      () -> Preconditions.checkPositionIndex(6, numbers.length - 1, message))
      .isInstanceOf(IndexOutOfBoundsException.class)
      .hasMessageStartingWith(message).hasNoCause();
}
```

### checkPositionIndexes

功能描述：检查[start, end]是一个长度为size的list， string或array合法的范围子集。伴随着错误信息。  
Throw Exception：IndexOutOfBoundsException

```java
@Test
public void checkPositionIndexes_throwsException() {
    int start = 1;
    int end = 6;
    int[] numbers = { 1, 2, 3, 4, 5 };

    assertThatThrownBy(
            () -> Preconditions.checkPositionIndexes(start, end, numbers.length))
            .isInstanceOf(IndexOutOfBoundsException.class)
            .hasNoCause();
}
```
## 总结

相比Apache Commons提供的类似方法，我们把Guava中的Preconditions作为首选。

* 在静态导入后，Guava方法非常清楚明晰。checkNotNull清楚地描述做了什么，会抛出什么异常；
* checkNotNull直接返回检查的参数，让你可以在构造函数中保持字段的单行赋值风格：this.field = checkNotNull(field)
* 简单的、参数可变的printf风格异常信息。鉴于这个优点，在JDK7已经引入Objects.requireNonNull的情况下，我们仍然建议你使用checkNotNull。

在编码时，如果某个值有多重的前置条件，我们建议你把它们放到不同的行，这样有助于在调试时定位。此外，把每个前置条件放到不同的行，也可以帮助你编写清晰和有用的错误消息。
