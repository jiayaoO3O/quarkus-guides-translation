# Quarkus - 创建你的第一个应用程序

学习如何创建一个**Hello World** Quarkus应用程序。本教程包括 :

- 启动一个应用程序
- 创建一个JAX-RS端点(即接口, 下面看到翻译为端点的都可以理解为接口)
- 注入beans
- 功能测试
- 应用程序的打包

## 1. 先决条件

要完成这个教程, 你需要 :

- 不到15分钟
- 一个IDE
- JDK 11或者以上版本, 并且正确地配置了**JAVA_HOME**环境变量
- Apache Maven 3.8.1+(注意这是2.0.0.Final的新限制)

> 验证Maven是否使用了正确的java版本
>
> 如果你安装了多个JDK, 那么Maven不一定能找到预期的Java, 你可能会得到意外的结果.
>
> 可以通过运行**mvn --version**来验证Maven使用的JDK

## 2. 架构

在本教程中，我们创建了一个服务于**hello**端点的简单的应用程序, 为了演示依赖性注入，这个端点使用了一个**greeting**Bean.

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
- 一个暴露在**/hello**上的**org.acme.getting.started.GreetingResource**接口
- 一个与接口相关的单元测试
- 一个启动应用程序后可在http://localhost:8080 上访问的页面
- src/main/docker中用于本地(本地打包模式后面的教程会讲到)和jvm模式的**Dockerfile**模板
- 应用程序的配置文件

一旦生成项目, 查看pom.xml, 你会发现导入了Quarkus BOM, 它允许你省略不同Quarkus依赖项上的版本. 此外，你还可以看到**quarkus-maven-plugin**, 它负责打包应用程序并提供开发模式.

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

