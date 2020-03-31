## 介绍

构建软件项目通常包括以下任务:下载依赖项、在类路径上放置额外的JAR、将源代码编译成二进制代码、运行测试、将编译后的代码打包成可部署的构件(如JAR、WAR和ZIP文件)，并将这些构件部署到应用服务器或仓库。

Apache Maven将这些任务自动化，在手工构建软件并将编译和打包代码的工作与代码构建工作分离开来的同时，将人类出错的风险降到最低。

在本教程中，我们将探索这个功能强大的工具，它使用用XML编写的中心信息—项目对象模型(Project Object Model, POM)—来描述、构建和管理Java软件项目。

## 为什么使用Maven

Maven的主要特性:

* 遵循最佳实践的简单项目设置: Maven通过提供项目模板(命名原型)，尽量避免配置
* 依赖项管理: 它包括自动更新、下载和验证兼容性，以及报告依赖项闭包(也称为传递依赖项)
* 项目依赖项和插件之间的隔离: Maven从依赖项仓库检索项目依赖项，而任何插件的依赖项都从插件仓库检索，从而在插件开始下载附加依赖项时减少冲突
* 中央仓库系统: 可以从本地文件系统或公共仓库(如Maven central)加载项目依赖项

为了学习如何在您的系统上安装Maven，请查看关于Maven安装教程。

## 项目对象模型(POM)

Maven项目的配置是通过项目对象模型(POM)完成的，POM由POM .xml文件表示。POM描述项目，管理依赖关系，并配置用于构建软件的插件。

POM还定义了多模块项目的模块之间的关系。让我们来看看一个典型的POM文件的基本结构:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.linuxcoming</groupId>
    <artifactId>com.linuxcoming</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>com.linuxcoming</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
            //...
            </plugin>
        </plugins>
    </build>
</project>
```

让我们仔细看看这些构造。

### 项目标识符

Maven使用一组标识符(也称为坐标)来唯一标识一个项目，并指定项目构件应该如何打包:

* groupId 创建项目的公司或组的唯一基名
* artifactId 项目的唯一名称
* version 项目的一个版本
* packaging 一种包装方法(例如WAR/JAR/ZIP)

前三个(groupId:artifactId:version)组合在一起形成惟一标识符，并且是指定项目将使用哪些版本的外部库(例如jar)的机制。

### 依赖

项目使用的这些外部库称为依赖关系。Maven中的依赖关系管理特性确保从中央仓库自动下载这些库，因此不必在本地存储它们。

这是Maven的一个关键特性，它提供了以下好处:

* 通过显著减少从远程仓库下载的数量，使用更少的存储空间
* 使检查项目更快
* 提供一个有效的平台来交换组织内外的二进制工件，而不需要每次都从源代码构建

为了声明对外部库的依赖关系，您需要提供groupId、artifactId和库的版本。让我们来看一个例子:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.3.5.RELEASE</version>
</dependency>
```

当Maven处理依赖项时，它将下载Spring Core库到本地Maven仓库中。

### 仓库

Maven中的仓库用于保存构建构件和不同类型的依赖项。默认的本地仓库位于用户主目录下的.m2/repository文件夹中。

如果工件或插件在本地仓库中可用，Maven将使用它。否则，它将从中央仓库下载并存储在本地仓库中。默认的中央仓库是Maven central。

一些库，如JBoss server，在中央仓库中不可用，但在备用仓库中可用。对于这些库，您需要提供pom.xml文件中备用仓库的URL:

```xml
<repositories>
    <repository>
        <id>JBoss repository</id>
        <url>http://repository.jboss.org/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

请注意，您可以在项目中使用多个仓库。

### 自定义属性

自定义属性可以帮助您简化pom.xml文件的读取和维护。在经典用例中，您将使用自定义属性为项目的依赖项定义版本。

Maven属性是值占位符，可以通过使用${name}符号(其中name是属性)访问pom.xml中的任何位置。

我们来看一个例子:

```xml
<properties>
    <spring.version>4.3.5.RELEASE</spring.version>
</properties>
 
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
</dependencies>
```

现在，如果您想将Spring升级到一个较新的版本，您只需要更改<spring.version>属性标记及其<version>标记中使用该属性的所有依赖项将被更新。

属性也经常用于定义构建路径变量:

```xml
<properties>
    <project.build.folder>${project.build.directory}/tmp/</project.build.folder>
</properties>
 
<plugin>
    //...
    <outputDirectory>${project.resources.build.folder}</outputDirectory>
    //...
</plugin>
```

### 构建

构建部分也是Maven POM的一个非常重要的部分。它提供关于默认Maven目标、已编译项目的目录和应用程序的最终名称的信息。默认构建部分如下:

```xml
<build>
    <defaultGoal>install</defaultGoal>
    <directory>${basedir}/target</directory>
    <finalName>${artifactId}-${version}</finalName>
    <filters>
      <filter>filters/filter1.properties</filter>
    </filters>
    //...
</build>
```

编译工件的默认输出文件夹名为target，打包工件的最终名称由artifactId和version组成，但是您可以随时更改它。

### 使用配置文件

Maven的另一个重要特性是它对概要文件的支持。概要文件基本上是一组配置值。通过使用概要文件，您可以为不同的环境定制构建，例如生产/测试/开发:

```xml
<profiles>
    <profile>
        <id>production</id>
        <build>
            <plugins>
                <plugin>
                //...
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>development</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <build>
            <plugins>
                <plugin>
                //...
                </plugin>
            </plugins>
        </build>
     </profile>
 </profiles>
```

正如您在上面的示例中所看到的，默认配置文件被设置为development。如果您想运行生产概要文件，您可以使用以下Maven命令:

```
mvn clean install -Pproduction
```

## Maven生命周期

每个Maven构建都遵循一个指定的生命周期。您可以执行几个构建生命周期目标，包括编译项目代码、创建包和在本地Maven依赖存储库中安装归档文件的目标。

### 生命周期阶段

下面的列表显示了最重要的Maven生命周期阶段:

* validate—检查项目的正确性
* compile—将提供的源代码编译成二进制工件
* test—执行单元测试
* package—将编译的代码打包成存档文件
* integration-test—执行额外的测试，这些测试需要打包
* verify—检查包是否有效
* install—将包文件安装到本地Maven存储库中
* deploy—将包文件部署到远程服务器或存储库

### 插件和目标

Maven插件是一个或多个目标的集合。目标是分阶段执行的，这有助于确定目标执行的顺序。

这里有Maven正式支持的插件列表。

要更好地理解默认情况下哪些目标在哪些阶段运行，请查看默认Maven生命周期绑定。

要经历以上任何一个阶段，我们只需要调用一个命令:

```
mvn <阶段>
```

例如，*mvn clean install* 将删除以前创建的jar/war/zip文件和已编译类(clean)，并执行安装新归档文件(install)所需的所有阶段。

请注意插件提供的目标可以与生命周期的不同阶段相关联。

## 第一个maven项目

在本节中，我们将使用Maven的命令行功能来创建Java项目。

### 生成一个简单的Java项目

为了构建一个简单的Java项目，让我们运行以下命令:

```
mvn archetype:generate
  -DgroupId=com.linuxcoming
  -DartifactId=com.linuxcoming.java
  -DarchetypeArtifactId=maven-archetype-quickstart
  -DinteractiveMode=false
```

groupId是一个参数，指示创建项目的组或个人，通常是一个颠倒的公司域名。artifactId是项目中使用的基本包名，我们使用标准原型。

由于我们没有指定版本和包装类型，这些将被设置为默认值——版本将被设置为1.0-SNAPSHOT，包装将被设置为jar。

如果您不知道要提供哪些参数，则始终可以指定interactiveMode=true，以便Maven请求所有必需的参数。

命令完成后，我们在src/main/ Java文件夹中有一个Java项目，其中包含一个App.java类，它只是一个简单的“Hello World”程序。

我们还在src/test/java中有一个示例测试类。这个项目的pom.xml将类似于:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.linuxcoming</groupId>
    <artifactId>com.linuxcoming.java</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>com.linuxcoming.java</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.1.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

junit依赖项是默认提供的。

### 编译和打包一个项目

下一步是编译项目:

```
mvn compile
```

Maven将运行编译阶段构建项目源代码所需的所有生命周期阶段。如果你想只运行测试阶段，你可以使用:

```
mvn test
```

现在让我们调用包阶段，它将生成编译后的归档jar文件:

```
mvn package
```

### 执行应用程序

最后，我们将使用execm -maven-plugin执行Java项目。让我们在pom.xml中配置必要的插件:

```xml
<build>
    <sourceDirectory>src</sourceDirectory>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.6.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>1.5.0</version>
            <configuration>
                <mainClass>com.linuxcoming.java.App</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

第一个插件maven-compiler-plugin负责使用Java version 1.8编译源代码。execm -maven-plugin在我们的项目中搜索mainClass。

要执行应用程序，我们运行以下命令:

```
mvn exec:java
```

## 多模块项目

Maven中处理多模块项目(也称为聚合器项目)的机制称为反应器。

反应器收集所有要构建的可用模块，然后按照正确的构建顺序对项目进行排序，最后一个一个地构建它们。

让我们看看如何创建一个多模块父项目。

### 创建父项目

首先，我们需要创建一个父项目。为了创建一个名为parent-project的新项目，我们使用以下命令:

```
mvn archetype:generate -DgroupId=com.linuxcoming -DartifactId=parent-project
```

接下来，我们更新pom.xml文件中的包装类型，以表明这是一个父模块:

```
<packaging>pom</packaging>
```

### 创建子项目

下一步，从父项目目录创建子模块项目:

```
cd parent-project
mvn archetype:generate -DgroupId=com.linuxcoming  -DartifactId=core
mvn archetype:generate -DgroupId=com.linuxcoming  -DartifactId=service
mvn archetype:generate -DgroupId=com.linuxcoming  -DartifactId=webapp
```

为了验证我们是否正确地创建了子模块，我们查看了父项目pom.xml文件，其中应该有三个模块:

```xml
<modules>
    <module>core</module>
    <module>service</module>
    <module>webapp</module>
</modules>
```

此外，每个子模块的pom.xml中将添加父部分:

```xml
<parent>
    <groupId>com.linuxcoming</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>
```

### 在父项目中启用依赖项管理

依赖项管理是一种机制，用于集中多模块父项目及其子项目的依赖项信息。

当您拥有一组继承公共父类的项目或模块时，您可以将所有有关依赖关系的必要信息放在公共pom.xml文件中。这将简化对子POMs中的构件的引用。

让我们来看一个示例父类的pom.xml:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.3.5.RELEASE</version>
        </dependency>
        //...
    </dependencies>
</dependencyManagement>
```

通过在父类中声明spring-core版本，所有依赖于spring-core的子模块都可以仅使用groupId和artifactId声明依赖关系，该版本将被继承:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
    </dependency>
    //...
</dependencies>
```

此外，您还可以在parent的pom中为依赖项管理提供排除。使特定的库不会被子模块继承:

```xml
<exclusions>
    <exclusion>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </exclusion>
</exclusions>
```

最后，如果子模块需要使用托管依赖项的不同版本，可以在子模块的pom.xml文件中覆盖托管版本:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.2.1.RELEASE</version>
</dependency>
```

虽然子模块继承自父项目，但父项目不一定具有它聚合的任何模块。另一方面，父项目也可以聚合不继承自它的项目。

### 更新子模块并构建项目

我们可以更改每个子模块的封装类型。例如，让我们通过更新pom.xml文件将webapp模块的打包更改为WAR:

```
<packaging>war</packaging>
```

现在，我们可以使用*mvn clean install*命令测试项目的构建。Maven日志的输出应该类似如下:

```
[INFO] Scanning for projects...
[INFO] Reactor build order:
[INFO]   parent-project
[INFO]   core
[INFO]   service
[INFO]   webapp
//.............
[INFO] -----------------------------------------
[INFO] Reactor Summary:
[INFO] -----------------------------------------
[INFO] parent-project .................. SUCCESS [2.041s]
[INFO] core ............................ SUCCESS [4.802s]
[INFO] service ......................... SUCCESS [3.065s]
[INFO] webapp .......................... SUCCESS [6.125s]
[INFO] ---------------------------------------
```

## 总结

在本教程中，我们讨论了使用Maven多模块的好处。此外，我们还区分了常规Maven的父POM和聚合POM。最后，我们展示了如何设置一个简单的多模块来开始使用。

Maven是一个很棒的工具，但是它本身很复杂。如果您想了解关于Maven的更多细节，请查看Sonatype Maven引用或Apache Maven指南。如果您寻求Maven多模块设置的高级用法，请查看Spring Boot项目如何利用它的。
