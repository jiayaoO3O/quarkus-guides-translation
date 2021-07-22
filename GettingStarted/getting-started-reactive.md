# Quarkus - 开始响应式编程

原文链接 : [https://quarkus.io/guides/getting-started-reactive](https://quarkus.io/guides/getting-started-reactive)

📢 : 本篇教程将会带你进入一个你可能从来没有接触过的世界, 即响应式编程, 这个响应式并不是指我们平时在网页上看到的响应式UI布局, 而是指程序通过异步, 非阻塞地对I/O事件做出反应与处理. 在Spring家族中, 与之对应的是Spring WebFlux, 如果你以前没有听过响应式编程, 那我强烈建议你先上网搜一下什么是"Spring WebFlux", 或者看一看这一篇[入门小文章](https://segmentfault.com/a/1190000021038373).

这是一个和传统java开发完全不一样的世界, 一个完全异步非阻塞的美妙世界, 开发响应式程序使用的语法, 代码风格等和以前的传统java编程模式是完全不一样, 希望你在学习前有心理准备😁.

需要注意以下两点 : 

1. quarkus并不强制你学习响应式编程, 你使用quarkus开发的代码既可以是同步的, 也可以是异步的; 既可以是响应式的, 也可以是非响应式的.
2. 响应式编程并**不会**让你的程序变得更显著的快, 它最显著的作用是能让程序在相同硬件性能下**提高并发数**, 但是如果你不差钱, 直接更换性能更好的机器也能够提高并发数. 所以在我看来, 学习响应式编程最大的作用是, 能让你享受收获知识的快乐😁.

了解如何用Quarkus创建一个响应式应用程序, 并探索Quarkus提供的不同响应式功能. 本教程包括 :

- 快速看一下Quarkus引擎以及它是如何实现响应式的
- 简要介绍Mutiny - Quarkus使用的响应式编程库 (📢 : 究极重点)
- RESTEasy、RESTEasy Reactive和Reactive Routes之间的区别
- 使用RESTEasy Reactive启动一个响应式应用程序
- 创建一个响应式JAX-RS接口
- 使用响应式数据库访问
- 与其他响应式API交互

## 先决条件

要完成这个教程, 你需要 :

- 不到15分钟
- 一个IDE
- JDK 11或者以上版本, 并且正确地配置了**JAVA_HOME**环境变量
- Apache Maven 3.8.1+ (📢 : 注意这是2.0.0.Final的新限制)

##  项目源码

我们建议你按照从[Bootstrapping](https://quarkus.io/guides/getting-started-reactive#bootstrapping-the-project)项目开始的说明, 一步一步地创建应用程序.

不过, 你可以直接进入已完成的例子.

下载[源码](https://github.com/quarkusio/quarkus-quickstarts/archive/main.zip)或克隆git仓库.

```shell
git clone https://github.com/quarkusio/quarkus-quickstarts.git
```

代码位于`getting-started-reactive`目录和`getting-started-reactive-crud`下.

## Quarkus响应式的多个方面

Quarkus是响应式的. 如果你打开引擎盖看一下, 会发现一个响应式引擎为你的Quarkus应用程序提供动力. 这个引擎就是**Eclipse Vert.x** (https://vertx.io). 所有的网络I/O都通过非阻塞和响应式的Vert.x引擎. 

![Quarkus is based on a reactive engine](https://quarkus.io/guides/images/quarkus-reactive-stack.png)

让我们举两个例子来解释它是如何工作的. 想象传入一个HTTP请求. 嵌入Quarkus的 (Vert.x)HTTP服务器接口接收到请求, 然后将其路由到应用程序. 如果请求的目标是一个命令式(📢 : 命令式是指传统的java开发代码)方法 (传统的JAX-RS接口, 用@Blocking注释的代码......), 路由层会在一个**工作线程worker thread**中调用资源方法, 并在数据可用时写入响应. 到目前为止, 和传统的开发方法没有区别, 没有什么新的或突出的问题. 下面的图片描述了这种行为. 在这种情况下, 应用程序代码在**工作线程**中被调用, 而业务逻辑可以**阻塞**该线程. 

![Behavior when using the imperative routes](https://quarkus.io/guides/images/http-blocking-sequence.png)

但是, 如果HTTP请求的目标是一个**响应式方法** (使用RESTEasy Reactive的JAX-RS, 响应式路由, @Incoming方法不注有@Blocking...), 路由层在**I/O线程**上调用路由, 这会带来很多好处, 比如更高的并发性和性能. 

![Behavior when using the reactive routes](https://quarkus.io/guides/images/http-reactive-sequence.png)

这时Quarkus使用I/O线程来调用你的代码, 这可以节省(📢 : save这里我不确定是指节省还是指保存)上下文切换, 避免大量的线程池管理, 从而提高资源利用率. 但是, 代码**绝不能**阻塞该线程. 为什么呢？因为, I/O线程是用来处理多个并发请求的. 一旦一个请求因为需要执行一些I/O而不能取得进展, quarkus就会调度这些I/O并继续程序剩下的部分 (📢 : passes a continuation真心难翻译). 它会释放线程以处理其他请求. 当刚刚没有进展的I/O完成后, 会继续被执行, 回到I/O线程上. 

因此, 许多Quarkus组件的设计都考虑到了响应式, 如数据库访问 (PostgreSQL、MySQL、Mongo等)、应用服务 (邮件、模板引擎等)、消息传递 (Kafka、AMQP等)等等. 但是, 为了充分受益于这种模式, 应用程序代码应该以**非阻塞的方式编写** (📢 : 指你的代码从接口层, service层, dao层这一整条链路的所有代码得是响应式的, 这也是目前响应式编程难以落地的原因). 在这里，响应式API是一种终极武器. 

## Mutiny - 一个响应式编程库

📢 : 这个部分是整个quarkus响应式编程的最重要的地方, 可以说, 学习怎么在quarkus中响应式编程的本质, 就是学习怎么调用Mutiny这个类库, 所以这里我要先介绍一下Mutiny是什么,Vert.x又是什么.

[Vert.x](https://vertx.io/)是一个基于全异步Java服务器Netty (Spring WebFlux也是基于Netty)的JVM、轻量级、高性能应用平台,  但是quarkus团队觉得Vert.x的上手门槛太高, 而且API设计过于拉了, 所以开发了一个Mutiny库, 来封装和统一Vert.x的API. 这里我强烈建议先看一遍Mutiny的[官网教程](https://smallrye.io/smallrye-mutiny/getting-started/first-lines-of-code), 我后续看看能不能也把这部分翻译了.

quarkus的教程就已经不多了, Mutiny的教程就更加少了.

[Mutiny](https://github.com/smallrye/smallrye-mutiny)是一个响应式编程库, 允许表达和编排异步动作. 它提供两种类型 :

- **io.smallrye.mutiny.Uni** - 用于提供0或1个item的异步行为 (意思是一个Uni< String >内部要么没有String, 要么有一个String)
- **io.smallrye.mutiny.Multi** - 用于0个或多个item的 (带背压)流 (意思是一个Multi < String > 内部要么没有String, 要么有一个或很多个String)

这两种类型都是惰性的, 并遵循一个订阅模式. 只有在有实际需要的时候才开始计算 (即有订阅者加入). 

```java
uni.subscribe().with(
    result -> System.out.println("result is " + result),
    failure -> failure.printStackTrace()
);

multi.subscribe().with(
    item -> System.out.println("Got " + item),
    failure -> failure.printStackTrace()
);
```

Uni和Multi都提供了事件驱动的API: 你可以表达你想在一个给定的事件 (成功、失败等)中做什么. 这些API被分为几组 (操作类型), 以使其更具表现力, 并避免在一个单一的类上有100多个方法. 主要的操作类型是关于对失败的反应, 完成, 操作项目, 提取, 或收集它们. 它通过可导航的API提供了一个流畅的编码体验, 因此不需要太多关于响应式的知识. 

```java
httpCall
    .onFailure().recoverWithItem("my fallback");
```

你可能想知道Reactive Streams (https://www.reactive-streams.org/). Multi实现了**Reactive Streams Publisher**, 因此实现了Reactive Streams的**反压机制** (也可翻译成背压, 背压的意思是当消费者消费的速度跟不上生产者生产速度时所采取的一种机制). Uni没有实现Publisher, 因为对Uni的订阅就足以表明你对结果感兴趣. 这又是考虑到了更简单、更顺畅的API的想法, 因为Reactive Streams的订阅/请求仪式更复杂. 

拥抱Quarkus的响应式和命令式统一的支柱吧, Uni和Multi都提供了通往命令式结构 (📢 : 命令式结构即传统的java代码形式)的桥梁. 例如, 你可以将Multi转化为Iterable或者*await* Uni产出的item. 

```java
// 阻塞直到结果可用
String result = uni.await().indefinitely();

// 将异步的流转换为同步的迭代器
stream.subscribe().asIterable().forEach(s -> System.out.println("Item is " + s));
```

在这一点上, 如果你是RxJava或Reactor的用户, 你可能会想如何使用你熟悉的Flowable、Single、Flux、Mono...Mutiny允许将Unis和Multis从RX Java和Reactor类型中转换出来. 

```java
Maybe<String> maybe = uni.convert().with(UniRxConverters.toMaybe());
Flux<String> flux = multi.convert().with(MultiReactorConverters.toFlux());
```

但是, Vert.x呢？Vert.x的API也可以使用Mutiny类型. 下面的片段显示了Vert.x Web客户端的用法. 

```java
// Use io.vertx.mutiny.ext.web.client.WebClient
client = WebClient.create(vertx,
                new WebClientOptions().setDefaultHost("fruityvice.com").setDefaultPort(443).setSsl(true)
                        .setTrustAll(true));
// ...
Uni<JsonObject> uni =
    client.get("/api/fruit/" + name)
        .send()
        .onItem().transform(resp -> {
            if (resp.statusCode() == 200) {
                return resp.bodyAsJsonObject();
            } else {
                return new JsonObject()
                        .put("code", resp.statusCode())
                        .put("message", resp.bodyAsString());
            }
        });
```

最后但并非最不重要的是, Mutiny内置了与`SmallRye Context Propagation`的集成, 因此你可以在你的响应式管道中传播事务、可追溯数据等. 

不过, 说得够多了, 让我们动手吧!

## 运行程序

有几种方法可以用Quarkus实现响应式应用. 在本教程中, 我们将使用`RESTEasy Reactive`, 这是一个受益于Quarkus响应式引擎的RESTEasy实现. 默认情况下, 它在I/O线程上调用HTTP端点. 

> 虽然可以使用传统的RESTEasy和Mutiny搭配, 但你需要添加`quarkus-resteasy-mutiny`扩展, 而且该方法仍然会在一个**工作线程**上被调用. 因此, 虽然这种搭配照样可以进行响应式编程, 但却还是需要工作线程, 这就违背了响应式的目的. 

对于Linux和MacOS用户,

```shell
mvn io.quarkus:quarkus-maven-plugin:2.0.2.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=getting-started-reactive \
    -DclassName="org.acme.getting.started.ReactiveGreetingResource" \
    -Dpath="/hello" \
    -Dextensions="resteasy-reactive"
cd getting-started-reactive
```

对于Win用户,

- 如果使用cmd, (别用反斜杠).

```shell
mvn io.quarkus:quarkus-maven-plugin:2.0.2.Final:create -DprojectGroupId=org.acme -DprojectArtifactId=getting-started-reactive -DclassName="org.acme.getting.started.ReactiveGreetingResource" -Dpath="/hello" -Dextensions="resteasy-reactive"
```

- 如果使用powershell, -D参数要用双引号括起来.

```shell
mvn io.quarkus:quarkus-maven-plugin:2.0.2.Final:create "-DprojectGroupId=org.acme" "-DprojectArtifactId=getting-started-reactive" "-DclassName=org.acme.getting.started.ReactiveGreetingResource" "-Dpath=/hello" "-Dextensions=resteasy-reactive"
```

项目会生成在`./getting-started-reactive`, 里面包含了如下内容 :

- 一个Maven架构的项目
- 一个暴露在`/hello` 上的 `org.acme.quickstart.ReactiveGreetingResource`接口
- 一个与接口相关的单元测试
- 一个启动应用程序后可在http://localhost:8080 上访问的页面
- `src/main/docker`中用于本地(本地打包模式后面的教程会讲到)和jvm模式的**Dockerfile**模板
- 应用程序的配置文件

### 响应式JAX-RS资源

在项目创建过程中, `src/main/java/org/acme/getting/started/ReactiveGreetingResource.java`文件已经被创建, 内容如下 :

```java
package org.acme.getting.started;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "Hello RESTEasy Reactive";
    }
}
```

这是一个非常简单的REST端点, 对`/hello`的请求返回 `Hello RESTEasy Reactive`字符串. 由于它使用了`RESTEAsy Reactive`, 这个方法是在**I/O线程**上调用的. 

> 如果想强行让Quarkus在一个**工作线程**上调用这个方法, 请用`io.smallrye.common.annotation.Blocking`注解来注解它. 你可以在一个方法、一个类上使用`@Blocking`, 也可以通过注解一个应用类来为整个应用程序启用它. 
>
> ```java
> import javax.ws.rs.ApplicationPath;
> import javax.ws.rs.core.Application;
> import io.smallrye.common.annotation.Blocking;
> 
> @ApplicationPath("/")
> @Blocking
> public class RestBlockingApplication extends Application {
> }
> ```

现在让我们创建一个`ReactiveGreetingService`类, 内容如下 :

```java
package org.acme.getting.started;

import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;

import javax.enterprise.context.ApplicationScoped;
import java.time.Duration;

@ApplicationScoped
public class ReactiveGreetingService {

    public Uni<String> greeting(String name) {
        return Uni.createFrom().item(name)
                .onItem().transform(n -> String.format("hello %s", n));
    }
}
```

然后, 编辑`ReactiveGreetingResource`类以匹配以下内容 :

```java
package org.acme.getting.started;

import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;
import org.reactivestreams.Publisher;

@Path("/hello")
public class ReactiveGreetingResource {

    @Inject
    ReactiveGreetingService service;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/greeting/{name}")
    public Uni<String> greeting(String name) {
        return service.greeting(name);
    }

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello";
    }
}
```

`ReactiveGreetingService`类包含一个产生Uni的简单方法. 虽然在这个例子中, 产生的item被立即返回, 但你可以想象到的是任何异步API的返回值都是一个Uni或者Multi. 我们将在本教程的后面介绍这一点. 

现在, 用以下方式启动应用程序 :

```shell
mvn quarkus:dev
```

一旦运行, 通过打开http://localhost:8080/hello/greeting/neo, 检查你是否得到预期的问候信息. 

## 处理数据流

到目前为止, 我们只返回一个异步的结果. 在这一节中, 我们用传达多个项目的流来扩展这个应用程序. 这些流可以来自Kafka或任何其他数据源, 但为了保持简单, 我们只是生成定期的问候信息. 

在`ReactiveGreetingService`中, 添加以下方法 :

```java
public Multi<String> greetings(int count, String name) {
  return Multi.createFrom().ticks().every(Duration.ofSeconds(1))
        .onItem().transform(n -> String.format("hello %s - %d", name, n))
        .select().first(count);
}
```

> 你可能需要添加 `import io.smallrye.mutiny.Multi`; 和`import java.time.Duration`; 语句. 

它每秒钟生成一条问候信息, 在计数信息后停止. 

在`ReactiveGreetingResource`中添加以下方法. 

```java
@GET
@Produces(MediaType.APPLICATION_JSON)
@Path("/greeting/{count}/{name}")
public Multi<String> greetings(int count, String name) {
  return service.greetings(count, name);
}
```

这个端点将项目以JSON数组的形式流向客户端. 消息的名称和数量是用路径参数来设定的. 

因此, 调用该端点会产生类似的结果 :

```shell
$ curl http://localhost:8080/hello/greeting/3/neo
["hello neo - 0", "hello neo - 1", "hello neo - 2"]
```

我们还可以通过返回一个Multi来生成**服务器发送的事件响应**:

```java
@GET
@Produces(MediaType.SERVER_SENT_EVENTS)
@RestSseElementType(MediaType.TEXT_PLAIN)
@Path("/stream/{count}/{name}")
public Multi<String> greetingsAsStream(int count, String name) {
    return service.greetings(count, name);
}
```

与前面的代码片段唯一不同的是`@Produces`的类型和 `@RestSseElementType` 注解代表的每个事件的类型. 由于 `@Produces` 注解定义了 `SERVER_SENT_EVENTS`, JAX-RS 需要它来知道每个 (嵌套)事件的内容类型. 

> 你可能需要添加 `import org.jboss.resteasy.reactive.RestSseElementType`; 语句. 

你可以通过以下方式看到结果 :

```shell
$ curl -N http://localhost:8080/hello/stream/5/neo
data: hello neo - 0

data: hello neo - 1

data: hello neo - 2

data: hello neo - 3

data: hello neo - 4
```

## 使用响应式API

### 使用Quarkus响应式API

Quarkus使用Mutiny模型提供了许多响应式API. 在这一节中, 我们将看到如何使用`Reactive PostgreSQL`驱动, 以非阻塞和响应式的方式与你的数据库交互. 

使用创建一个新的项目 :

```bash
mvn io.quarkus:quarkus-maven-plugin:2.0.2.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=getting-started-reactive-crud \
    -DclassName="org.acme.reactive.crud.FruitResource" \
    -Dpath="/fruits" \
    -Dextensions="resteasy-reactive,resteasy-reactive-jackson,reactive-pg-client"
cd getting-started-reactive-crud
```

这个应用程序是与PostgreSQL数据库交互的, 所以你需要一个数据库, 这里直接用docker启动一个, 如果没安装docker则需要本机安装一个数据库 :

```bash
docker run --ulimit memlock=-1:-1 -it --rm=true --memory-swappiness=0 \
           --name postgres-quarkus-reactive -e POSTGRES_USER=quarkus_test \
           -e POSTGRES_PASSWORD=quarkus_test -e POSTGRES_DB=quarkus_test \
           -p 5432:5432 postgres:11.2
```

然后, 让我们来配置我们的数据源. `打开src/main/resources/application.properties`, 添加以下内容 :

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=quarkus_test
quarkus.datasource.password=quarkus_test
quarkus.datasource.reactive.url=postgresql://localhost:5432/quarkus_test
myapp.schema.create=true
```

前面的3行定义了数据源. 最后一行将在应用程序中使用, 以表明我们是否在应用程序被初始化时插入一些项目. 

现在, 让我们来创建我们的实体. 创建`org.acme.reactive.crud.Fruit`类, 内容如下 :

```java
package org.acme.reactive.crud;

import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;
import io.vertx.mutiny.pgclient.PgPool;
import io.vertx.mutiny.sqlclient.Row;
import io.vertx.mutiny.sqlclient.RowSet;
import io.vertx.mutiny.sqlclient.Tuple;

import java.util.stream.StreamSupport;

public class Fruit {

    public Long id;

    public String name;

    public Fruit() {
        // default constructor.
    }

    public Fruit(String name) {
        this.name = name;
    }

    public Fruit(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    public static Multi<Fruit> findAll(PgPool client) {
        return client.query("SELECT id, name FROM fruits ORDER BY name ASC").execute()
                // Create a Multi from the set of rows:
                .onItem().transformToMulti(set -> Multi.createFrom().items(() -> StreamSupport.stream(set.spliterator(), false)))
                // For each row create a fruit instance
                .onItem().transform(Fruit::from);
    }

    public static Uni<Fruit> findById(PgPool client, Long id) {
        return client.preparedQuery("SELECT id, name FROM fruits WHERE id = $1").execute(Tuple.of(id))
                .onItem().transform(RowSet::iterator)
                .onItem().transform(iterator -> iterator.hasNext() ? from(iterator.next()) : null);
    }

    public Uni<Long> save(PgPool client) {
        return client.preparedQuery("INSERT INTO fruits (name) VALUES ($1) RETURNING (id)").execute(Tuple.of(name))
                .onItem().transform(pgRowSet -> pgRowSet.iterator().next().getLong("id"));
    }

    public Uni<Boolean> update(PgPool client) {
        return client.preparedQuery("UPDATE fruits SET name = $1 WHERE id = $2").execute(Tuple.of(name, id))
                .onItem().transform(pgRowSet -> pgRowSet.rowCount() == 1);
    }

    public static Uni<Boolean> delete(PgPool client, Long id) {
        return client.preparedQuery("DELETE FROM fruits WHERE id = $1").execute(Tuple.of(id))
                .onItem().transform(pgRowSet -> pgRowSet.rowCount() == 1);
    }

    private static Fruit from(Row row) {
        return new Fruit(row.getLong("id"), row.getString("name"));
    }
}
```

这个实体包含一些字段和方法, 用于从数据库中查找、更新和删除行. 这些方法返回Unis或Multis, 因为当结果被检索到时, 所产生的项目会被异步地返回出来. 注意, quarkus已经封装好了响应式PostgreSQL客户端以兼容Mutiny, 所以你只需将数据库中的结果转化为业务友好的对象. 

为了在应用程序启动时初始化数据库, 我们将创建一个名为`DBInit`的类, 内容如下. 

```java
package org.acme.reactive.crud;

import io.quarkus.runtime.StartupEvent;
import io.vertx.mutiny.pgclient.PgPool;
import org.eclipse.microprofile.config.inject.ConfigProperty;

import javax.enterprise.context.ApplicationScoped;
import javax.enterprise.event.Observes;

@ApplicationScoped
public class DBInit {

    private final PgPool client;
    private final boolean schemaCreate;

    public DBInit(PgPool client, @ConfigProperty(name = "myapp.schema.create", defaultValue = "true") boolean schemaCreate) {
        this.client = client;
        this.schemaCreate = schemaCreate;
    }

    void onStart(@Observes StartupEvent ev) {
        if (schemaCreate) {
            initdb();
        }
    }

    private void initdb() {
        client.query("DROP TABLE IF EXISTS fruits").execute()
                .flatMap(r -> client.query("CREATE TABLE fruits (id SERIAL PRIMARY KEY, name TEXT NOT NULL)").execute())
                .flatMap(r -> client.query("INSERT INTO fruits (name) VALUES ('Kiwi')").execute())
                .flatMap(r -> client.query("INSERT INTO fruits (name) VALUES ('Durian')").execute())
                .flatMap(r -> client.query("INSERT INTO fruits (name) VALUES ('Pomelo')").execute())
                .flatMap(r -> client.query("INSERT INTO fruits (name) VALUES ('Lychee')").execute())
                .await().indefinitely();
    }
}
```

然后, 让我们在`FruitResource`中使用这个`Fruit`类. 编辑`FruitResource`类, 使其与以下内容相匹配 :

```java
package org.acme.reactive.crud;

import java.net.URI;

import javax.ws.rs.Consumes;
import javax.ws.rs.DELETE;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.PUT;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.Response.ResponseBuilder;
import javax.ws.rs.core.Response.Status;

import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;
import io.vertx.mutiny.pgclient.PgPool;

@Path("fruits")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class FruitResource {

    private final PgPool client;

    public FruitResource(PgPool client) {
        this.client = client;
    }

    @GET
    public Multi<Fruit> get() {
        return Fruit.findAll(client);
    }

    @GET
    @Path("{id}")
    public Uni<Response> getSingle(Long id) {
        return Fruit.findById(client, id)
                .onItem().transform(fruit -> fruit != null ? Response.ok(fruit) : Response.status(Status.NOT_FOUND))
                .onItem().transform(ResponseBuilder::build);
    }

    @POST
    public Uni<Response> create(Fruit fruit) {
        return fruit.save(client)
                .onItem().transform(id -> URI.create("/fruits/" + id))
                .onItem().transform(uri -> Response.created(uri).build());
    }

    @PUT
    @Path("{id}")
    public Uni<Response> update(Long id, Fruit fruit) {
        return fruit.update(client)
                .onItem().transform(updated -> updated ? Status.OK : Status.NOT_FOUND)
                .onItem().transform(status -> Response.status(status).build());
    }

    @DELETE
    @Path("{id}")
    public Uni<Response> delete(Long id) {
        return Fruit.delete(client, id)
                .onItem().transform(deleted -> deleted ? Status.NO_CONTENT : Status.NOT_FOUND)
                .onItem().transform(status -> Response.status(status).build());
    }
}
```

该资源根据`Fruit`类产生的结果返回**Uni**和**Multi**实例. 

### 使用Vert.x客户端

前面的例子使用的是Quarkus提供的服务. 同时, 你也可以直接使用Vert.x客户端. 

首先, 确保`quarkus-vertx`扩展已经存在. 如果没有, 通过执行下面的命令激活该扩展 :

```bash
mvn io.quarkus:quarkus-maven-plugin:2.0.2.Final:add-extensions \
    -Dextensions=vertx
```

或者手动添加`quarkus-vertx`到你的依赖项中 :

```xml
<dependency>
	<groupId>io.quarkus</groupId>
	<artifactId>quarkus-vertx</artifactId>
</dependency>
```

这里是一个Mutiny版本的Vert.x API列表. 这个API被分为几个工件, 你可以独立导入. 

| groupId:artifactId                                           | Description                                |
| :----------------------------------------------------------- | :----------------------------------------- |
| `io.smallrye.reactive:smallrye-mutiny-vertx-core`            | Mutiny API for Vert.x Core                 |
| `io.smallrye.reactive:smallrye-mutiny-vertx-mail-client`     | Mutiny API for the Vert.x Mail Client      |
| `io.smallrye.reactive:smallrye-mutiny-vertx-web-client`      | Mutiny API for the Vert.x Web Client       |
| `io.smallrye.reactive:smallrye-mutiny-vertx-mongo-client`    | Mutiny API for the Vert.x Mongo Client     |
| `io.smallrye.reactive:smallrye-mutiny-vertx-redis-client`    | Mutiny API for the Vert.x Redis Client     |
| `io.smallrye.reactive:smallrye-mutiny-vertx-cassandra-client` | Mutiny API for the Vert.x Cassandra Client |
| `io.smallrye.reactive:smallrye-mutiny-vertx-consul-client`   | Mutiny API for the Vert.x Consul Client    |
| `io.smallrye.reactive:smallrye-mutiny-vertx-kafka-client`    | Mutiny API for the Vert.x Kafka Client     |
| `io.smallrye.reactive:smallrye-mutiny-vertx-amqp-client`     | Mutiny API for the Vert.x AMQP Client      |
| `io.smallrye.reactive:smallrye-mutiny-vertx-rabbitmq-client` | Mutiny API for the Vert.x RabbitMQ Client  |

你也可以在http://smallrye.io/smallrye-reactive-utils/apidocs/, 查看可用的API. 

让我们举个例子. 在你的应用程序中添加以下依赖关系 :

```xml
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>smallrye-mutiny-vertx-web-client</artifactId>
</dependency>
```

它提供了Vert.x Web客户端的Mutiny API. 然后, 你可以按以下方式使用网络客户端 :

```java
package org.acme.vertx;

import io.smallrye.mutiny.Uni;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.web.client.WebClientOptions;
import io.vertx.mutiny.core.Vertx;
import io.vertx.mutiny.ext.web.client.WebClient;
import org.jboss.resteasy.annotations.jaxrs.PathParam;

import javax.annotation.PostConstruct;
import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/fruit-data")
public class ResourceUsingWebClient {

    private final WebClient client;

    public ResourceUsingWebClient(Vertx vertx) {
        this.client = WebClient.create(vertx,
                new WebClientOptions().setDefaultHost("fruityvice.com").setDefaultPort(443).setSsl(true)
                        .setTrustAll(true));
    }

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    @Path("/{name}")
    public Uni<JsonObject> getFruitData(@PathParam("name") String name) {
        return client.get("/api/fruit/" + name)
                .send()
                .map(resp -> {
                    if (resp.statusCode() == 200) {
                        return resp.bodyAsJsonObject();
                    } else {
                        return new JsonObject()
                                .put("code", resp.statusCode())
                                .put("message", resp.bodyAsString());
                    }
                });
    }

}
```

有两个重要的点. 

- 注入的Vert.x实例具有`io.vertx.mutiny.core.Vertx`类型, 这是Vert.x的Mutiny变体. 

- Web客户端是由`io.vertx.mutiny.ext.web.client.WebClient.Customer`创建. 

Mutiny版本的Vert.x APIs还提供了. 

- andAwait方法, 如sendAndAwait. andAwait表示调用者线程被阻塞, 直到结果可用. 注意不要用这种方式阻塞事件循环/IO线程. 

- andForget方法, 如writeAndForget. andForget适用于返回Uni的方法. andForget表明你不需要表示操作成功或失败的结果Uni. 然而, 请记住, 如果你不订阅, 操作就不会被触发. andForget为你管理这个, 并管理订阅. 

- toMulti方法允许将Vert.x `ReadStream`转化为Multi. 

- toBlockingIterable / toBlockingStream方法允许将Vert.x ReadStream转换为一个阻塞迭代的或阻塞的java.util.Stream. 

### 使用RxJava或Reactor APIs

Mutiny提供了将`RxJava 2`和`Project Reactor`类型转换为Uni和Multi的实用程序. 

RxJava 2转换器在以下依赖性中可用 :

```xml
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>mutiny-rxjava</artifactId>
</dependency>
```

因此, 如果你有一个返回RxJava 2类型 (Completable, Single, Maybe, Observable, Flowable)的API, 你可以按以下方式创建Unis和Multis :

```java
mport io.smallrye.mutiny.converters.multi.MultiRxConverters;
import io.smallrye.mutiny.converters.uni.UniRxConverters;
// ...
Uni<Void> uniFromCompletable = Uni.createFrom().converter(UniRxConverters.fromCompletable(), completable);
Uni<String> uniFromSingle = Uni.createFrom().converter(UniRxConverters.fromSingle(), single);
Uni<String> uniFromMaybe = Uni.createFrom().converter(UniRxConverters.fromMaybe(), maybe);
Uni<String> uniFromEmptyMaybe = Uni.createFrom().converter(UniRxConverters.fromMaybe(), emptyMaybe);
Uni<String> uniFromObservable = Uni.createFrom().converter(UniRxConverters.fromObservable(), observable);
Uni<String> uniFromFlowable = Uni.createFrom().converter(UniRxConverters.fromFlowable(), flowable);

Multi<Void> multiFromCompletable = Multi.createFrom().converter(MultiRxConverters.fromCompletable(), completable);
Multi<String> multiFromSingle = Multi.createFrom().converter(MultiRxConverters.fromSingle(), single);
Multi<String> multiFromMaybe = Multi.createFrom().converter(MultiRxConverters.fromMaybe(), maybe);
Multi<String> multiFromEmptyMaybe = Multi.createFrom().converter(MultiRxConverters.fromMaybe(), emptyMaybe);
Multi<String> multiFromObservable = Multi.createFrom().converter(MultiRxConverters.fromObservable(), observable);
Multi<String> multiFromFlowable = Multi.createFrom().converter(MultiRxConverters.fromFlowable(), flowable);
```

你也可以将Unis和Multis转化为RxJava类型 :

```java
Completable completable = uni.convert().with(UniRxConverters.toCompletable());
Single<Optional<String>> single = uni.convert().with(UniRxConverters.toSingle());
Single<String> single2 = uni.convert().with(UniRxConverters.toSingle().failOnNull());
Maybe<String> maybe = uni.convert().with(UniRxConverters.toMaybe());
Observable<String> observable = uni.convert().with(UniRxConverters.toObservable());
Flowable<String> flowable = uni.convert().with(UniRxConverters.toFlowable());
// ...
Completable completable = multi.convert().with(MultiRxConverters.toCompletable());
Single<Optional<String>> single = multi.convert().with(MultiRxConverters.toSingle());
Single<String> single2 = multi.convert().with(MultiRxConverters
        .toSingle().onEmptyThrow(() -> new Exception("D'oh!")));
Maybe<String> maybe = multi.convert().with(MultiRxConverters.toMaybe());
Observable<String> observable = multi.convert().with(MultiRxConverters.toObservable());
Flowable<String> flowable = multi.convert().with(MultiRxConverters.toFlowable());
```

Project Reactor转换器在以下依赖性中可用 :

```xml
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>mutiny-reactor</artifactId>
</dependency>
```

因此, 如果你有一个返回Reactor类型的API (Mono, Flux), 你可以按以下方式创建Unis和Multis :

```java
import io.smallrye.mutiny.converters.multi.MultiReactorConverters;
import io.smallrye.mutiny.converters.uni.UniReactorConverters;
// ...
Uni<String> uniFromMono = Uni.createFrom().converter(UniReactorConverters.fromMono(), mono);
Uni<String> uniFromFlux = Uni.createFrom().converter(UniReactorConverters.fromFlux(), flux);

Multi<String> multiFromMono = Multi.createFrom().converter(MultiReactorConverters.fromMono(), mono);
Multi<String> multiFromFlux = Multi.createFrom().converter(MultiReactorConverters.fromFlux(), flux);
```

你也可以将Unis和Multis转化为反应器类型 :

```java
Mono<String> mono = uni.convert().with(UniReactorConverters.toMono());
Flux<String> flux = uni.convert().with(UniReactorConverters.toFlux());

Mono<String> mono2 = multi.convert().with(MultiReactorConverters.toMono());
Flux<String> flux2 = multi.convert().with(MultiReactorConverters.toFlux());
```

### 使用 CompletionStages 或 Publisher API

如果你面对一个使用CompletionStage、CompletableFuture或Publisher的API, 你可以来回转换. 首先, Uni和Multi都可以从一个CompletionStage或从一个Supplier< CompletionStage >创建. 比如说 :

```java
CompletableFuture<String> future = Uni
        // Create from a Completion Stage
        .createFrom().completionStage(CompletableFuture.supplyAsync(() -> "hello"));
```

在Uni上, 你也可以使用`subscribeAsCompletionStage()`产生一个CompletionStage, 它将获得Uni所发出的item或失败. 

你还可以使用`createFrom().publisher(Publisher)`从Publisher的实例中创建Unis和Multis. 你可以使用toMulti将Uni转换为Publisher. 事实上, Multi 实现了 Publisher. 

## 总结

📢 : 如果你已经看到这里了, 说明你已经学会1+1=2了, 接下来可以去解微分方程了👏

简单归纳一下这篇教程的重点 :

1. 什么是响应式编程 ?
2. 怎么使用Mutiny这个库 ?

quarkus厉害就厉害在它官方原生统一了命令式编程和响应式编程, 并且所有的响应式API都会统一返回一个Uni或者Multi, 因此你只要学会怎么用Mutiny即可. 

## What’s next?

本教程是对Quarkus中的reactive的介绍. 有很多Quarkus的功能已经是响应式的了. 下面的列表给你几个例子 :

- [Using Mutiny with RestEasy](https://quarkus.io/guides/rest-json#reactive)
- [Sending email](https://quarkus.io/guides/mailer)
- [Using MongoDB](https://quarkus.io/guides/mongodb#reactive) and [MongoDB with Panache](https://quarkus.io/guides/mongodb-panache#reactive)
- [Reactive Database Clients](https://quarkus.io/guides/reactive-sql-clients)
- [Using Vert.x](https://quarkus.io/guides/vertx)
- [Interacting with Kafka](https://quarkus.io/guides/kafka) and [Interacting with AMQP](https://quarkus.io/guides/amqp)
- [Using Neo4J](https://quarkus.io/guides/neo4j#reactive)
- [Using reactive routes](https://quarkus.io/guides/reactive-routes)