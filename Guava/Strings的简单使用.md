
## Joiner

用分隔符把字符串序列连接起来也可能会遇上不必要的麻烦。如果字符串序列中含有`null`，那连接操作会更难。`Fluent` 风格的[Joiner](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Joiner.html)让连接字符串更简单。

```java
Joiner joiner = Joiner.on("; ").skipNulls();
return joiner.join("Harry", null, "Ron", "Hermione");
```

上述代码返回”Harry; Ron; Hermione”。另外，`useForNull(String)`方法可以给定某个字符串来替换`null`，而不像`skipNulls()`方法是直接忽略`null`。 `Joiner`也可以用来连接对象类型，在这种情况下，它会把对象的`toString()`值连接起来。

```java
Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"
```

> 警告：joiner 实例总是不可变的。用来定义`joiner`目标语义的配置方法总会返回一个新的`joiner`实例。这使得`joiner`实例都是线程安全的，你可以将其定义为`static final`常量。

## Splitter

JDK内建的字符串拆分工具有一些古怪的特性。比如，`String.split`悄悄丢弃了尾部的分隔符。 

问题：”,a,,b,”.split(“,”)返回？

1. “”, “a”, “”, “b”, “”
2. null, “a”, null, “b”, null
3. “a”, null, “b”
4. “a”, “b”
5. 以上都不对

正确答案是 5：””, “a”, “”, “b”。只有尾部的空字符串被忽略了。 [Splitter](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html)使用令人放心的、直白的流畅API模式对这些混乱的特性作了完全的掌控。

```java
Splitter.on(',')
    .trimResults()
    .omitEmptyStrings()
    .split("foo,bar,,   qux");
```

上述代码返回`Iterable`，其中包含”foo”、”bar”和”qux”。`Splitter`可以被设置为按照任何模式、字符、字符串或字符匹配器拆分。

### 基本构建器

| 方法 | 描述 | 范例 |
| - | - | - |
| [Splitter.on(char)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-char-) | 按单个字符拆分 | Splitter.on(‘;’) |
| [Splitter.on(CharMatcher)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-com.google.common.base.CharMatcher-) | 按字符匹配器拆分 | Splitter.on(CharMatcher.BREAKING_WHITESPACE) |
| [Splitter.on(String)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-java.lang.String-) | 按字符串拆分 | Splitter.on(“, “) |
| [Splitter.on(Pattern)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-java.util.regex.Pattern-) [Splitter.onPattern(String)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#onPattern-java.lang.String-) | 按正则表达式拆分 | Splitter.onPattern(“\r?\n”) |
| [Splitter.fixedLength(int)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#fixedLength-int-) | 按固定长度拆分；最后一段可能比给定长度短，但不会为空。 | Splitter.fixedLength(3) |

### 修饰符

| 方法 | 描述 |
| - | - |
| [omitEmptyStrings()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#omitEmptyStrings--) | 从结果中自动忽略空字符串 |
| [trimResults()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults--) | 移除结果字符串的前导空白和尾部空白 |
| [trimResults(CharMatcher)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults-com.google.common.base.CharMatcher-) | 给定匹配器，移除结果字符串的前导匹配字符和尾部匹配字符 |
| [limit(int)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#limit-int-) | 限制拆分出的字符串数量 |

如果你想要拆分器返回List，只要使用[splitToList()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#splitToList-java.lang.CharSequence-)。 

> 警告：`splitter`实例总是不可变的。用来定义`splitter`目标语义的配置方法总会返回一个新的`splitter`实例。这使得`splitter`实例都是线程安全的，你可以将其定义为`static final`常量。

### Map Splitters

还可以使用拆分器通过使用[withKeyValueSeparator()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#withKeyValueSeparator-java.lang.String-)指定第二个分隔符来反序列化映射。生成的[MapSplitter](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.MapSplitter.html)将使用splitter的分隔符将输入拆分为条目，然后使用给定的键值分隔符将这些条目拆分为键和值，返回Map<String, String>。

## CharMatcher

在过去，我们的`StringUtil`类不受约束地增长，并且有许多这样的方法:

* allAscii
* collapse
* collapseControlChars
* collapseWhitespace
* lastIndexNotOf
* numSharedChars
* removeChars
* removeCrLf
* retainAllChars
* strip
* stripAndCollapse
* stripNonDigits

这些方法指向两个概念上的问题：

1. 怎么才算匹配字符？
2. 如何处理这些匹配字符？

为了收拾这个泥潭，我们开发了`CharMatcher`。

直观上，你可以认为一个`CharMatcher`实例代表着某一类字符，如数字或空白字符。事实上来说，`CharMatcher`实例就是对字符的布尔判断——`CharMatcher`确实也实现了`Predicate`——但类似”所有空白字符”或”所有小写字母”的需求太普遍了，Guava因此创建了这一 API。

然而使用`CharMatcher`的好处更在于它提供了一系列方法，让你对字符作特定类型的操作：修剪[trim]、折叠[collapse]、移除[remove]、保留[retain]等等。`CharMatcher`实例首先代表概念。

1. 怎么才算匹配字符？然后它还提供了很多操作概念。
2. 如何处理这些匹配字符？这样的设计使得API复杂度的线性增加可以带来灵活性和功能两方面的增长。

```java
String noControl = CharMatcher.javaIsoControl().removeFrom(string); // remove control characters
String theDigits = CharMatcher.digit().retainFrom(string); // only the digits
String spaced = CharMatcher.whitespace().trimAndCollapseFrom(string, ' ');
  // trim whitespace at ends, and replace/collapse whitespace into single spaces
String noDigits = CharMatcher.javaDigit().replaceFrom(string, "*"); // star out all digits
String lowerAndDigit = CharMatcher.javaDigit().or(CharMatcher.javaLowerCase()).retainFrom(string);
  // eliminate all characters that aren't digits or lowercase
```

> 注：`CharMatcher` 只处理`char`类型代表的字符；它不能理解0x10000-0x10FFFF的`Unicode`增补字符。这些逻辑字符以代理对[surrogate pairs]的形式编码进字符串，而`CharMatcher`只能将这种逻辑字符看待成两个独立的字符。

### 获取字符匹配器

* [any()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#any--)
* [none()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#none--)
* [whitespace()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#whitespace--)
* [breakingWhitespace()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#breakingWhitespace--)
* [invisible()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#invisible--)
* [digit()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#digit--)
* [javaLetter()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaLetter--)
* [javaDigit()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaDigit--)
* [javaLetterOrDigit()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaLetterOrDigit--)
* [javaIsoControl()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaIsoControl--)
* [javaLowerCase()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaLowerCase--)
* [javaUpperCase()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaUpperCase--)
* [ascii()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#ascii--)
* [singleWidth()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#singleWidth--)

其他获取字符匹配器的常见方法包括：

| 方法 | 描述 |
| - | - |
| anyOf(CharSequence) | 枚举匹配字符。如 CharMatcher.anyOf(“aeiou”)匹配小写英语元音 |
| is(char) | 给定单一字符匹配。 |
| inRange(char, char) | 给定字符范围匹配，如 CharMatcher.inRange(‘a’, ‘z’) |

此外，`CharMatcher`还有[negate()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#negate--), [and(CharMatcher)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#and-com.google.common.base.CharMatcher-), [or(CharMatcher)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#or-com.google.common.base.CharMatcher-)它们在CharMatcher上提供简单的布尔操作。

### 使用字符匹配器

`CharMatcher`提供了多种方法来操作任何`CharSequence`中出现的指定字符。提供的方法比我们在这里列出的要多，但其中一些最常用的方法是:

| 方法 | 描述 |
| - | - |
| [collapseFrom(CharSequence, char)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#collapseFrom-java.lang.CharSequence-char-) | 把每组连续的匹配字符替换为特定字符。如 WHITESPACE.collapseFrom(string, ‘ ‘)把字符串中的连| 续空白字符替换为单个空格。 |
| [matchesAllOf(CharSequence)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#matchesAllOf-java.lang.CharSequence-) | 测试是否字符序列中的所有字符都匹配。 |
| [removeFrom(CharSequence)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#removeFrom-java.lang.CharSequence-) | 从字符序列中移除所有匹配字符。 |
| [retainFrom(CharSequence)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#retainFrom-java.lang.CharSequence-) | 在字符序列中保留匹配字符，移除其他字符。 |
| [trimFrom(CharSequence)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#trimFrom-java.lang.CharSequence-) | 移除字符序列的前导匹配字符和尾部匹配字符。 |
| [replaceFrom(CharSequence, CharSequence)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#replaceFrom-java.lang.CharSequence-java.lang.CharSequence-) | 用特定字符序列替代匹配字符。 |

> 所有这些方法返回`String`，除了`matchesAllOf`返回的是`boolean`。

### 字符集

不要这样做字符集处理：

```java
try {
  bytes = string.getBytes("UTF-8");
} catch (UnsupportedEncodingException e) {
  // how can this possibly happen?
  throw new AssertionError(e);
}
```

试试这样写：

```java
bytes = string.getBytes(Charsets.UTF_8);
```

[Charsets](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Charsets.html)针对所有`Java`平台都要保证支持的六种字符集提供了常量引用。尝试使用这些常量，而不是通过名称获取字符集实例。

## CaseFormat

`CaseFormat`被用来方便地在各种`ASCII`大小写规范间转换字符串——比如，编程语言的命名规范。`CaseFormat`支持的格式如下：

| 格式 | 范例 |
| - | - |
| LOWER_CAMEL | lowerCamel |
| LOWER_HYPHEN | lower-hyphen |
| LOWER_UNDERSCORE | lower_underscore |
| UPPER_CAMEL | UpperCamel |
| UPPER_UNDERSCORE | UPPER_UNDERSCORE |

CaseFormat的用法很直接：

```java
CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "CONSTANT_NAME")); // returns "constantName"
```

`CaseFormat` 在某些时候尤其有用，比如编写代码生成器的时候。

## Strings

`Strings`类中保留的通用`String`实用程序数量有限。
