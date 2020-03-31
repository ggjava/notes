## 概览

在这个快速教程中，我们将展示如何在Maven中设置Java版本。

在继续之前，我们可以检查Maven的默认JDK版本。运行mvn -v命令将显示Maven运行的Java版本。

```bash
$ mvn -v
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /opt/apache-maven-3.6.1
Java version: 1.8.0_212, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk1.8.0_212.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.14.6", arch: "x86_64", family: "mac"
```

## 使用Compiler插件

我们可以在编译器插件中指定所需的Java版本。

### Compiler插件

在编译器插件属性中设置版本:

```xml
<properties>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.source>1.8</maven.compiler.source>
</properties>
```

Maven编译器接受带有 –target 和–source版本的命令。如果我们想使用Java 8语言的特性，应该将-source设置为1.8。

此外，要使编译后的类与JVM 1.8兼容，-target值应该是1.8。

它们的默认值都是1.6版本。

```xml
<plugins>
    <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```

maven-compiler-plugin还具有额外的配置属性，允许我们在源版本和目标版本之外对编译过程有更多的控制。

### Java 9及以上版本

此外，从JDK 9版本开始，我们可以使用一个新的-release命令行选项。这个新参数将自动配置编译器来生成类文件，该类文件将链接到给定平台版本的实现。

默认情况下，-source和-target选项不保证交叉编译。

这意味着我们不能在平台的旧版本上运行我们的应用程序。此外，要编译和运行用于旧Java版本的程序，还需要指定-bootclasspath选项。

为了正确地交叉编译，新的-release选项替换了三个标志:-source、-target和-bootclasspath。

在转换我们的例子之后，对于插件属性，我们可以声明:

```xml
<properties>
    <maven.compiler.release>7</maven.compiler.release>
</properties>
```

对于maven-compiler-plugin从3.6版本开始，我们可以这样写:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <configuration>
        <release>7</release>
    </configuration>
</plugin>
```

正如我们注意到的，我们可以在一个新的<release>属性中添加Java版本。在本例中，我们使用Java 7编译应用程序。

更重要的是，我们的机器中不需要安装JDK 7。Java 9已经包含了将新的语言特性链接到JDK 7的所有信息。

## Spring Boot规范

Spring Boot应用程序在pom.xml文件的属性标记中指定JDK版本。

首先，我们需要添加spring-boot-starter-parent作为我们项目的父类:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.5.RELEASE</version>
</parent>
```

这个父POM允许我们配置默认插件和多个属性，包括Java版本:默认情况下，Java版本是1.8。

但是，我们可以通过指定java来覆盖父版本的默认版本。版本属性:

```xml
<properties>
    <java.version>1.9</java.version>
</properties>
```

通过设置java.version属性声明源和目标Java版本都等于1.9。

最重要的是，我们应该记住这个属性是Spring引导规范。此外，从Spring Boot 2.0开始，Java 8是最小版本。

这意味着我们不能为旧的JDK版本使用或配置Spring Boot。

## 最后

本快速教程演示了在Maven项目中设置Java版本的可能方法。

总而言之:

* 使用<java.version>只可能与Spring引导应用程序一起使用
* 对于简单的情况，maven.compiler.source和maven.compiler.target属性应该是最合适的
* 最后，要对编译过程有更多的控制，请使用maven-compiler-plugin配置设置
