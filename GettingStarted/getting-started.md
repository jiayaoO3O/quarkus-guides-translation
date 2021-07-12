# Quarkus - 创建你的第一个应用程序

学习如何创建一个**Hello World** Quarkus应用程序. 本教程包括 :

- 启动一个应用程序
- 创建一个JAX-RS端点(👋 : 即接口, 下面看到翻译为端点的都可以理解为接口)
- 注入beans
- 功能测试
- 应用程序的打包

## 1. 先决条件

要完成这个教程, 你需要 :

- 不到15分钟
- 一个IDE
- JDK 11或者以上版本, 并且正确地配置了**JAVA_HOME**环境变量
- Apache Maven 3.8.1+(👋 : 注意这是2.0.0.Final的新限制)

> 验证Maven是否使用了正确的java版本
>
> 如果你安装了多个JDK, 那么Maven不一定能找到预期的Java, 你可能会得到意外的结果.
>
> 可以通过运行**mvn --version**来验证Maven使用的JDK

## 2. 架构

在本教程中, 我们创建了一个服务于**hello**端点的简单的应用程序, 为了演示依赖性注入, 这个端点使用了一个**greeting**Bean.

![](https://quarkus.io/guides/images/getting-started-architecture.png)



本教程还涵盖了对终端的测试.

## 3. 教程源码

我们建议你按照从[Bootstrapping](https://quarkus.io/guides/getting-started#bootstrapping-the-project)项目开始的说明, 一步一步地创建应用程序.

不过, 你可以直接进入已完成的例子.

下载[源码](https://github.com/quarkusio/quarkus-quickstarts/archive/main.zip)或克隆git仓库.

```shell
git clone https://github.com/quarkusio/quarkus-quickstarts.git
```

代码位于**getting-started**目录下.

## 4. 创建项目

创建一个新的Quarkus项目最简单的方法是打开终端并运行以下命令 :

对于Linux和MacOS用户,

```shell
mvn io.quarkus:quarkus-maven-plugin:2.0.1.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=getting-started \
    -DclassName="org.acme.getting.started.GreetingResource" \
    -Dpath="/hello"
    
cd getting-started
```

对于Win用户,

- 如果使用cmd, (别用反斜杠).

```shell
mvn io.quarkus:quarkus-maven-plugin:2.0.1.Final:create -DprojectGroupId=org.acme -DprojectArtifactId=getting-started -DclassName="org.acme.getting.started.GreetingResource" -Dpath="/hello"
```

- 如果使用powershell, -D参数要用双引号括起来.

```shell
mvn io.quarkus:quarkus-maven-plugin:2.0.1.Final:create "-DprojectGroupId=org.acme" "-DprojectArtifactId=getting-started" "-DclassName=org.acme.getting.started.GreetingResource" "-Dpath=/hello"
```

项目会生成在`./getting-started`, 里面包含了如下内容 :

- 一个Maven架构的项目
- 一个暴露在`/hello` 上的 **org.acme.getting.started.GreetingResource**接口
- 一个与接口相关的单元测试
- 一个启动应用程序后可在http://localhost:8080 上访问的页面
- src/main/docker中用于本地(本地打包模式后面的教程会讲到)和jvm模式的**Dockerfile**模板
- 应用程序的配置文件

一旦生成项目, 查看pom.xml, 你会发现导入了Quarkus BOM, 它允许你省略不同Quarkus依赖项上的版本. 此外, 你还可以看到**quarkus-maven-plugin**, 它负责打包应用程序并提供开发模式.

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.quarkus</groupId>
            <artifactId>quarkus-universe-bom</artifactId>
            <version>${quarkus.platform.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>io.quarkus</groupId>
            <artifactId>quarkus-maven-plugin</artifactId>
            <version>${quarkus-plugin.version}</version>
            <extensions>true</extensions>
            <executions>
                <execution>
                    <goals>
                        <goal>build</goal>
                        <goal>generate-code</goal>
                        <goal>generate-code-tests</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

如果我们把重点放在依赖关系部分, 你可以看到允许开发REST应用程序的依赖.

```xml
    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-resteasy</artifactId>
    </dependency>
```

### 4.1 The JAX-RS 资源

在项目创建过程中, **src/main/java/org/acme/getting/started/GreetingResource.java**文件已经被创建, 内容如下 :

```java
package org.acme.getting.started;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class GreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello";
    }
}
```

这是一个非常简单的REST接口, 对"/hello "的请求返回字符串"hello".

> 与传统的 JAX-RS的区别
> 使用Quarkus, 不需要创建一个**Application**类, 可以, 但没必要.
>
> 此外, 多个请求只会有一个资源实例被创建, 而不是每个请求都会创建一个实例. 你可以使用不同的*Scoped注解（ApplicationScoped、RequestScoped等）来配置它.

## 运行程序

现在准备运行我们的应用程序. 使用 ：**./mvnw  compile quarkus:dev**命令 :

```shell
$ ./mvnw compile quarkus:dev
[INFO] --------------------< org.acme:getting-started >---------------------
[INFO] Building getting-started 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ getting-started ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/starksm/Dev/JBoss/Quarkus/starksm64-quarkus-quickstarts/getting-started/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ getting-started ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /Users/starksm/Dev/JBoss/Quarkus/starksm64-quarkus-quickstarts/getting-started/target/classes
[INFO]
[INFO] --- quarkus-maven-plugin:<version>:dev (default-cli) @ getting-started ---
Listening for transport dt_socket at address: 5005
2019-02-28 17:05:22,347 INFO  [io.qua.dep.QuarkusAugmentor] (main) Beginning quarkus augmentation
2019-02-28 17:05:22,635 INFO  [io.qua.dep.QuarkusAugmentor] (main) Quarkus augmentation completed in 288ms
2019-02-28 17:05:22,770 INFO  [io.quarkus] (main) Quarkus started in 0.668s. Listening on: http://localhost:8080
2019-02-28 17:05:22,771 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
```

一旦启动, 你可以开始请求对外提供的接口 :

```shell
$ curl -w "\n" http://localhost:8080/hello
hello
```

点击**CTRL+C**停止应用程序, 或保持运行并享受极快的热重载(👋 : 注意这里的热重载是quarkus开发模式的一项很给力的功能, 当你在程序运行的过程中修改了代码, 不需要重新编译运行程序, 只需要再一次调用某个接口, quarkus的dev插件会自动帮你热更新代码, 且重新启动项目)

> 用 curl -w "\n" 自动添加换行符
> 我们在这个例子中使用curl -w "\n", 以避免你的终端打印一个'%'或把结果和下一个命令提示放在同一行.

## 使用注入

Quarkus中的依赖注入是基于ArC的, ArC是一个基于CDI的依赖注入解决方案, 为Quarkus的架构量身定做. 如果你是CDI的新手, 那么我们推荐你阅读[CDI简介](https://quarkus.io/guides/cdi)教程.

Quarkus只实现了CDI功能的一个子集, 并且带有非标准的功能和特定的APIs, 你可以在[Contexts和依赖注入](https://quarkus.io/guides/cdi-reference)教程中了解更多.

ArC作为quarkus-resteasy的一个依赖项, 在创建项目的时候已经自动添加了.

让我们修改应用程序并添加一个Bean. 创建**src/main/java/org/acme/getting/started/GreetingService.java**文件, 内容如下 :

```java
package org.acme.getting.started;

import javax.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class GreetingService {

    public String greeting(String name) {
        return "hello " + name;
    }
}
```

编辑**GreetingResource**类, 注入**GreetingService**并使用它创建一个新的端点. 

```java
package org.acme.getting.started;

import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import org.jboss.resteasy.annotations.jaxrs.PathParam;

@Path("/hello")
public class GreetingResource {

    @Inject
    GreetingService service;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/greeting/{name}")
    public String greeting(@PathParam String name) {
        return service.greeting(name);
    }

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello";
    }
}
```

如果你停止了应用程序, 用**./mvnw compile quarkus:dev**重新启动应用程序. 然后检查接口是否按预期返回**hello quarkus**. 

```shell
$ curl -w "\n" http://localhost:8080/hello/greeting/quarkus
hello quarkus
```

