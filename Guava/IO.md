## 字节流和字符流

`Guava`使用术语“stream”来表示I/O数据的可关闭流，这些数据在底层资源中具有位置状态。术语`“byte stream”`指的是`InputStream`或`OutputStream`，而`“char stream”`指的是阅读器或写入器(尽管它们的`Readable`和`Appendable`常用作方法参数)。相应的实用程序分为[ByteStreams](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html)和[CharStreams](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html)实用程序类。

大多数`Guava`流工具一次处理一个完整的流，并且为了效率自己处理缓冲。还要注意到，接受流为参数的`Guava`方法不会关闭这个流：关闭流的职责通常属于打开流的代码块。

其中的一些工具方法列举如下：

| ByteStreams | CharStreams |
| - | - |
| [byte[] toByteArray(InputStream)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteStreams.html#toByteArray(java.io.InputStream)) | [String toString(Readable)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/CharStreams.html#toString(java.lang.Readable)) |
| N/A | [List<String> readLines(Readable)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharStreams.html#readLines(java.lang.Readable)) |
| [long copy(InputStream, OutputStream)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteStreams.html#copy(java.io.InputStream,%20java.io.OutputStream)) | [long copy(Readable, Appendable)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/CharStreams.html#copy(java.lang.Readable,%20java.lang.Appendable)) |
| [void readFully(InputStream, byte[])](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteStreams.html#readFully(java.io.InputStream,%20byte%5B%5D)) | N/A |
| [void skipFully(InputStream, long)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteStreams.html#skipFully(java.io.InputStream,%20long)) | [void skipFully(Reader, long)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharStreams.html#skipFully(java.io.Reader,%20long)) |
| [OutputStream nullOutputStream()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteStreams.html#nullOutputStream()) | [Writer nullWriter()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharStreams.html#nullWriter()) | 

### Sources和sinks

通常我们都会创建`I/O`工具方法，这样可以避免在做基础运算时总是直接和流打交道。例如，`Guava`有`Files.toByteArray(File)`和`Files.write(File, byte[])`。然而，流工具方法的创建经常最终导致散落各处的相似方法，每个方法读取不同类型的源或写入不同类型的`sink`。例如，`Guava`中的`Resources.toByteArray(URL)`和`Files.toByteArray(File)`做了同样的事情，只不过数据源一个是URL，一个是文件。

为了解决这个问题，`Guava`有一系列关于`sources`和`sinks`的抽象。`sources`和`sinks`指某个你知道如何从中打开流的资源，比如`File`或`URL`。`sources`是可读的，`sinks`是可写的。此外，`sources`和`sinks`按照字节和字符划分类型。

| Operations | Bytes | Chars |
| - | - | - |
| Reading | ByteSource | CharSource |
| Writing | ByteSink | CharSink |

`sources`和`sinks` API的好处是它们提供了通用的一组操作。比如，一旦你把数据源包装成了`ByteSource`，无论它原先的类型是什么，你都得到了一组按字节操作的方法。

### 创建Sources和Sinks

Guava 提供了若干`sources`和`sinks`的实现：

| Bytes | Chars |
| - | - |
| [Files.asByteSource(File)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asByteSource-java.io.File-) | [Files.asCharSource(File, Charset)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asCharSource-java.io.File-java.nio.charset.Charset-) |
| [Files.asByteSink(File, FileWriteMode...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asByteSink-java.io.File-com.google.common.io.FileWriteMode...-) | [Files.asCharSink(File, Charset, FileWriteMode...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asCharSink-java.io.File-java.nio.charset.Charset-com.google.common.io.FileWriteMode...-) |
| [MoreFiles.asByteSource(Path, OpenOption...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/MoreFiles.html#asByteSource-java.nio.file.Path-java.nio.file.OpenOption...-) | [MoreFiles.asCharSource(Path, Charset, OpenOption...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/MoreFiles.html#asCharSource-java.nio.file.Path-java.nio.charset.Charset-java.nio.file.OpenOption...-) |
| [MoreFiles.asByteSink(Path, OpenOption...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/MoreFiles.html#asByteSink-java.nio.file.Path-java.nio.file.OpenOption...-) | [MoreFiles.asCharSink(Path, Charset, OpenOption...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/MoreFiles.html#asCharSink-java.nio.file.Path-java.nio.charset.Charset-java.nio.file.OpenOption...-) |
| [Resources.asByteSource(URL)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Resources.html#asByteSource-java.net.URL-) | [Resources.asCharSource(URL, Charset)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Resources.html#asCharSource-java.net.URL-java.nio.charset.Charset-) |
| [ByteSource.wrap(byte[])](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#wrap-byte:A-) | [CharSource.wrap(CharSequence)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#wrap-java.lang.CharSequence-) |
| [ByteSource.concat(ByteSource...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#concat-com.google.common.io.ByteSource...-) | [CharSource.concat(CharSource...)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#concat-com.google.common.io.CharSource...-) |
| [ByteSource.slice(long, long)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#slice-long-long-) | N/A |
| [CharSource.asByteSource(Charset)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#asByteSource-java.nio.charset.Charset-) | [ByteSource.asCharSource(Charset)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#asCharSource-java.nio.charset.Charset-) |
| N/A | [ByteSink.asCharSink(Charset)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#asCharSink-java.nio.charset.Charset-) |

此外，你也可以继承这些类，以创建新的实现。

> 注：把已经打开的流（比如`InputStream`）包装为`source`和`sink`听起来是很有诱惑力的，但是应该避免这样做。`source`和`sink`的实现应该在每次`openStream()`方法被调用时都创建一个新的流。始终创建新的流可以让`source`或`sink`管理流的整个生命周期，并且让多次调用`openStream()`返回的流都是可用的。此外，如果你在创建源或汇之前创建了流，你不得不在异常的时候自己保证关闭流，这压根就违背了发挥`source`和`sink`API优点的初衷。

### 使用Sources和Sinks

一旦有了`source`和`sink`的实例，就可以进行若干读写操作。

#### 通用操作

所有`sources`和`sinks`都有一些方法用于打开新的流用于读或写。默认情况下，其他源与汇操作都是先用这些方法打开流，然后做一些读或写，最后保证流被正确地关闭了。这些方法列举如下：

* `openStream()`：根据`sources`和`sinks`的类型，返回`InputStream`、`OutputStream`、`Reader`或者`Writer`。
* `openBufferedStream()`：根据`sources`和`sinks`的类型，返回`InputStream`、`OutputStream`、`BufferedReader`或者 `BufferedWriter`。返回的流保证在必要情况下做了缓冲。例如，从字节数组读数据的源就没有必要再在内存中作缓冲，这就是为什么该方法针对字节源不返回`BufferedInputStream`。字符源属于例外情况，它一定返回`BufferedReader`，因为`BufferedReader`中才有`readLine()`方法。

#### Source操作

| ByteSource | CharSource |
| - | - |
| [byte[] read()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#read--) | [String read()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#read--) |
| N/A | [ImmutableList<String> readLines()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#readLines--) |
| N/A | [String readFirstLine()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#readFirstLine--) |
| [long copyTo(ByteSink)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#copyTo-com.google.common.io.ByteSink-) | [long copyTo(CharSink)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#copyTo-com.google.common.io.CharSink-) |
| [long copyTo(OutputStream)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#copyTo-java.io.OutputStream-) | [long copyTo(Appendable)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#copyTo-java.lang.Appendable-) |
| [Optional<Long> sizeIfKnown()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#sizeIfKnown--) | [Optional<Long> lengthIfKnown()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#lengthIfKnown--) |
| [long size()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#size--) | [long length()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#length--) |
| [boolean isEmpty()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#isEmpty--) | [boolean isEmpty()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#isEmpty--) |
| [boolean contentEquals(ByteSource)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#contentEquals-com.google.common.io.ByteSource-) | N/A |
| [HashCode hash(HashFunction)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#hash-com.google.common.hash.HashFunction-) | N/A |

#### Sink操作

| ByteSink | CharSink |
| - | - |
| [void write(byte[])](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#write-byte:A-) | [void write(CharSequence)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#write-java.lang.CharSequence-) |
| [long writeFrom(InputStream)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#writeFrom-java.io.InputStream-) | [long writeFrom(Readable)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeFrom-java.lang.Readable-) |
| N/A | [void writeLines(Iterable<? extends CharSequence>)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeLines-java.lang.Iterable-) |
| N/A | [void writeLines(Iterable<? extends CharSequence>, String)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeLines-java.lang.Iterable-java.lang.String-) |

#### 实例

```java
// Read the lines of a UTF-8 text file
ImmutableList<String> lines = Files.asCharSource(file, Charsets.UTF_8)
    .readLines();

// Count distinct word occurrences in a file
Multiset<String> wordOccurrences = HashMultiset.create(
    Splitter.on(CharMatcher.whitespace())
        .trimResults()
        .omitEmptyStrings()
        .split(Files.asCharSource(file, Charsets.UTF_8).read()));

// SHA-1 a file
HashCode hash = Files.asByteSource(file).hash(Hashing.sha1());

// Copy the data from a URL to a file
Resources.asByteSource(url).copyTo(Files.asByteSink(file));
```

## 文件操作

除了创建文件源和文件的方法，`Files`类还包含了若干你可能感兴趣的便利方法。

| 方法 | 描述 |
| - | - |
| [createParentDirs(File)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#createParentDirs-java.io.File-) | 必要时为文件创建父目录 |
| [getFileExtension(String)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#getFileExtension-java.lang.String-) | 返回给定路径所表示文件的扩展名 |
| [getNameWithoutExtension(String)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#getNameWithoutExtension-java.lang.String-) | 返回去除了扩展名的文件名 |
| [simplifyPath(String)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#simplifyPath-java.lang.String-) | 规范文件路径，并不总是与文件系统一致，请仔细测试 |
| [fileTreeTraverser()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#fileTraverser--) | 返回 TreeTraverser 用于遍历文件树 |
