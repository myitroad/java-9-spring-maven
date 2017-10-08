# Java 9 Spring MVC Example

This Repository is an example of how to use Spring with Java 9 modules, leveraging maven multi-modules for per module
dependency management.

## Intention for Writing

Interoperability between Spring and Java 9 is still very fiddly. This repository serves as a functioning example
that can be used to help diagnose some common issues.

## Running this example

**NOTE**: At the moment, you can't just dive on in and run the application inside an IDE. The support simply isn't there
yet.

From the root of the project:

```bash
mvn clean install
java -jar app/target/app-1.0-SNAPSHOT-exec.jar
```

> Note:
如果maven没有配置jdk9，则可能报错`无效的标记: --module-path`。
解决方法为在本地Maven安装目录conf/toolchains.xml文件中加入jdk9的toolchain节点，如下：
```xml
<toolchain>
    <type>jdk</type>
    <provides>
      <version>9</version>
      <vendor>oracle</vendor>
    </provides>
    <configuration>
      <!-- Change path to JDK9 -->
      <jdkHome>D:\Programs\Java\jdk_9</jdkHome>
    </configuration>
</toolchain>
```
并在项目父pom.xml中引入该toolchain，如下：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-toolchains-plugin</artifactId>
    <version>1.1</version>
    <configuration>
        <toolchains>
            <jdk>
                <version>9</version>
                <vendor>oracle</vendor>
            </jdk>
        </toolchains>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>toolchain</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
[以上参考](https://zhuanlan.zhihu.com/p/28010505)

This will spin up a spring boot application with a two endpoints, `/users/{id}` that you can fire a GET request at
to get some dummy output and `/admin/metrics` which will give you a dummy, basic metrics endpoint.

## How this works

### Dependency Management

Each of the modules are themselves maven modules, which a parent pom in the root of the project. The parent pom
orchestrates building each of the modules.

### Working with Spring

The admin and users modules don't contain a main method in them. They act as collections of all the functionality related
to admins and users.

The app module then brings these in as dependencies, which can be seen in `pom.xml` for the app module and in the `module-info.java`
inside `/src/main` in the app module.

This is configured to automatically detect beans on the classpath and in the current package or lower, so each of the modules
*must* be of the form `java9.spring.mvc.**`, otherwise Spring will not detect the beans.

The advantage of this approach is that the users and admin modules don't need to know anything about how the app is running.
They simply spin up the APIs they're interested in.

### Weird bits

Because they're maven multi modules, you need to define the dependencies in multiple different places. It would have been
preferable to define them in one place and require them in the `module-info.java`. Perhaps this is possible; I'm not a 
maven wizard so will investigate further.