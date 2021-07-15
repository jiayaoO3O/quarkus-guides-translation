# Quarkus - 创建你的第一个应用程序

原文链接 : https://quarkus.io/guides/getting-started

学习如何创建一个**Hello World** Quarkus应用程序. 本教程包括 :

- 启动一个应用程序
- 创建一个JAX-RS端点 (📢 : 即接口, 下面看到翻译为端点的都可以理解为接口)
- 注入beans
- 功能测试
- 应用程序的打包

## 1. 先决条件

要完成这个教程, 你需要 :

- 不到15分钟
- 一个IDE
- JDK 11或者以上版本, 并且正确地配置了**JAVA_HOME**环境变量
- Apache Maven 3.8.1+ (📢 : 注意这是2.0.0.Final的新限制)

> 验证Maven是否使用了正确的java版本, 如果你安装了多个JDK, 那么Maven不一定能找到预期的Java, 你可能会得到意外的结果.
>
> 可以通过运行**mvn --version**来验证Maven使用的JDK

## 2. 架构

在本教程中, 我们创建了一个服务于`hello`端点的简单的应用程序, 为了演示依赖性注入, 这个端点使用了一个`greeting`Bean.

![](https://quarkus.io/guides/images/getting-started-architecture.png)



本教程还涵盖了对终端的测试.

## 3. 教程源码

我们建议你按照从[Bootstrapping](https://quarkus.io/guides/getting-started#bootstrapping-the-project)项目开始的说明, 一步一步地创建应用程序.

不过, 你可以直接进入已完成的例子.

下载[源码](https://github.com/quarkusio/quarkus-quickstarts/archive/main.zip)或克隆git仓库.

```shell
git clone https://github.com/quarkusio/quarkus-quickstarts.git
```

代码位于`getting-started`目录下.

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
- 一个暴露在`/hello` 上的 `org.acme.getting.started.GreetingResource`接口
- 一个与接口相关的单元测试
- 一个启动应用程序后可在http://localhost:8080 上访问的页面
- `src/main/docker`中用于本地(本地打包模式后面的教程会讲到)和jvm模式的**Dockerfile**模板
- 应用程序的配置文件

一旦生成项目, 查看pom.xml, 你会发现导入了Quarkus BOM, 它允许你省略不同Quarkus依赖项上的版本. 此外, 你还可以看到`quarkus-maven-plugin`, 它负责打包应用程序并提供开发模式.

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

在项目创建过程中, `src/main/java/org/acme/getting/started/GreetingResource.java`文件已经被创建, 内容如下 :

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

这是一个非常简单的REST接口, 对`/hello`的请求返回字符串`hello`.

> 与传统的 JAX-RS的区别
>
> 使用Quarkus, 不需要创建一个**Application**类, 可以, 但没必要. 此外, 多个请求只会有一个资源实例被创建, 而不是每个请求都会创建一个实例. 你可以使用不同的*Scoped注解（ApplicationScoped、RequestScoped等）来配置它.

## 5. 运行程序

现在准备运行我们的应用程序. 使用 ：`./mvnw  compile quarkus:dev`命令 :

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

点击`CTRL+C`停止应用程序, 或保持运行并享受极快的热重载 (📢 : 注意这里的热重载是quarkus开发模式的一项很给力的功能, 当你在程序运行的过程中修改了代码, 不需要重新编译运行程序, 只需要再一次调用某个接口, quarkus的dev插件会自动帮你热更新代码, 且重新启动项目)

> 用 curl -w "\n" 自动添加换行符
> 我们在这个例子中使用curl -w "\n", 以避免你的终端打印一个'%'或把结果和下一个命令提示放在同一行.

## 6. 使用注入

Quarkus中的依赖注入是基于ArC的, ArC是一个基于CDI的依赖注入解决方案, 为Quarkus的架构量身定做. 如果你是CDI的新手, 那么我们推荐你阅读[CDI简介](https://quarkus.io/guides/cdi)教程.

Quarkus只实现了CDI功能的一个子集, 并且带有非标准的功能和特定的APIs, 你可以在[Contexts和依赖注入](https://quarkus.io/guides/cdi-reference)教程中了解更多.

ArC作为quarkus-resteasy的一个依赖项, 在创建项目的时候已经自动添加了.

让我们修改应用程序并添加一个Bean. 创建`src/main/java/org/acme/getting/started/GreetingService.java`文件, 内容如下 :

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

编辑`GreetingResource`类, 注入`GreetingService`并使用它创建一个新的端点. 

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

如果你停止了应用程序, 用`./mvnw compile quarkus:dev`重新启动应用程序. 然后检查接口是否按预期返回`hello quarkus`. 

```shell
$ curl -w "\n" http://localhost:8080/hello/greeting/quarkus
hello quarkus
```

## 7. 开发模式

`quarkus:dev`命令会在开发模式下运行 Quarkus. 这可以实现代码的后台编译和热部署, 这意味着当你修改你的Java文件或者资源文件并刷新浏览器或者重新请求接口时, 这些变化将自动生效. 这也适用于配置属性文件. 刷新浏览器会触发对工作区的扫描, 如果检测到任何代码变化, Java文件会被重新编译, 应用程序会被重新部署; 然后你的请求会被重新部署的应用程序服务. 如果编译或部署有任何问题, 会显示一个错误的页面.

这也会监听5005debug端口. 如果你想在运行前等待调连接到5005端口, 可以在命令行中传递`-Dsuspend`参数. 如果你不想要debug, 可以使用`-Ddebug=false`. 

## 8. 测试

在生成的pom.xml文件中, 你可以看到2个测试依赖项. 

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-junit5</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <scope>test</scope>
</dependency>
```

Quarkus支持[Junit 5](https://junit.org/junit5/)测试. 正因为如此, 必须设置[Surefire Maven插件](https://maven.apache.org/surefire/maven-surefire-plugin/)的版本, 因为默认版本不支持Junit 5. 

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>${surefire-plugin.version}</version>
    <configuration>
       <systemPropertyVariables>
          <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
          <maven.home>${maven.home}</maven.home>
       </systemPropertyVariables>
    </configuration>
</plugin>
```

我们还设置了`java.util.logging`系统属性, 以确保测试使用正确的`logmanager`, 并设置了`maven.home`, 以确保应用`${maven.home}/conf/settings.xml`中的自定义配置被正确识别(如果有配置的话).

生成的项目包含一个简单的测试. 编辑 `src/test/java/org/acme/getting/started/GreetingResourceTest.java`, 以对应以下内容 :

```java
package org.acme.getting.started;

import io.quarkus.test.junit.QuarkusTest;
import org.junit.jupiter.api.Test;

import java.util.UUID;

import static io.restassured.RestAssured.given;
import static org.hamcrest.CoreMatchers.is;

@QuarkusTest
public class GreetingResourceTest {

    @Test    // (1)
    public void testHelloEndpoint() {
        given()
          .when().get("/hello")
          .then()
             .statusCode(200)    // (2)
             .body(is("hello"));
    }

    @Test
    public void testGreetingEndpoint() {
        String uuid = UUID.randomUUID().toString();
        given()
          .pathParam("name", uuid)
          .when().get("/hello/greeting/{name}")
          .then()
            .statusCode(200)
            .body(is("hello " + uuid));
    }

}
```

> (1) 通过使用QuarkusTest, 你指示JUnit在测试前启动应用程序. 
>
> (2) 检查HTTP响应状态码和内容

这些测试使用[RestAssured](http://rest-assured.io/), 但也可以随意使用你最喜欢的库 (📢 : 这里说的RestAssured强烈建议上网查一下用法, 是一个非常好用的判断测试结果的类库, 尤其是对返回的json的解析).

你可以用Maven命令来运行这些测试 :

```shell
./mvnw test
```

你也可以直接从你的IDE中运行测试 (要确保你先停止了应用程序).

默认情况下, 测试会在`8081`端口上运行, 以便不和正在运行的默认使用`8080`端口的应用程序冲突. 我们自动将RestAssured配置为使用该端口. 如果你想使用一个不同的测试类, 应该使用`@TestHTTPResource`注解来直接将被测试的接口的URL注入到该测试类的一个字段中. 这个字段可以是字符串, URL或URI的类型. 这个注解也可以被赋予测试路径的值. 

例如, 如果我想测试一个映射到`/myservlet`的接口方法, 我只需在我的测试中添加以下内容 :

```java
@TestHTTPResource("/myservlet")
URL testUrl;
```

测试端口可以通过`quarkus.http.test-port`配置属性来控制. Quarkus还创建了一个名为`test.url`的系统属性, 在不能使用注入的情况下设置基本测试URL. 

## 9. 与多模块项目或外部模块一起工作

Quarkus在构建时大量使用[Jandex](https://github.com/wildfly/jandex), 以发现各种类或注解. 其中一个可以立即识别的应用是CDI Bean. 因此, 如果没有正确设置jandex, 大多数Quarkus扩展将无法正常工作. 

感谢我们的Maven和Gradle插件, 这个索引默认已经在Quarkus的项目上创建时配置好了. 

不过, 在处理多模块项目时, 请务必阅读[Maven](https://quarkus.io/guides/maven-tooling#multi-module-maven)或[Gradle](https://quarkus.io/guides/gradle-tooling#multi-module-maven)指南中的多模块项目工作部分. 

如果你打算使用外部依赖模块 (例如, 一个用于所有对象的外部三方库), 你得需要通过添加Jandex依赖插件 (如果你能修改这个三方库的pom.xml) 或通过`application.properties`里面的`quarkus.index-dependency`属性（在你不能修改外部依赖的源码的情况下很有用）使得这些模块中得类容能够被正确索引.

请务必阅读CDI指南中的[Bean Discovery](https://quarkus.io/guides/cdi-reference#bean_discovery)部分以了解更多信息. 

📢 : 这一部分看起来比较复杂, 我简单一点解释就是, 如果你的项目想要做多模块打包, 类似于springboot上经常用的按照不同的模块实现不同的功能, 你必须在最外层的pom.xml文件里面添加以下的插件 :

```xml
<plugin>
    <groupId>org.jboss.jandex</groupId>
    <artifactId>jandex-maven-plugin</artifactId>
    <version>1.0.8</version>
    <executions>
        <execution>
            <id>make-index</id>
            <goals>
                <goal>jandex</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

剩下的配置照抄网上的springboot多模块项目教程创建就可以了.

## 10. 打包和运行应用程序

该应用程序使用`./mvnw package`进行打包. 打包完之后会在`/target`目录中产生几个结果 :

- `getting-started-1.0.0-SNAPSHOT.jar`, 它只包含项目的类和资源, 是Maven构建时产生的常规工件, 它不是可运行的jar (📢  : 可运行的jar是指可以使用`java -jar`命令运行的jar包)

- `quarkus-app`目录, 其中包含`quarkus-run.jar` 文件 , 这是一个可执行的jar. 请注意, 这并不是一个*`über-jar`*, 因为依赖关系被复制到了`quarkus-app/lib/`的子目录中. 

你可以通过使用 : `java -jar target/quarkus-app/quarkus-run.jar` 来运行程序.

> 如果你想把你的应用程序部署到某个地方（通常是在一个容器中）, 你需要部署整个quarkus-app目录. 
>
> 在运行应用程序之前, 不要忘记把你IDE正在跑的程序停了, 不然会提示端口冲突.

📢 : 这里我得解释一下, 这里说的jar包类型是什么意思, quarkus打包的时候, 如果你不指定任何jar包类型, 那么他默认的打包类型就是`fast-jar`类型, 这个类型下, 所有的打包结果, 包括依赖等待都会放在`quarkus-app`这个文件夹里面, 要把程序复制到服务器上运行的时候必须把整个`quarkus-app`文件夹都复制过去而不能只复制`quarkus-run.jar`这个jar包.

而当我们在配置文件里面指定打包类型`quarkus.package.type=uber-jar`, 这个时候整个项目就会类似于springboot那样被打包成一个"超级jar", 所有的依赖文件都被打包进去这个jar里面了, 部署到服务器时, 只要直接复制这个大jar包即可, 可以直接使用`java -jar`命令运行整个项目.

一般情况下都是直接配置`quarkus.package.type=uber-jar`以方便部署的.

## 11. 配置程序banner (横幅)

默认情况下, 当Quarkus应用程序启动时 (在常规或开发模式下), 它将显示一个ASCII艺术横幅. 通过在`application.properties`中设置`quarkus.banner.enabled=false`, 或者设置Java系统属性`-Dquarkus.banner.enabled=false` , 或者通过设置`QUARKUS_BANNER_ENABLED`环境变量为false, 可以禁用该横幅. 

此外, 用户可以通过将横幅文件放在` src/main/resources` 中并在 `application.properties `中配置 `quarkus.banner.path=name-of-file` 来提供一个自定义横幅.

## 12. What’s next?

本教程涵盖了使用Quarkus创建一个应用程序, 然而, 还有很多内容需要学习, 建议继续学习[构建本地可执行文件](https://quarkus.io/guides/building-native-image)教程(还没翻), 在那里你可以了解到创建本地可执行文件并将其打包到容器中. 如果你对响应式感兴趣, 推荐学习响应式入门指南(还没翻), 在那里你可以看到如何用Quarkus实现响应式应用. 

此外, [tooling教程](https://quarkus.io/guides/tooling)(还没翻)解释了如何 :

- 用一行命令建立一个项目的脚手架
- 启用开发模式（热重载）
- 在你最喜欢的IDE中导入该项目
- 以及更多

