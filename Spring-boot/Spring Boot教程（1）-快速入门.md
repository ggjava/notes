## 概述

`Spring Boot`是由Pivotal团队提供的全新框架，它的出现使创建独立的、基于生产级`Spring`的应用程序变得很容易，您可以“just run”这些应用程序。

`Spring Boot`的核心思想就是约定大于配置，一切自动完成，大多数`Spring`引导应用程序只需要很少的配置。采用`Spring Boot`可以大大的简化你的开发模式，所有你想集成的常用框架，它都有对应的组件支持。

## 特征

* 创建独立的`Spring`应用程序
* 直接嵌入`Tomcat`、`Jetty`或`Undertow`(不需要部署`WAR`文件)
* 提供自定义的`starter`依赖项，以简化构建配置
* 在可能的情况下自动配置`Spring`和第三方库
* 提供可用于生产的特性，如度量标准、健康检查和外部配置
* 绝对不需要代码生成，也不需要`XML`配置

## 快速上手

接下来我们将学习如何快速的创建一个`Spring Boot`应用，并且实现一个简单的`Http`请求处理。通过这个例子对`Spring Boot`有一个初步的了解，并体验其结构简单、开发快速的特性。

`Spring`官方提供了非常方便的工具[Spring Initializr](https://start.spring.io/)来帮助我们创建`Spring Boot`应用。

### 使用Spring Initializr

1. 打开 https://start.spring.io/

![GC](/images/spring_initializr.png)

2. 选择构建工具 Maven Project、Java、Spring Boot 版本 2.1.8 以及一些工程基本信息，可参考下图所示：

3. 点击 Generate Project 下载项目压缩包

4. 解压后，使用 Idea 导入项目，File -> New -> Model from Existing Source.. -> 选择解压后的文件夹 -> OK，选择 Maven 一路 Next

### 使用Idea

1. 选择 File -> New —> Project… 弹出新建项目的框
![GC](/images/idea_spring_initializr.png)
2. 选择 Spring Initializr，Next 也会出现上述类似的配置界面，Idea 帮我们做了集成
3. 填写相关内容后，点击 Next 选择依赖的包再点击 Next，最后确定信息无误点击 Finish。

### 项目结构介绍

`Spring Boot` 建议的目录结果如下：

```
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── linuxcoming
    │   │           ├── Application.java
    │   │           ├── controller
    │   │           ├── domain
    │   │           └── service
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── com
                └── linuxcoming
                    └── ApplicationTests.java
```

`Spring Boot`的基础结构共三个文件:

* src/main/java 程序开发以及主程序入口
* src/main/resources 配置文件
* src/test/java 测试程序

`Spring Boot`建议的目录结果如下：

* Application.java 建议放到根目录下面,主要用于做一些框架配置
* domain 目录主要用于实体与数据访问层（Repository）
* service 层主要是业务类代码
* controller 负责页面访问控制

当然也可以根据自己的喜欢来进行更改
最后，启动`Application`的`main`方法，至此一个 `Java`项目搭建好了！

### 项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.linuxcoming</groupId>
    <artifactId>hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-hello</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

* 项目元数据：创建时候输入的`Project Metadata`部分，也就是Maven项目的基本元素，包括：`groupId`、`artifactId`、`version`、`name`、`description`等
* parent：继承`spring-boot-starter-parent`的依赖管理，控制版本与打包等内容
* dependencies：项目具体依赖，这里包含了`spring-boot-starter-web`用于实现HTTP接口（该依赖中包含了`Spring MVC`）；`spring-boot-starter-test`用于编写单元测试的依赖包。
* build：构建配置部分。默认使用了`spring-boot-maven-plugin`，配合`spring-boot-starter-parent`就可以把`Spring Boot`应用打包成JAR来直接运行。

### 编写Http接口

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

### 启动程序

直接运行 Application.java

或者使用`maven`命令:

```bash
mvn spring-boot:run
```

使用浏览器或`PostMan`等工具发起请求：http://localhost:8080/hello

可以看到页面返回：Hello World

> 默认使用tomcat容器，端口：8080

## 总结

`Spring Boot`让我们的`Spring`应用变的更轻量化。我们不必像以前那样繁琐的构建项目、打包应用、部署到`Tomcat`等应用服务器中来运行我们的业务服务。通过`Spring Boot`实现的服务，只需要依靠一个`Java`类，把它打包成`jar`，并通过`java -jar`命令就可以运行起来。这一切相较于传统`Spring`应用来说，已经变得非常的轻便、简单。我们不用关心框架之间的兼容性，适用版本等各种问题，我们想使用任何东西，仅仅添加一个配置就可以，所以使用`Spring Boot`非常适合构建微服务。





