## 概述

Lombok是一个库，它简化了许多繁琐的任务，并减少了Java源代码的冗长。当然，我们通常希望能够在IDE中使用库，这需要额外的设置。在本教程中，我们将讨论如何在两种最流行的Java ide (IntelliJ IDEA和Eclipse)中配置它。

## IntelliJ IDEA

### 启用注释处理

Lombok通过APT使用注释处理，因此，当编译器调用它时，库根据原始文件中的注释生成新的源文件。

> 不过，默认情况下不启用注释处理。

因此，我们要做的第一件事是在项目中启用注释处理。

打开 Preferences -> Build, Execution, Deployment -> Compiler -> Annotation Processors 勾选以下:

    * Enable annotation processing

    * Obtain processors from project classpath

![Enable annotation processing](/images/idea_enable_annotation_processing.png)

### 安装IDE插件

由于Lombok只在编译过程中生成代码，如果没有安装插件，IDE会突出显示原始源代码中的错误。 安装之后，错误就会消失，像查找用法、导航这样的常规功能就会开始工作。

打开 Preferences -> Plugins, 找到Marketplace标签页, 输入lombok，选择lombok Plugin by Michail Plushnikov

![idea install lombok plugin](/images/idea_install_lombok_plugin.png)

进入插件介绍页，点击安装：

![idea lombok plugin install button](/images/idea_lombok_plugin_install_button.png)

安装完成之后, 重启IDE：

![Enable annotation processing](/images/idea_lombok_install_finish.png)

## Eclipse

如果使用Eclipse IDE，首先需要获得Lombok.jar。最新版本位于Maven Central。在我们的示例中，我们使用的是lombok-1.18.4.jar。
接下来，我们可以通过java -jar命令运行jar，并打开一个安装程序UI。这将尝试自动检测所有可用的Eclipse安装，但也可以手动指定位置。
然后我们点击安装/更新按钮:

![eclipse lombok install](/images/eclipse_lombok_install.png)

如果安装成功，我们可以退出安装程序。

安装插件后，我们需要重新启动IDE并确保正确配置了Lombok。我们可以在About对话框中检查这个:

![eclipse lombok install finish](/images/eclipse_lombok_install_finish.png)

## 添加maven依赖

使用Maven，我们可以将依赖项添加到pom.xml:

```xml
<dependencies>
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.18.8</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

## 最后

Lombok在减少Java的冗长和覆盖底层的样板文件方面做得很好。在本文中，我介绍了如何为两个最流行的Java ide配置该工具。