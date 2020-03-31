## 生成可执行jar

我们通过创建一个完全独立的可执行 jar 文件来完成我们的示例，该文件可以在生产中运行。可执行 jar（有时称为"fat jars"）是包含已编译类以及代码需要运行的所有 jar 依赖项的存档。

### 添加maven-plugin

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 打包

执行`mvn package`

```bash
$ mvn package

[INFO] Scanning for projects...
[INFO] 
[INFO] -----------------------< com.linuxcoming:hello >------------------------
[INFO] Building spring-boot-hello 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:3.1.2:jar (default-jar) @ hello ---
[INFO] Building jar: /Users/.../spring-boot-examples/spring-boot-hello/target/hello-0.0.1-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.336 s
[INFO] Finished at: 2019-10-10T15:55:55+08:00
[INFO] ------------------------------------------------------------------------
```

### 查看jar内容

解压 hello-0.0.1-SNAPSHOT.jar

```bash
tar -zxvf target/hello-0.0.1-SNAPSHOT.jar
```

## 运行

```bash
$ java -jar target/hello-0.0.1-SNAPSHOT.jar

 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.8.RELEASE)

....... . . .
....... . . . (log output here)
....... . . .
2019-10-10 16:18:44.536  INFO 16976 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-10-10 16:18:44.536  INFO 16976 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1914 ms
2019-10-10 16:18:44.964  INFO 16976 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-10-10 16:18:45.274  INFO 16976 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-10-10 16:18:45.279  INFO 16976 --- [           main] com.linuxcoming.Application              : Started Application in 3.363 seconds (JVM running for 4.024)
```
