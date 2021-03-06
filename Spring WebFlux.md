# Web on Reactive Stack

版本5.3.1



文档的此部分涵盖对基于[响应流](https://www.reactive-streams.org/) API构建的[reactive-stack](https://www.reactive-streams.org/)Web应用程序的支持，该应用程序可在非阻塞服务器（例如Netty，Undertow和Servlet 3.1+容器）上运行。涵盖了[Spring WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/webflux.html#webflux)框架，Reactive[`WebClient`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client)，[测试](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-test)支持和[Reactive库](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)的独立章节。对于Servlet栈Web应用程序，请参阅[Servlet Stack上的Web](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#spring-web)。

## 1. Spring WebFlux

Spring框架中包含的原始Web框架Spring Web MVC是专门为Servlet API和Servlet容器而构建的。响应式堆栈Web框架Spring WebFlux在稍后的5.0版中添加。它是完全无阻塞的，支持 [响应流](https://www.reactive-streams.org/)背压，并在Netty，Undertow和Servlet 3.1+容器等服务器上运行。

这两个Web框架都反映了其源模块的名称（[spring-webmvc](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc)和 [spring-webflux](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux)），并在Spring Framework中并存。每个模块都是可选的。应用程序可以使用一个模块，也可以使用另一个模块，在某些情况下，也可以使用两个模块，例如，带有react的Spring MVC控制器`WebClient`。

### 1.1。总览

为什么创建Spring WebFlux？

答案的一部分是需要一个非阻塞式的Web堆栈使用少量线程处理并发，并使用更少的硬件资源进行扩展。Servlet 3.1确实提供了用于非阻塞I / O的API。然而，使用它从在Servlet API的其余部分，其中，合同是同步的（导致远`Filter`，`Servlet`）或阻断（`getParameter`， `getPart`）。这是促使新的通用API成为所有非阻塞运行时的基础的动机。这很重要，因为服务器（例如Netty）在异步，非阻塞空间中已建立良好。

答案的另一部分是函数式编程。就像在Java 5中添加注释会创造机会（例如带注释的REST控制器或单元测试）一样，在Java 8中添加lambda表达式也会为Java中的功能API创造机会。这对于无阻塞的应用程序和继续式API（如[ReactiveX](http://reactivex.io/)`CompletableFuture`和[ReactiveX](http://reactivex.io/)所普及）是一个福音，这些API允许以声明的方式构造异步逻辑。在编程模型级别，Java 8使Spring WebFlux能够与带注释的控制器一起提供函数式的Web端点。

#### 1.1.1。定义“Reactive”

我们谈到了“非阻塞”和“函数式”，但是Reactive意味着什么？

术语“Reactive”是指围绕响应变化——网络组件响应I / O事件，UI控制器响应鼠标事件等——而构建的编程模型。从这个意义上说，非阻塞是Reactive的，因为当操作完成或数据可用，我们现在处于响应通知的模式中，而不是被阻塞。

我们Spring团队还有另一个重要机制与“Reactive”相关联，这是不阻碍背压的机制。在同步命令式代码中，阻塞调用是背压的自然形式，它迫使调用者等待。在非阻塞代码中，控制事件的速率非常重要，这样快速的生产者就不会淹没其目的地。

响应流是一个 [小的规范](https://github.com/reactive-streams/reactive-streams-jvm/blob/master/README.md#specification) （在Java 9中也[采用](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html)了），它定义了带有反压力的异步组件之间的交互。例如，数据存储库（充当 [Publisher](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Publisher.html)）可以生成HTTP服务器（充当 [Subscriber](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Subscriber.html)）然后可以写入响应的数据。响应流的主要目的是让订阅者控制发布者生成数据的速度或速度。

|      | **常见问题：如果出版商不能放慢脚步怎么办？** 反应流的目的仅仅是建立机制和边界。如果发布者无法放慢速度，则必须决定是缓冲，删除还是失败。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.1.2。响应式API

响应流对于互操作性起着重要作用。库和基础结构组件对此很感兴趣，但由于它太底层了，因此它不适合用作应用程序API。应用程序需要更高级别且功能更丰富的API来构成异步逻辑-与Java 8 `Stream`API类似，但不仅适用于集合。这就是Reactive库发挥的作用。

[Reactor](https://github.com/reactor/reactor)是Spring WebFlux的首选响应库。它提供了 [`Mono`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)和 [`Flux`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)API类型，以通过与ReactiveX[运算符词汇](http://reactivex.io/documentation/operators.html)对齐的丰富运算符集来处理0..1（`Mono`）和0..N（`Flux`）数据序列。Reactor是响应流库，因此，它的所有运算符都支持无阻塞背压。Reactor非常注重服务器端Java。它是与Spring紧密合作开发的。

WebFlux要求Reactor作为核心依赖项，但是它可以通过响应流与其他React库进行互操作。通常，WebFlux API接受平原`Publisher` 作为输入，在内部将其适应于Reactor类型，使用它，然后返回a `Flux`或a`Mono`作为输出。因此，您可以将任何值`Publisher`作为输入传递，并且可以对输出应用操作，但是您需要调整输出以与另一个Reactive库一起使用。只要可行（例如，带注释的控制器），WebFlux就会透明地适应RxJava或其他Reactive库的使用。有关更多详细信息，请参见[Reactive库](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)。

|      | 除了响应式API外，WebFlux还可以与Kotlin中的[Coroutines](https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#coroutines) API一起使用， 从而提供了更强制的编程风格。以下Kotlin代码示例将随Coroutines API一起提供。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.1.3。编程模型

`spring-web`模块包含Spring WebFlux基础的Reactive基础，包括HTTP抽象，用于支持的服务器的响应流[适配器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler)，[编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)以及与Servlet API相似但具有非阻塞合同的核心[`WebHandler`API](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)。

在此基础上，Spring WebFlux提供了两种编程模型的选择：

- [带注释的控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-controller)：与Spring MVC一致，并基于`spring-web`模块中的相同注释。Spring MVC和WebFlux控制器都支持Reactive（Reactor和RxJava）返回类型，因此，区分它们并不容易。一个显着的区别是WebFlux还支持响应`@RequestBody`参数。
- [函数式端点](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn)：基于Lambda的轻量级功能编程模型。您可以将其视为一个小型库或一组实用程序，应用程序可以使用它们来路由和处理请求。带注释的控制器的最大区别在于，应用程序负责从头到尾的请求处理，而不是通过注释声明意图并被回调。

#### 1.1.4。适用性

Spring MVC还是WebFlux？

一个自然的问题要问，但建立了不合理的二分法。实际上，两者共同努力扩大了可用选项的范围。两者的设计旨在实现彼此的连续性和一致性，它们可以并行使用，并且来自双方的反馈对双方都有利。下图显示了两者之间的关系，它们的共同点以及各自的独特支持：

![Spring MVC和Webflux Venn](https://docs.spring.io/spring-framework/docs/current/reference/html/images/spring-mvc-and-webflux-venn.png)

我们建议您考虑以下几点：

- 如果您有运行正常的Spring MVC应用程序，则无需更改。命令式编程是编写，理解和调试代码的最简单方法。您有最大的库选择空间，因为从历史上看，大多数库都处于阻塞状态。
- 如果您已经在购买无阻塞的Web堆栈，Spring WebFlux可以提供与该领域其他服务器相同的执行模型优势，还可以选择服务器（Netty，Tomcat，Jetty，Undertow和Servlet 3.1+容器），选择编程模型（带注释的控制器和功能性Web端点），以及选择Reactive库（Reactor，RxJava或其他）。
- 如果您对与Java 8 lambda或Kotlin一起使用的轻量级功能性Web框架感兴趣，则可以使用Spring WebFlux功能性Web端点。对于要求较低复杂性的较小应用程序或微服务（可以受益于更高的透明度和控制）而言，这也是一个不错的选择。
- 在微服务架构中，您可以混合使用带有Spring MVC或Spring WebFlux控制器或带有Spring WebFlux功能端点的应用程序。在两个框架中都支持相同的基于注释的编程模型，这使得重用知识变得更加容易，同时还为正确的工作选择了正确的工具。
- 评估应用程序的一种简单方法是检查其依赖关系。如果您要使用阻塞性持久性API（JPA，JDBC）或网络API，则Spring MVC至少是常见体系结构的最佳选择。使用Reactor和RxJava在单独的线程上执行阻塞调用在技术上是可行的，但您不会充分利用非阻塞Web堆栈。
- 如果您的Spring MVC应用程序具有对远程服务的调用，请尝试使用active `WebClient`。您可以直接从Spring MVC控制器方法返回响应类型（Reactor，RxJava[或其他](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)）。每个呼叫的等待时间或呼叫之间的相互依赖性越大，好处就越明显。Spring MVC控制器也可以调用其他Reactive组件。
- 如果您有庞大的团队，请牢记向无阻塞，功能性和声明性编程的过渡过程中的学习曲线很陡。在没有完全切换的情况下启动的一种实用方法是使用电抗器`WebClient`。除此之外，从小处着手并衡量收益。我们希望对于广泛的应用而言，这种转变是不必要的。如果不确定要寻找什么好处，请先了解无阻塞I / O的工作原理（例如，单线程Node.js上的并发性）及其影响。

#### 1.1.5。伺服器

Tomcat，Jetty，Servlet 3.1+容器以及Netty和Undertow等非Servlet运行时均支持Spring WebFlux。所有服务器都适应于低级 [通用API，](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler)因此可以跨服务器支持更高级别的 [编程模型](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-programming-models)。

Spring WebFlux不具有内置支持来启动或停止服务器。但是，从Spring配置和[WebFlux基础](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)结构[组装](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)应用程序 并用几行代码[运行它](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler)很容易。

Spring Boot具有一个WebFlux启动器，可以自动执行这些步骤。默认情况下，启动程序使用Netty，但通过更改Maven或Gradle依赖关系，可以轻松切换到Tomcat，Jetty或Undertow。Spring Boot默认为Netty，因为它在异步，非阻塞空间中得到更广泛的使用，并允许客户端和服务器共享资源。

Tomcat和Jetty可以与Spring MVC和WebFlux一起使用。但是请记住，它们的使用方式非常不同。Spring MVC依靠Servlet阻塞I / O，并允许应用程序在需要时直接使用Servlet API。Spring WebFlux依赖Servlet 3.1非阻塞I / O，并在低级适配器后面使用Servlet API。请勿将其直接使用。

对于Undertow，Spring WebFlux直接使用Undertow API，而无需使用Servlet API。

#### 1.1.6。性能

表演具有许多特征和意义。响应和非阻塞通常不会使应用程序运行得更快。在某些情况下，它们可以（例如，如果使用 `WebClient`并行运行远程调用）。总体而言，以非阻塞方式进行处理需要更多的工作，这可能会稍微增加所需的处理时间。

Reactive和非阻塞性的主要预期好处是能够以较少的固定数量的线程和较少的内存进行扩展。这使应用程序在负载下更具弹性，因为它们以更可预测的方式扩展。但是，为了观察这些好处，您需要有一些延迟（包括缓慢的和不可预测的网络I / O的混合）。这就是reactive-stack开始显示其优势的地方，差异可能很大。

#### 1.1.7。并发模型

Spring MVC和Spring WebFlux都支持带注释的控制器，但是并发模型和默认的阻塞和线程假设存在关键差异。

在Spring MVC（通常是Servlet应用程序）中，假定应用程序可以阻塞当前线程（例如，用于远程调用）。因此，Servlet容器使用大线程池来吸收请求处理期间的潜在阻塞。

在Spring WebFlux（通常是非阻塞服务器）中，假定应用程序未阻塞。因此，非阻塞服务器使用固定大小的小型线程池（事件循环工作器）来处理请求。

|      | “按比例缩放”和“少量线程”听起来可能是矛盾的，但是从不阻塞当前线程（而是依靠回调）意味着您不需要额外的线程，因为没有阻塞调用可以吸收。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

调用阻止API

如果确实需要使用阻止库怎么办？Reactor和RxJava都为`publishOn`操作员提供了 在不同线程上继续进行处理的能力。这意味着容易逃生。但是请记住，阻塞式API不适用于此并发模型。

可变状态

在Reactor和RxJava中，您可以通过运算符声明逻辑。在运行时，会形成一个Reactive管道，其中在不同的阶段依次处理数据。这样做的主要好处是，它使应用程序不必保护可变状态，因为该管道中的应用程序代码永远不会被并发调用。

线程模型

您期望在运行Spring WebFlux的服务器上看到哪些线程？

- 在“原始” Spring WebFlux服务器上（例如，没有数据访问权限或其他可选依赖项），您可以期望该服务器有一个线程，而其他几个线程则可以进行请求处理（通常与CPU核心数量一样多）。但是，Servlet容器可能以更多线程开始（例如，Tomcat上为10），以支持Servlet（阻塞）I / O和Servlet 3.1（非阻塞）I / O使用。
- 响应`WebClient`式以事件循环方式运行。因此，您可以看到与之相关的固定数量的处理线程（例如，`reactor-http-nio-`使用Reactor Netty连接器）。但是，如果客户端和服务器都使用Reactor Netty，则默认情况下两者共享事件循环资源。
- Reactor和RxJava提供了称为调度程序的线程池抽象，以与`publishOn`用于将处理切换到另一个线程池的运算符一起使用 。调度程序具有建议特定并发策略的名称，例如，“并行”（用于有限数量的线程的CPU绑定工作）或“弹性”（用于具有大量线程的I / O绑定）。如果看到这样的线程，则意味着某些代码正在使用特定的线程池`Scheduler`策略。
- 数据访问库和其他第三方依赖性也可以创建和使用自己的线程。

配置中

Spring框架不提供启动和停止[服务器的](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-server-choice)支持 。要为服务器配置线程模型，您需要使用服务器特定的配置API，或者，如果您使用的是Spring Boot，请检查每个服务器的Spring Boot配置选项。您可以 [配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client-builder)的`WebClient`直接。对于所有其他库，请参阅其各自的文档。

### 1.2。reactive-stack芯

该`spring-web`模块包含以下对响应式Web应用程序的基础支持：

- 对于服务器请求处理，有两个级别的支持。
  - [HttpHandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler)：HTTP请求处理的基本协议，具有无阻塞I / O和响应流背压，以及Reactor Netty，Undertow，Tomcat，Jetty和任何Servlet 3.1+容器的适配器。
  - [`WebHandler`API](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)：稍高级别的通用Web API，用于处理请求，在此之上构建了具体的编程模型，例如带注释的控制器和功能端点。
- 对于客户端，有一个基本`ClientHttpConnector`协议来执行具有无阻塞I / O和响应流反压力的HTTP请求，以及用于[Reactor Netty](https://github.com/reactor/reactor-netty)，响应式 [Jetty HttpClient](https://github.com/jetty-project/jetty-reactive-httpclient) 和[Apache HttpComponents的](https://hc.apache.org/)适配器 。应用程序中使用的更高级别的[WebClient](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client)基于此基本协定。
- 对于客户端和服务器，[编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)用于HTTP请求和响应内容的序列化和反序列化。

#### 1.2.1。 `HttpHandler`

[HttpHandler](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/server/reactive/HttpHandler.html) 是具有一个用于处理请求和响应的单一方法的简单协定。有意最小化这个协定，其主要且唯一的目的是成为对不同HTTP服务器API的最小化抽象。

下表描述了受支持的服务器API：

| 服务器名称      | 使用的服务器API                                              | Reactive流支持                                               |
| :-------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 净额            | Netty API                                                    | [reactive-stack净值](https://github.com/reactor/reactor-netty) |
| 底拖            | Undertow API                                                 | spring-web：向响应流桥过渡                                   |
| 雄猫            | Servlet 3.1非阻塞I / O；读取和写入ByteBuffers与byte []的Tomcat API | spring-web：Servlet 3.1非阻塞I / O到响应流桥                 |
| 码头            | Servlet 3.1非阻塞I / O；Jetty API编写ByteBuffers与byte []    | spring-web：Servlet 3.1非阻塞I / O到响应流桥                 |
| Servlet 3.1容器 | Servlet 3.1非阻塞I / O                                       | spring-web：Servlet 3.1非阻塞I / O到响应流桥                 |

下表描述了服务器依赖性（另请参阅 [受支持的版本](https://github.com/spring-projects/spring-framework/wiki/What's-New-in-the-Spring-Framework)）：

| 服务器名称         | 群组编号                | 工件名称               |
| :----------------- | :---------------------- | :--------------------- |
| reactive-stack净值 | io.projectreactor.netty | reactive-stack净额     |
| 底拖               | io.toow                 | 底核                   |
| 雄猫               | org.apache.tomcat.embed | Tomcat嵌入式核心       |
| 码头               | org.eclipse.jetty       | 码头服务器，码头服务器 |

下面的代码段显示了如何将`HttpHandler`适配器与每个服务器API一起使用：

**reactive-stack净值**

爪哇

科特林

```java
HttpHandler handler = ...
ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(handler);
HttpServer.create().host(host).port(port).handle(adapter).bind().block();
```

**底拖**

爪哇

科特林

```java
HttpHandler handler = ...
UndertowHttpHandlerAdapter adapter = new UndertowHttpHandlerAdapter(handler);
Undertow server = Undertow.builder().addHttpListener(port, host).setHandler(adapter).build();
server.start();
```

**雄猫**

爪哇

科特林

```java
HttpHandler handler = ...
Servlet servlet = new TomcatHttpHandlerAdapter(handler);

Tomcat server = new Tomcat();
File base = new File(System.getProperty("java.io.tmpdir"));
Context rootContext = server.addContext("", base.getAbsolutePath());
Tomcat.addServlet(rootContext, "main", servlet);
rootContext.addServletMappingDecoded("/", "main");
server.setHost(host);
server.setPort(port);
server.start();
```

**码头**

爪哇

科特林

```java
HttpHandler handler = ...
Servlet servlet = new JettyHttpHandlerAdapter(handler);

Server server = new Server();
ServletContextHandler contextHandler = new ServletContextHandler(server, "");
contextHandler.addServlet(new ServletHolder(servlet), "/");
contextHandler.start();

ServerConnector connector = new ServerConnector(server);
connector.setHost(host);
connector.setPort(port);
server.addConnector(connector);
server.start();
```

**Servlet 3.1+容器**

要将其作为WAR部署到任何Servlet 3.1+容器，您可以扩展并包含 [`AbstractReactiveWebInitializer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/server/adapter/AbstractReactiveWebInitializer.html) 在WAR中。该类包装`HttpHandler`with`ServletHttpHandlerAdapter`并将其注册为`Servlet`。

#### 1.2.2。`WebHandler`API

该`org.springframework.web.server`软件包基于[`HttpHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler)合同提供了通用Web API，用于通过多个[`WebExceptionHandler`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/server/WebExceptionHandler.html)，多个 [`WebFilter`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/server/WebFilter.html)和单个 [`WebHandler`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/server/WebHandler.html)组件链处理请求 。可以`WebHttpHandlerBuilder`通过简单地指向[自动检测](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api-special-beans)`ApplicationContext`组件 的Spring 和/或通过向构建器注册组件来将链组合在一起。

尽管`HttpHandler`有一个简单的目标来抽象使用不同的HTTP服务器，但该 `WebHandler`API的目的是提供Web应用程序中通常使用的更广泛的功能集，例如：

- 具有属性的用户会话。
- 请求属性。
- 已解决`Locale`或`Principal`针对请求。
- 访问已解析和缓存的表单数据。
- 多部分数据的抽象。
- 和更多..

##### 特殊bean类

下表列出了`WebHttpHandlerBuilder`可以在Spring ApplicationContext中自动检测的组件，或可以直接向其注册的组件：

| bean名                       | bean类                       | 计数 | 描述                                                         |
| :--------------------------- | :--------------------------- | :--- | :----------------------------------------------------------- |
| <任何>                       | `WebExceptionHandler`        | 0..N | 提供`WebFilter`实例链和目标 链中的异常处理`WebHandler`。有关更多详细信息，请参见[Exceptions](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-exception-handler)。 |
| <任何>                       | `WebFilter`                  | 0..N | 在其余过滤链和目标之前和之后应用拦截式逻辑`WebHandler`。有关更多详细信息，请参见过[滤器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-filters)。 |
| `webHandler`                 | `WebHandler`                 | 1个  | 请求的处理程序。                                             |
| `webSessionManager`          | `WebSessionManager`          | 0..1 | `WebSession`通过上的方法公开的实例的管理器`ServerWebExchange`。 `DefaultWebSessionManager`默认。 |
| `serverCodecConfigurer`      | `ServerCodecConfigurer`      | 0..1 | 用于访问`HttpMessageReader`实例以解析表单数据和多部分数据，然后通过上的方法公开这些数据`ServerWebExchange`。`ServerCodecConfigurer.create()`默认。 |
| `localeContextResolver`      | `LocaleContextResolver`      | 0..1 | `LocaleContext`通过在上的方法公开的解析器`ServerWebExchange`。 `AcceptHeaderLocaleContextResolver`默认。 |
| `forwardedHeaderTransformer` | `ForwardedHeaderTransformer` | 0..1 | 对于处理转发的类型标头，可以通过提取和删除它们或仅通过删除它们来进行。默认不使用。 |

##### 表格数据

`ServerWebExchange` 公开以下访问表单数据的方法：

爪哇

科特林

```java
Mono<MultiValueMap<String, String>> getFormData();
```

的，`DefaultServerWebExchange`用于将`HttpMessageReader`表格数据（`application/x-www-form-urlencoded`）解析为`MultiValueMap`。默认情况下， `FormHttpMessageReader`配置为供`ServerCodecConfigurer`Bean使用（请参阅[Web Handler API](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)）。

##### 多部分数据

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart)

`ServerWebExchange` 公开以下访问多部分数据的方法：

爪哇

科特林

```java
Mono<MultiValueMap<String, Part>> getMultipartData();
```

`DefaultServerWebExchange`使用配置的 `HttpMessageReader<MultiValueMap<String, Part>>`解析`multipart/form-data`内容成`MultiValueMap`。默认情况下，这是`DefaultPartHttpMessageReader`，没有任何第三方依赖项。或者，`SynchronossPartHttpMessageReader`可以使用基于 [Synchronoss NIO Multipart](https://github.com/synchronoss/nio-multipart)库的。两者都是通过`ServerCodecConfigurer`bean配置的（请参阅[Web Handler API](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)）。

要以流方式解析多部分数据，可以使用`Flux<Part>`从中返回的内容 `HttpMessageReader<Part>`。例如，在带注释的控制器中，使用 `@RequestPart`隐含`Map`名称之类的名称可以访问各个部分，因此需要完全解析多部分数据。相反，您可以使用`@RequestBody`解码内容`Flux<Part>`而不收集到`MultiValueMap`。

##### 转发的标题

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-forwarded-headers)

当请求通过代理（例如负载平衡器）处理时，主机，端口和方案可能会更改。从客户端的角度来看，这给创建指向正确主机，端口和方案的链接带来了挑战。

[RFC 7239](https://tools.ietf.org/html/rfc7239)定义了`Forwarded`代理可用来提供有关原始请求的信息的HTTP标头。还有其他一些非标头，也包括`X-Forwarded-Host`，`X-Forwarded-Port`， `X-Forwarded-Proto`，`X-Forwarded-Ssl`，和`X-Forwarded-Prefix`。

`ForwardedHeaderTransformer`是一个组件，可根据转发的标头修改请求的主机，端口和方案，然后删除这些标头。如果将其声明为具有名称的Bean `forwardedHeaderTransformer`，它将被 [检测](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api-special-beans)并使用。

对于转发的标头，存在安全方面的考虑，因为应用程序无法知道标头是由代理添加的，还是由恶意客户端添加的。这就是为什么应配置信任边界处的代理以删除来自外部的不受信任的转发流量的原因。您还可以配置`ForwardedHeaderTransformer`with `removeOnly=true`，在这种情况下，它会删除但不使用标题。

|      | 在5.1`ForwardedHeaderFilter`中已弃用并取代了该`ForwardedHeaderTransformer`标题， 因此可以在创建交换之前更早地处理转发的标头。如果仍然配置了过滤器，则将其从过滤器列表中删除，并`ForwardedHeaderTransformer`改为使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.2.3。筛选器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters)

在[`WebHandler`API中](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)，您可以`WebFilter`在过滤器和target的其余处理链的前后使用来应用拦截样式的逻辑 `WebHandler`。使用[WebFlux Config时](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)，注册a`WebFilter`就像将其声明为Spring bean一样简单，并且（可选）通过`@Order`在bean声明上使用或实现来表达优先级`Ordered`。

##### CORS

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-cors)

Spring WebFlux通过控制器上的注释为CORS配置提供了细粒度的支持。但是，当将它与Spring Security结合使用时，我们建议您依赖内置的 `CorsFilter`，必须在Spring Security的过滤器链之前订购它们。

有关更多详细信息，请参见[CORS](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-cors)和[webflux-cors.html](https://docs.spring.io/spring-framework/docs/current/reference/html/webflux-cors.html#webflux-cors-webfilter)上的部分。

#### 1.2.4。例外情况

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-customer-servlet-container-error-page)

在[`WebHandler`API中](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)，您可以使用A`WebExceptionHandler`处理`WebFilter`实例链和目标链中的异常`WebHandler`。使用 [WebFlux Config时](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)，注册a`WebExceptionHandler`就像将其声明为Spring bean一样简单，并且（可选）通过`@Order`在bean声明上使用或实现来表达优先级`Ordered`。

下表描述了可用的`WebExceptionHandler`实现：

| 异常处理程序                            | 描述                                                         |
| :-------------------------------------- | :----------------------------------------------------------- |
| `ResponseStatusExceptionHandler`        | [`ResponseStatusException`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/server/ResponseStatusException.html) 通过将响应设置为异常的HTTP状态代码，提供对类型异常的处理 。 |
| `WebFluxResponseStatusExceptionHandler` | 扩展名`ResponseStatusExceptionHandler`还可以确定`@ResponseStatus`任何异常的注释的HTTP状态代码。该处理程序在[WebFlux Config中](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)声明。 |

#### 1.2.5。编解码器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-message-conversion)

的`spring-web`和`spring-core`模块通过与Reactive流背压非阻塞I / O用于序列化和反序列化字节内容和从更高级别的对象提供支持。以下介绍了此支持：

- [`Encoder`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/codec/Encoder.html)并且 [`Decoder`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/codec/Decoder.html)是与HTTP无关的编码和解码内容的低级合同。
- [`HttpMessageReader`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/codec/HttpMessageReader.html)并且 [`HttpMessageWriter`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/codec/HttpMessageWriter.html)是对HTTP消息内容进行编码和解码的合同。
- 一个`Encoder`可以被包装`EncoderHttpMessageWriter`，以适应其在Web应用程序使用，而`Decoder`可与包裹`DecoderHttpMessageReader`。
- [`DataBuffer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/io/buffer/DataBuffer.html)抽象不同字节的缓冲区表示（如Netty的`ByteBuf`，`java.nio.ByteBuffer`等等），并是所有的编解码器上工作。有关此主题的更多信息，请参见“ Spring核心”部分中的[数据缓冲区和编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers)。

该`spring-core`模块提供`byte[]`，`ByteBuffer`，`DataBuffer`，`Resource`，和 `String`编码器和解码器实现。该`spring-web`模块提供了Jackson JSON，Jackson Smile，JAXB2，协议缓冲区和其他编码器和解码器，以及用于表单数据，多部分内容，服务器发送的事件等的纯Web HTTP消息读取器和写入器实现。

`ClientCodecConfigurer`并且`ServerCodecConfigurer`通常用于配置和自定义编解码器以在应用程序中使用。请参阅有关配置[HTTP消息编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-message-codecs)的部分 。

##### 杰克逊JSON

当存在Jackson库时，都支持JSON和二进制JSON（[Smile](https://github.com/FasterXML/smile-format-specification)）。

该`Jackson2Decoder`工作原理如下：

- Jackson的异步，非阻塞解析器用于将字节块流聚合到中`TokenBuffer`，每个字节块代表一个JSON对象。
- 每个`TokenBuffer`都传递给Jackson`ObjectMapper`来创建更高级别的对象。
- 当解码为单值发布者（例如`Mono`）时，有个`TokenBuffer`。
- 当解码为多值发布者（例如`Flux`）时，一旦接收到足够的字节以形成一个完整的对象，它们`TokenBuffer`就会传递到`ObjectMapper`。输入内容可以是JSON数组，也可以是任何以 [行分隔的JSON](https://en.wikipedia.org/wiki/JSON_streaming)格式，例如NDJSON，JSON Lines或JSON Text Sequences。

该`Jackson2Encoder`工作原理如下：

- 对于单个价值发布者（例如`Mono`），只需将其序列化即可 `ObjectMapper`。
- 对于具有的多值发布者`application/json`，默认情况下使用收集值， `Flux#collectToList()`然后序列化结果集合。
- 对于具有流媒体类型（例如`application/x-ndjson`或）的多值发布者，请 `application/stream+x-jackson-smile`使用[行定界的JSON](https://en.wikipedia.org/wiki/JSON_streaming)格式分别对每个值进行编码，写入和刷新 。其他流媒体类型可以在编码器中注册。
- 对于SSE，将`Jackson2Encoder`在每个事件中调用，并刷新输出以确保交付没有延迟。

|      | 默认情况下，两者`Jackson2Encoder`和`Jackson2Decoder`都不支持type的元素 `String`。取而代之的是，默认假设是一个字符串或一系列字符串表示要由呈现的序列化JSON内容`CharSequenceEncoder`。如果您需要从中呈现JSON数组`Flux<String>`，请使用`Flux#collectToList()`和编码`Mono<List<String>>`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 表格数据

`FormHttpMessageReader`并`FormHttpMessageWriter`支持对`application/x-www-form-urlencoded`内容进行解码和编码 。

在经常需要从多个位置访问表单内容的服务器端， `ServerWebExchange`提供了一种专用`getFormData()`方法，该方法可以解析内容`FormHttpMessageReader`，然后缓存结果以进行重复访问。请参阅[API](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)部分中的[表单数据](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-form-data)。[`WebHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)

一旦`getFormData()`使用了原始内容，就无法再从请求正文中读取原始内容。因此，`ServerWebExchange` 与从原始请求主体读取数据相比，期望应用程序能够始终访问访问缓存的表单数据。

##### 多部分

`MultipartHttpMessageReader`并`MultipartHttpMessageWriter`支持对“多部分/表单数据”内容进行解码和编码。反过来，`MultipartHttpMessageReader`委派给另一个人`HttpMessageReader`，将其实际解析为a `Flux<Part>`，然后将这些部分简单地收集到a中`MultiValueMap`。默认情况下，`DefaultPartHttpMessageReader`使用，但是可以通过进行更改 `ServerCodecConfigurer`。有关的更多信息`DefaultPartHttpMessageReader`，请参考的 [javadoc`DefaultPartHttpMessageReader`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/codec/multipart/DefaultPartHttpMessageReader.html)。

在可能需要从多个位置访问多部分表单内容的服务器端，`ServerWebExchange`提供了一种专用`getMultipartData()`方法，该方法可以解析内容`MultipartHttpMessageReader`，然后缓存结果以进行重复访问。请参阅[API](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)部分中的[多部分数据](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multipart)。[`WebHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)

一旦`getMultipartData()`使用了原始内容，就无法再从请求正文中读取原始内容。因此，应用程序必须始终如一地`getMultipartData()` 用于对零件的重复，类似地图的访问，否则就不得不依赖于 `SynchronossPartHttpMessageReader`一次访问`Flux<Part>`。

##### 限度

`Decoder`和`HttpMessageReader`该缓冲区一些或所有输入流的实施方式中可以与字节在内存中缓冲的最大数量的限制来配置。在某些情况下，发生缓冲是因为输入被汇总并表示为单个对象，例如，带有`@RequestBody byte[]`， `x-www-form-urlencoded`数据等的控制器方法。在分割输入流（例如，定界文本，JSON对象流等）时，流处理也会发生缓冲。对于这些流情况，该限制适用于与流中一个对象关联的字节数。

要配置缓冲区大小，您可以检查给定`Decoder`或`HttpMessageReader` 公开的`maxInMemorySize`属性，如果是，则Javadoc将具有有关默认值的详细信息。在服务器端，`ServerCodecConfigurer`提供了一个放置所有编解码器的位置，请参阅[HTTP消息编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-message-codecs)。在客户端，可以在[WebClient.Builder中](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client-builder-maxinmemorysize)更改所有编解码器的限制 。

对于“[多部分”分析，](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-multipart)该`maxInMemorySize`属性限制了非文件部分的大小。对于文件部件，它确定将部件写入磁盘的阈值。对于写入磁盘的文件部件，还有一个附加 `maxDiskUsagePerPart`属性可以限制每个部件的磁盘空间量。还有一个`maxParts`属性可以限制多部分请求中的部分总数。要配置所有三个WebFlux，你需要提供的预先配置的实例 `MultipartHttpMessageReader`来`ServerCodecConfigurer`。

##### 流媒体

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-http-streaming)

当流传输到HTTP响应（例如，`text/event-stream`， `application/x-ndjson`），它周期性地发送的数据，以便可靠地检测断开的客户端宜早不宜迟是重要的。这样的发送可以是仅注释，空的SSE事件或任何其他可以有效充当心跳的“无操作”数据。

##### `DataBuffer`

`DataBuffer`是WebFlux中字节缓冲区的表示形式。本参考资料的Spring Core部分在“[数据缓冲区和编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers)”部分中有更多介绍 。要理解的关键点是，在诸如Netty之类的某些服务器上，字节缓冲被池化并计数引用，并且在使用时必须将其释放以避免内存泄漏。

WebFlux applications generally do not need to be concerned with such issues, unless they consume or produce data buffers directly, as opposed to relying on codecs to convert to and from higher level objects, or unless they choose to create custom codecs. For such cases please review the information in [Data Buffers and Codecs](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers), especially the section on [Using DataBuffer](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers-using).

#### 1.2.6. Logging

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-logging)

`DEBUG` level logging in Spring WebFlux is designed to be compact, minimal, and human-friendly. It focuses on high value bits of information that are useful over and over again vs others that are useful only when debugging a specific issue.

`TRACE` level logging generally follows the same principles as `DEBUG` (and for example also should not be a firehose) but can be used for debugging any issue. In addition, some log messages may show a different level of detail at `TRACE` vs `DEBUG`.

Good logging comes from the experience of using the logs. If you spot anything that does not meet the stated goals, please let us know.

##### Log Id

In WebFlux, a single request can be run over multiple threads and the thread ID is not useful for correlating log messages that belong to a specific request. This is why WebFlux log messages are prefixed with a request-specific ID by default.

在服务器端，日志ID存储在`ServerWebExchange`属性（[`LOG_ID_ATTRIBUTE`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/server/ServerWebExchange.html#LOG_ID_ATTRIBUTE)）中，而基于ID的全格式前缀可从访问 `ServerWebExchange#getLogPrefix()`。在`WebClient`一侧，日志ID存储在 `ClientRequest`属性（[`LOG_ID_ATTRIBUTE`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/function/client/ClientRequest.html#LOG_ID_ATTRIBUTE)）中，而可从获取完整格式的前缀`ClientRequest#logPrefix()`。

##### 敏感数据

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-logging-sensitive-data)

`DEBUG`并且`TRACE`日志记录可以记录敏感信息。这就是默认情况下屏蔽表单参数和标题的原因，并且必须显式启用它们的完整日志记录。

下面的示例说明如何针对服务器端请求执行此操作：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
class MyConfig implements WebFluxConfigurer {

    @Override
    public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
        configurer.defaultCodecs().enableLoggingRequestDetails(true);
    }
}
```

下面的示例说明如何针对客户端请求执行此操作：

爪哇

科特林

```java
Consumer<ClientCodecConfigurer> consumer = configurer ->
        configurer.defaultCodecs().enableLoggingRequestDetails(true);

WebClient webClient = WebClient.builder()
        .exchangeStrategies(strategies -> strategies.codecs(consumer))
        .build();
```

##### 追加者

日志库（例如SLF4J和Log4J 2）提供了避免阻塞的异步记录器。尽管它们有其自身的缺点，例如可能丢弃无法排队等待记录的消息，但它们是当前在Reactive，非阻塞应用程序中使用的最佳可用选项。

##### 自定义编解码器

应用程序可以注册自定义编解码器以支持其他媒体类型，也可以注册默认编解码器不支持的特定行为。

开发人员表达的某些配置选项在默认编解码器上强制执行。自定义编解码器可能希望有机会与这些首选项保持一致，例如[强制执行缓冲限制](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-limits) 或[记录敏感数据](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-logging-sensitive-data)。

下面的示例说明如何针对客户端请求执行此操作：

爪哇

科特林

```java
WebClient webClient = WebClient.builder()
        .codecs(configurer -> {
                CustomDecoder decoder = new CustomDecoder();
                configurer.customCodecs().registerWithDefaultConfig(decoder);
        })
        .build();
```

### 1.3。 `DispatcherHandler`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)

弹簧WebFlux，类似于弹簧MVC，被设计围绕前端控制器模式，其中一个中心`WebHandler`，所述`DispatcherHandler`，提供了一种用于请求处理的共享算法，而实际的工作是由可配置的，委托组件来执行。该模型非常灵活，并支持多种工作流程。

`DispatcherHandler`从Spring配置中发现所需的委托组件。它还被设计为Spring Bean本身，并实现`ApplicationContextAware` 对其运行上下文的访问。如果`DispatcherHandler`用Bean名称声明了if `webHandler`，那么反过来，它由发现 [`WebHttpHandlerBuilder`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/server/adapter/WebHttpHandlerBuilder.html)，并将请求处理链组合在一起，如[`WebHandler`API中所述](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)。

WebFlux应用程序中的Spring配置通常包含：

- `DispatcherHandler` 用bean子的名字 `webHandler`
- `WebFilter`和`WebExceptionHandler`bean子
- [`DispatcherHandler` 特殊bean](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-special-bean-types)
- 其他

The configuration is given to `WebHttpHandlerBuilder` to build the processing chain, as the following example shows:

Java

Kotlin

```java
ApplicationContext context = ...
HttpHandler handler = WebHttpHandlerBuilder.applicationContext(context).build();
```

The resulting `HttpHandler` is ready for use with a [server adapter](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler).

#### 1.3.1. Special Bean Types

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-special-bean-types)

The `DispatcherHandler` delegates to special beans to process requests and render the appropriate responses. By “special beans,” we mean Spring-managed `Object` instances that implement WebFlux framework contracts. Those usually come with built-in contracts, but you can customize their properties, extend them, or replace them.

The following table lists the special beans detected by the `DispatcherHandler`. Note that there are also some other beans detected at a lower level (see [Special bean types](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api-special-beans) in the Web Handler API).

| Bean type              | Explanation                                                  |
| :--------------------- | :----------------------------------------------------------- |
| `HandlerMapping`       | Map a request to a handler. The mapping is based on some criteria, the details of which vary by `HandlerMapping` implementation — annotated controllers, simple URL pattern mappings, and others.The main `HandlerMapping` implementations are `RequestMappingHandlerMapping` for `@RequestMapping` annotated methods, `RouterFunctionMapping` for functional endpoint routes, and `SimpleUrlHandlerMapping` for explicit registrations of URI path patterns and `WebHandler` instances. |
| `HandlerAdapter`       | Help the `DispatcherHandler` to invoke a handler mapped to a request regardless of how the handler is actually invoked. For example, invoking an annotated controller requires resolving annotations. The main purpose of a `HandlerAdapter` is to shield the `DispatcherHandler` from such details. |
| `HandlerResultHandler` | Process the result from the handler invocation and finalize the response. See [Result Handling](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-resulthandling). |

#### 1.3.2。WebFlux配置

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-config)

应用程序可以声明处理请求所需的基础结构Bean（在[Web Handler API](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api-special-beans)和中 列出 [`DispatcherHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-special-bean-types)）。但是，在大多数情况下，[WebFlux Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)是最佳起点。它声明了所需的bean，并提供了更高级别的配置回调API来对其进行自定义。

|      | Spring Boot依靠WebFlux配置来配置Spring WebFlux，并且还提供了许多额外的方便选项。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.3.3。处理中

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-sequence)

`DispatcherHandler` 处理请求如下：

- `HandlerMapping`要求每个人找到匹配的处理程序，并使用第一个匹配项。
- 如果找到处理程序，则会通过适当的处理程序运行该处理程序，该处理程序会将`HandlerAdapter`执行返回的值公开为`HandlerResult`。
- 将`HandlerResult`被提供给一个适当的`HandlerResultHandler`通过写直接响应或通过使用视图来呈现到完整的处理。

#### 1.3.4。结果处理

调用处理程序通过返回的返回值，将其与和其他一些上下文一起`HandlerAdapter`包装为`HandlerResult`，并传递给`HandlerResultHandler`要求支持它的第一个上下文 。下表显示了可用的 `HandlerResultHandler`实现，所有实现均在[WebFlux Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)中声明：

| 结果处理程序类型              | 返回值                                                       | 默认订单            |
| :---------------------------- | :----------------------------------------------------------- | :------------------ |
| `ResponseEntityResultHandler` | `ResponseEntity`，通常来自`@Controller`实例。                | 0                   |
| `ServerResponseResultHandler` | `ServerResponse`，通常来自功能端点。                         | 0                   |
| `ResponseBodyResultHandler`   | 处理`@ResponseBody`方法或`@RestController`类的返回值。       | 100                 |
| `ViewResolutionResultHandler` | `CharSequence`，[`View`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/result/view/View.html)， [型号](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/ui/Model.html)，`Map`， [渲染](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/result/view/Rendering.html)，或者任何其他`Object`被视为一个模型属性。另请参阅[查看分辨率](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-viewresolution)。 | `Integer.MAX_VALUE` |

#### 1.3.5。例外情况

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers)

从`HandlerResult`返回的值`HandlerAdapter`可以公开基于某些特定于处理程序的机制进行错误处理的函数。在以下情况下将调用此错误函数：

- 处理程序（例如`@Controller`）调用失败。
- 通过`HandlerResultHandler`失败对处理程序返回值的处理。

只要在从处理程序返回的响应类型产生任何数据项之前发生错误信号，错误函数就可以更改响应（例如，更改为错误状态）。

This is how `@ExceptionHandler` methods in `@Controller` classes are supported. By contrast, support for the same in Spring MVC is built on a `HandlerExceptionResolver`. This generally should not matter. However, keep in mind that, in WebFlux, you cannot use a `@ControllerAdvice` to handle exceptions that occur before a handler is chosen.

See also [Managing Exceptions](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-controller-exceptions) in the “Annotated Controller” section or [Exceptions](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-exception-handler) in the WebHandler API section.

#### 1.3.6. View Resolution

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-viewresolver)

View resolution enables rendering to a browser with an HTML template and a model without tying you to a specific view technology. In Spring WebFlux, view resolution is supported through a dedicated [HandlerResultHandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-resulthandling) that uses `ViewResolver` instances to map a String (representing a logical view name) to a `View` instance. The `View` is then used to render the response.

##### Handling

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-handling)

The `HandlerResult` passed into `ViewResolutionResultHandler` contains the return value from the handler and the model that contains attributes added during request handling. The return value is processed as one of the following:

- `String`, `CharSequence`: A logical view name to be resolved to a `View` through the list of configured `ViewResolver` implementations.
- `void`: Select a default view name based on the request path, minus the leading and trailing slash, and resolve it to a `View`. The same also happens when a view name was not provided (for example, model attribute was returned) or an async return value (for example, `Mono` completed empty).
- [Rendering](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/result/view/Rendering.html): API for view resolution scenarios. Explore the options in your IDE with code completion.
- `Model`, `Map`: Extra model attributes to be added to the model for the request.
- Any other: Any other return value (except for simple types, as determined by [BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)) is treated as a model attribute to be added to the model. The attribute name is derived from the class name by using [conventions](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/Conventions.html), unless a handler method `@ModelAttribute` annotation is present.

The model can contain asynchronous, reactive types (for example, from Reactor or RxJava). Prior to rendering, `AbstractView` resolves such model attributes into concrete values and updates the model. Single-value reactive types are resolved to a single value or no value (if empty), while multi-value reactive types (for example, `Flux<T>`) are collected and resolved to `List<T>`.

配置视图分辨率就像将一个`ViewResolutionResultHandler`bean添加到Spring配置中一样简单。[WebFlux Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-view-resolvers)提供了专用于视图分辨率的配置API。

有关与Spring WebFlux集成的视图技术的更多信息，请参见[View Technologies](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view)。

##### 重新导向

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-redirecting-redirect-prefix)

`redirect:`视图名称中的特殊前缀使您可以执行重定向。的 `UrlBasedViewResolver`（和子类别）识别为是需要重定向的指令。视图名称的其余部分是重定向URL。

最终效果与控制器返回a`RedirectView`或一样 `Rendering.redirectTo("abc").build()`，但是现在控制器本身可以根据逻辑视图名称进行操作。诸如 `redirect:/some/resource`相对于当前应用程序的视图名称，而诸如`redirect:https://example.com/arbitrary/path`重定向到绝对URL的视图名称 。

##### 内容协商

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multiple-representations)

`ViewResolutionResultHandler`支持内容协商。它将请求的媒体类型与每个选定的支持的媒体类型进行比较`View`。使用`View` 支持请求的媒体类型的第一个。

为了支持JSON和XML等媒体类型，Spring WebFlux提供 `HttpMessageWriterView`了一种特殊的特性`View`，它通过[HttpMessageWriter进行](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)呈现 。通常，您可以通过[WebFlux Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-view-resolvers)将它们配置为默认视图。如果默认视图与请求的媒体类型匹配，则始终会选择和使用它们。

### 1.4。带注释的控制器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-controller)

Spring WebFlux提供了一个基于注释的编程模型，其中`@Controller`和 `@RestController`组件使用注释来表达请求映射，请求输入，处理异常等等。带注释的控制器具有灵活的方法签名，无需扩展基类或实现特定的接口。

以下清单显示了一个基本示例：

爪哇

科特林

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String handle() {
        return "Hello WebFlux";
    }
}
```

在前面的示例中，该方法将a`String`写入响应主体。

#### 1.4.1。 `@Controller`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller)

您可以使用标准的Spring bean定义来定义控制器bean。该`@Controller`原型允许自动检测，并与Spring普遍支持排列用于检测`@Component`在类路径中类和自动注册bean定义他们。它还充当带注释类的构造型，表明其作为Web组件的作用。

要启用对此类`@Controller`bean的自动检测，可以将组件扫描添加到Java配置中，如以下示例所示：

爪哇

科特林

```java
@Configuration
@ComponentScan("org.example.web") 
public class WebConfig {

    // ...
}
```

|      | 扫描`org.example.web`包裹。 |
| ---- | --------------------------- |
|      |                             |

`@RestController` is a [composed annotation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations) that is itself meta-annotated with `@Controller` and `@ResponseBody`, indicating a controller whose every method inherits the type-level `@ResponseBody` annotation and, therefore, writes directly to the response body versus view resolution and rendering with an HTML template.

#### 1.4.2. Request Mapping

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping)

The `@RequestMapping` annotation is used to map requests to controllers methods. It has various attributes to match by URL, HTTP method, request parameters, headers, and media types. You can use it at the class level to express shared mappings or at the method level to narrow down to a specific endpoint mapping.

There are also HTTP method specific shortcut variants of `@RequestMapping`:

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

前面的注释是提供的“[自定义注释”](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-composed)，因为可以说，大多数控制器方法应映射到特定的HTTP方法，而不是使用using `@RequestMapping`，默认情况下，使用using匹配所有HTTP方法。同时，`@RequestMapping`在类级别仍然需要a 来表示共享映射。

以下示例使用类型和方法级别的映射：

爪哇

科特林

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

##### URI模式

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-uri-templates)

您可以使用全局模式和通配符来映射请求：

| 模式            | 描述                                                         | 例                                                           |
| :-------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `?`             | 匹配一个字符                                                 | `"/pages/t?st.html"`匹配`"/pages/test.html"`和`"/pages/t3st.html"` |
| `*`             | 匹配路径段中的零个或多个字符                                 | `"/resources/*.png"` 火柴 `"/resources/file.png"``"/projects/*/versions"`匹配`"/projects/spring/versions"`但不匹配`"/projects/spring/boot/versions"` |
| `**`            | 匹配零个或多个路径段，直到路径结束                           | `"/resources/**"`匹配`"/resources/file.png"`和`"/resources/images/file.png"``"/resources/**/file.png"`无效，`**`仅在路径末尾才允许。 |
| `{name}`        | 匹配路径段并将其捕获为名为“ name”的变量                      | `"/projects/{project}/versions"`比赛`"/projects/spring/versions"`和捕获`project=spring` |
| `{name:[a-z]+}` | 将正则表达式匹配`"[a-z]+"`为名为“名称”的路径变量             | `"/projects/{project:[a-z]+}/versions"`匹配`"/projects/spring/versions"`但不匹配`"/projects/spring1/versions"` |
| `{*path}`       | 匹配零个或多个路径段，直到路径结尾，并将其捕获为名为“ path”的变量 | `"/resources/{*file}"`比赛`"/resources/images/file.png"`和捕获`file=images/file.png` |

捕获的URI变量可以通过访问`@PathVariable`，如以下示例所示：

爪哇

科特林

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

您可以在类和方法级别声明URI变量，如以下示例所示：

爪哇

科特林

```java
@Controller
@RequestMapping("/owners/{ownerId}") 
public class OwnerController {

    @GetMapping("/pets/{petId}") 
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

|      | 类级URI映射。   |
| ---- | --------------- |
|      | 方法级URI映射。 |

URI变量会自动转换为适当的类型，或者`TypeMismatchException` 引发a。简单类型（`int`，`long`，`Date`，等）默认支持，你可以注册任何其它数据类型的支持。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-typeconversion)和[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-initbinder)。

URI变量可以显式命名（例如`@PathVariable("customId")`），但是如果名称相同，则可以省略此详细信息，并且可以使用调试信息或`-parameters`Java 8上的编译器标志来编译代码。

该语法`{*varName}`声明了一个与零个或多个剩余路径段匹配的URI变量。例如，`/resources/{*path}`匹配下方的所有文件`/resources/`，并且 `"path"`变量捕获完整的相对路径。

该语法`{varName:regex}`使用具有以下语法的正则表达式声明URI变量`{varName:regex}`。例如，给定URL为`/spring-web-3.0.5 .jar`，以下方法提取名称，版本和文件扩展名：

爪哇

科特林

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}
```

URI路径模式还可以具有嵌入的`${…}`占位符，这些占位符在启动时会`PropertyPlaceHolderConfigurer`针对本地，系统，环境和其他属性源进行解析。您可以使用它来例如基于某些外部配置参数化基本URL。

|      | Spring WebFlux使用`PathPattern`和`PathPatternParser`for URI路径匹配支持。这两个类都位于`spring-web`Web应用程序中，并专门设计用于HTTP URL路径，在Web应用程序中，在运行时会匹配大量URI路径模式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring WebFlux不支持后缀模式匹配-与Spring MVC不同，Spring MVC中的映射`/person`也与匹配`/person.*`。对于基于URL的内容协商，如果需要，我们建议使用查询参数，该参数更简单，更明确，并且不易受到基于URL路径的攻击。

##### 模式比较

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-pattern-comparison)

当多个模式与URL匹配时，必须将它们进行比较以找到最佳匹配。这是通过完成的`PathPattern.SPECIFICITY_COMPARATOR`，它会寻找更具体的模式。

对于每个模式，都会根据URI变量和通配符的数量计算得分，其中URI变量的得分低于通配符。总得分较低的模式将获胜。如果两个模式的分数相同，则选择更长的时间。

包罗万象的模式（例如`**`，`{*varName}`）被排除在得分，总是最后整理代替。如果两种模式都通用，则选择较长的模式。

##### 消耗媒体类型

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-consumes)

您可以根据请求的来缩小请求映射`Content-Type`，如以下示例所示：

爪哇

科特林

```java
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

消耗属性还支持否定表达式-例如，`!text/plain`表示以外的任何内容类型`text/plain`。

您可以`consumes`在类级别声明共享属性。但是，与大多数其他请求映射属性不同，在类级别使用时，方法级别的`consumes`属性会覆盖而不是扩展类级别的声明。

|      | `MediaType`提供常用媒体类型的常量，例如 `APPLICATION_JSON_VALUE`和`APPLICATION_XML_VALUE`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 可生产的媒体类型

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-produces)

您可以根据`Accept`请求标头和控制器方法生成的内容类型列表来缩小请求映射，如以下示例所示：

爪哇

科特林

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json")
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

媒体类型可以指定字符集。支持否定表达式-例如， `!text/plain`表示以外的任何内容类型`text/plain`。

您可以`produces`在类级别声明共享属性。但是，与大多数其他请求映射属性不同，在类级别使用时，方法级别的`produces`属性将覆盖而不是扩展类级别的声明。

|      | `MediaType`提供了常用的介质类型的常量-例如 `APPLICATION_JSON_VALUE`，`APPLICATION_XML_VALUE`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 参数和标题

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-params-and-headers)

您可以根据查询参数条件来缩小请求映射。您可以测试查询参数（`myParam`）的存在，不存在（`!myParam`）或特定值（`myParam=myValue`）。以下示例测试具有值的参数：

爪哇

科特林

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

|      | 检查是否`myParam`相等`myValue`。 |
| ---- | -------------------------------- |
|      |                                  |

您还可以将其与请求标头条件一起使用，如以下示例所示：

爪哇

科特林

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

|      | 检查是否`myHeader`相等`myValue`。 |
| ---- | --------------------------------- |
|      |                                   |

##### HTTP HEAD，选项

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-head-options)

`@GetMapping`并`@RequestMapping(method=HttpMethod.GET)`透明地支持HTTP HEAD以进行请求映射。控制器方法无需更改。`HttpHandler`服务器适配器中应用的响应包装器可确保将`Content-Length` 标头设置为写入的字节数，而无需实际写入响应。

By default, HTTP OPTIONS is handled by setting the `Allow` response header to the list of HTTP methods listed in all `@RequestMapping` methods with matching URL patterns.

For a `@RequestMapping` without HTTP method declarations, the `Allow` header is set to `GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS`. Controller methods should always declare the supported HTTP methods (for example, by using the HTTP method specific variants — `@GetMapping`, `@PostMapping`, and others).

You can explicitly map a `@RequestMapping` method to HTTP HEAD and HTTP OPTIONS, but that is not necessary in the common case.

##### Custom Annotations

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-composed)

Spring WebFlux支持将[组合注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations) 用于请求映射。这些注解本身是元注解， `@RequestMapping`并组成它们以`@RequestMapping` 更狭窄，更具体的目的重新声明属性的子集（或全部）。

`@GetMapping`，`@PostMapping`，`@PutMapping`，`@DeleteMapping`，和`@PatchMapping`由注解的例子。之所以提供它们，是因为可以说，大多数控制器方法应该映射到特定的HTTP方法，而不是使用using `@RequestMapping`（默认情况下与所有HTTP方法都匹配）。如果需要组合注释的示例，请查看如何声明它们。

Spring WebFlux还支持具有自定义请求匹配逻辑的自定义请求映射属性。这是一个更高级的选项，它需要`RequestMappingHandlerMapping`对`getCustomMethodCondition`方法进行子类化 和覆盖，在该方法中您可以检查custom属性并返回自己的属性`RequestCondition`。

##### 明确注册

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-registration)

您可以以编程方式注册Handler方法，该方法可用于动态注册或高级案例，例如，不同URL下同一处理程序的不同实例。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) 
            throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build(); 

        Method method = UserHandler.class.getMethod("getUser", Long.class); 

        mapping.registerMapping(info, handler, method); 
    }

}
```

|      | 注入目标处理程序和控制器的处理程序映射。 |
| ---- | ---------------------------------------- |
|      | 准备请求映射元数据。                     |
|      | 获取处理程序方法。                       |
|      | 添加注册。                               |

#### 1.4.3。处理程序方法

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-methods)

`@RequestMapping` 处理程序方法具有灵活的签名，可以从支持的控制器方法参数和返回值的范围中进行选择。

##### 方法参数

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)

下表显示了受支持的控制器方法参数。

需要解析I / O（例如，读取请求正文）的自变量支持Reactive类型（Reactor，RxJava[或other](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)）。这在“描述”列中进行了标记。不需要阻塞的参数不应使用Reactive类型。

JDK 1.8的`java.util.Optional`被支撑作为组合的方法的参数与具有注解`required`的属性（例如，`@RequestParam`，`@RequestHeader`，和其它物质）和相当于`required=false`。

| 控制器方法参数                                               | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ServerWebExchange`                                          | 访问full `ServerWebExchange` — HTTP请求和响应，请求和会话属性，`checkNotModified`方法等的容器。 |
| `ServerHttpRequest`， `ServerHttpResponse`                   | 访问HTTP请求或响应。                                         |
| `WebSession`                                                 | 访问会话。除非添加了属性，否则这不会强制开始新的会话。支持反应类型。 |
| `java.security.Principal`                                    | 当前经过身份验证的用户-可能是特定的`Principal`实现类（如果已知）。支持反应类型。 |
| `org.springframework.http.HttpMethod`                        | 请求的HTTP方法。                                             |
| `java.util.Locale`                                           | 当前请求的语言环境，取决于最具体的`LocaleResolver`可用语言（实际上是配置的`LocaleResolver`/）`LocaleContextResolver`。 |
| `java.util.TimeZone` + `java.time.ZoneId`                    | 与当前请求关联的时区，由决定`LocaleContextResolver`。        |
| `@PathVariable`                                              | 用于访问URI模板变量。请参阅[URI模式](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestmapping-uri-templates)。 |
| `@MatrixVariable`                                            | 用于访问URI路径段中的名称/值对。请参阅[矩阵变量](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-matrix-variables)。 |
| `@RequestParam`                                              | 用于访问Servlet请求参数。参数值将转换为声明的方法参数类型。请参阅[`@RequestParam`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestparam)。请注意，使用of`@RequestParam`是可选的-例如，设置其属性。请参阅此表后面的“其他任何参数”。 |
| `@RequestHeader`                                             | 用于访问请求标头。标头值将转换为声明的方法参数类型。请参阅[`@RequestHeader`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestheader)。 |
| `@CookieValue`                                               | 用于访问cookie。Cookie值将转换为声明的方法参数类型。请参阅[`@CookieValue`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-cookievalue)。 |
| `@RequestBody`                                               | 用于访问HTTP请求正文。正文内容通过使用`HttpMessageReader`实例转换为声明的方法参数类型。支持反应类型。请参阅[`@RequestBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestbody)。 |
| `HttpEntity<B>`                                              | 用于访问请求标头和正文。主体将通过`HttpMessageReader`实例进行转换。支持反应类型。请参阅[`HttpEntity`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-httpentity)。 |
| `@RequestPart`                                               | 用于访问`multipart/form-data`请求中的零件。支持反应类型。请参见[多部分内容](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multipart-forms)和[多部分数据](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multipart)。 |
| `java.util.Map`，`org.springframework.ui.Model`和`org.springframework.ui.ModelMap`。 | 用于访问HTML控制器中使用的模型，并作为视图渲染的一部分公开给模板。 |
| `@ModelAttribute`                                            | 用于访问应用了数据绑定和验证的模型中的现有属性（如果不存在，则进行实例化）。见[`@ModelAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-method-args)以及[`Model`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-methods)和[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-initbinder)。请注意，使用of`@ModelAttribute`是可选的-例如，设置其属性。请参阅此表后面的“其他任何参数”。 |
| `Errors`， `BindingResult`                                   | For access to errors from validation and data binding for a command object, i.e. a `@ModelAttribute` argument. An `Errors`, or `BindingResult` argument must be declared immediately after the validated method argument. |
| `SessionStatus` + class-level `@SessionAttributes`           | For marking form processing complete, which triggers cleanup of session attributes declared through a class-level `@SessionAttributes` annotation. See [`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-sessionattributes) for more details. |
| `UriComponentsBuilder`                                       | For preparing a URL relative to the current request’s host, port, scheme, and context path. See [URI Links](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-uri-building). |
| `@SessionAttribute`                                          | For access to any session attribute — in contrast to model attributes stored in the session as a result of a class-level `@SessionAttributes` declaration. See [`@SessionAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-sessionattribute) for more details. |
| `@RequestAttribute`                                          | For access to request attributes. See [`@RequestAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestattrib) for more details. |
| Any other argument                                           | 如果一个方法参数不匹配于任何上述的，它是，在默认情况下，解析为`@RequestParam`，如果它是一个简单的类型，如由下式确定 [的BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)，或作为`@ModelAttribute`，否则。 |

##### 返回值

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)

下表显示了受支持的控制器方法返回值。注意从库，比如reactive-stack，RxJava，是响应型[或其他](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)一般都支持所有的返回值。

| 控制器方法返回值                                             | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `@ResponseBody`                                              | 返回值通过`HttpMessageWriter`实例进行编码并写入响应。请参阅[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-responsebody)。 |
| `HttpEntity<B>`， `ResponseEntity<B>`                        | 返回值指定完整的响应，包括HTTP标头，并且正文通过`HttpMessageWriter`实例进行编码并写入响应。请参阅[`ResponseEntity`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-responseentity)。 |
| `HttpHeaders`                                                | 用于返回不包含标题的响应。                                   |
| `String`                                                     | 通过`ViewResolver`实例解析并与隐式模型一起使用的视图名称（通过命令对象和`@ModelAttribute`方法确定）。该处理程序方法还可以通过声明一个`Model`参数（[如前所述](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-viewresolution-handling)）以编程方式丰富模型。 |
| `View`                                                       | 甲`View`实例以使用用于与所述隐式模型一起渲染-通过命令对象和确定`@ModelAttribute`方法。该处理程序方法还可以通过声明一个`Model`参数（[如前所述](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-viewresolution-handling)）以编程方式丰富模型。 |
| `java.util.Map`， `org.springframework.ui.Model`             | 要添加到隐式模型的属性，其中视图名称根据请求路径隐式确定。   |
| `@ModelAttribute`                                            | 要添加到模型的属性，视图名称根据请求路径隐式确定。请注意，这`@ModelAttribute`是可选的。请参阅此表后面的“其他任何返回值”。 |
| `Rendering`                                                  | 用于模型和视图渲染方案的API。                                |
| `void`                                                       | 具有`void`可能是异步的（例如`Mono<Void>`）返回类型（或`null`返回值）的方法（如果它也具有`ServerHttpResponse`，`ServerWebExchange`参数或`@ResponseStatus`注释）被认为已完全处理了响应。如果控制器对ETag或`lastModified`时间戳进行肯定检查，也是如此。// TODO：有关详细信息，请参见[控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-caching-etag-lastmodified)。如果以上条件都不成立，则`void`返回类型还可以为REST控制器指示“无响应正文”，或者为HTML控制器指示默认视图名称选择。 |
| `Flux<ServerSentEvent>`，`Observable<ServerSentEvent>`或其它反应类型 | 发出服务器发送的事件。的`ServerSentEvent`可以在将要写入的数据仅需要被省略包装物（但是，`text/event-stream`必须要求或在通过映射声明`produces`属性）。 |
| 任何其他返回值                                               | 如果返回值与上述任何一个都不匹配，则默认情况下将其视为视图名称（如果为`String`或，则视为视图名称）`void`（适用于默认视图名称选择），或作为要添加至模型的模型属性，除非它是一个简单类型，由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ，在这种情况下，它仍然无法解析。 |

##### 类型转换

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-typeconversion)

Some annotated controller method arguments that represent String-based request input (for example, `@RequestParam`, `@RequestHeader`, `@PathVariable`, `@MatrixVariable`, and `@CookieValue`) can require type conversion if the argument is declared as something other than `String`.

For such cases, type conversion is automatically applied based on the configured converters. By default, simple types (such as `int`, `long`, `Date`, and others) are supported. Type conversion can be customized through a `WebDataBinder` (see [`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-initbinder)) or by registering `Formatters` with the `FormattingConversionService` (see [Spring Field Formatting](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format)).

##### Matrix Variables

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-matrix-variables)

[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3)讨论路径段中的名称/值对。在Spring WebFlux中，基于Tim Berners-Lee的[“旧帖子”](https://www.w3.org/DesignIssues/MatrixURIs.html)，我们将其称为“矩阵变量” ，但它们也可以称为URI路径参数。

矩阵变量可以出现在任何路径段中，每个变量用分号分隔，多个值用逗号分隔，例如`"/cars;color=red,green;year=2012"`。也可以通过重复的变量名称指定多个值，例如 `"color=red;color=green;color=blue"`。

与Spring MVC不同，在WebFlux中，URL中是否存在矩阵变量不会影响请求映射。换句话说，您不需要使用URI变量来屏蔽变量内容。就是说，如果要从控制器方法访问矩阵变量，则需要将URI变量添加到期望矩阵变量的路径段中。以下示例显示了如何执行此操作：

爪哇

科特林

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

鉴于所有路径段都可以包含矩阵变量，因此有时可能需要消除矩阵变量应位于哪个路径变量的歧义，如以下示例所示：

爪哇

科特林

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

您可以定义一个矩阵变量，可以将其定义为可选变量并指定一个默认值，如以下示例所示：

爪哇

科特林

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

要获取所有矩阵变量，请使用`MultiValueMap`，如以下示例所示：

爪哇

科特林

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

##### `@RequestParam`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestparam)

您可以使用`@RequestParam`批注将查询参数绑定到控制器中的方法参数。以下代码段显示了用法：

爪哇

科特林

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...
}
```

|      | 使用`@RequestParam`。 |
| ---- | --------------------- |
|      |                       |

|      | Servlet API的“请求参数”概念将查询参数，表单数据和多部分合并为一个。但是，在WebFlux中，每个都可以通过单独访问 `ServerWebExchange`。虽然`@RequestParam`仅绑定到查询参数，但是您可以使用数据绑定将查询参数，表单数据和多部分应用于 [命令对象](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-method-args)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`@RequestParam`默认情况下，使用注释的方法参数是必需的，但是您可以通过将a的必需标志设置为`@RequestParam` to`false`或通过使用`java.util.Optional` 包装器声明参数来指定方法参数是可选的。

如果目标方法参数类型不是，则类型转换将自动应用 `String`。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-typeconversion)。

在或 参数`@RequestParam`上声明注释时，将使用所有查询参数填充地图。`Map<String, String>``MultiValueMap<String, String>`

请注意，使用of`@RequestParam`是可选的-例如，设置其属性。默认情况下，任何简单值类型的参数（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ）都没有被其他任何参数解析器解析，就如同使用注释一样`@RequestParam`。

##### `@RequestHeader`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestheader)

您可以使用`@RequestHeader`注释将请求标头绑定到控制器中的方法参数。

以下示例显示了带有标头的请求：

```
主机本地主机：8080
接受text / html，application / xhtml + xml，application / xml; q = 0.9
接受语言fr，en-gb; q = 0.7，en; q = 0.3
接受编码gzip，放气
接受字符集ISO-8859-1，utf-8; q = 0.7，*; q = 0.7
保持生命300
```

以下示例获取`Accept-Encoding`和`Keep-Alive`标头的值：

爪哇

科特林

```java
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

|      | 获取`Accept-Encoging`标头的值。 |
| ---- | ------------------------------- |
|      | 获取`Keep-Alive`标头的值。      |

如果目标方法参数类型不是，则类型转换将自动应用 `String`。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-typeconversion)。

当`@RequestHeader`注解上的使用`Map<String, String>`， `MultiValueMap<String, String>`或`HttpHeaders`参数，则地图被填充有所有标头值。

|      | 内置支持可用于将逗号分隔的字符串转换为数组或字符串集合或类型转换系统已知的其他类型。例如，用注释的方法参数`@RequestHeader("Accept")`可以是类型 `String`，也可以是`String[]`或`List<String>`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### `@CookieValue`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-cookievalue)

您可以使用`@CookieValue`注释将HTTP cookie的值绑定到控制器中的方法参数。

以下示例显示了一个带有cookie的请求：

```
JSESSIONID = 415A4AC178C59DACE0B2C9CA727CDD84
```

以下代码示例演示如何获取cookie值：

爪哇

科特林

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
    //...
}
```

|      | 获取cookie值。 |
| ---- | -------------- |
|      |                |

如果目标方法参数类型不是，则类型转换将自动应用 `String`。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-typeconversion)。

##### `@ModelAttribute`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-method-args)

您可以`@ModelAttribute`在方法参数上使用注释，以从模型访问属性，或将其实例化（如果不存在）。model属性还覆盖了名称与字段名称匹配的查询参数和表单字段的值。这称为数据绑定，它使您不必处理解析和转换单个查询参数和表单字段的工作。以下示例绑定了的一个实例`Pet`：

爪哇

科特林

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { } 
```

|      | 绑定的实例`Pet`。 |
| ---- | ----------------- |
|      |                   |

上`Pet`例中的实例解析如下：

- 从模型（如果已通过添加）[`Model`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-methods)。
- 从HTTP会话到[`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-sessionattributes)。
- 从默认构造函数的调用开始。
- 从带有匹配查询参数或表单字段的参数的“主要构造函数”的调用开始。参数名称是通过JavaBeans `@ConstructorProperties`或字节码中运行时保留的参数名称确定的。

获取模型属性实例后，将应用数据绑定。在 `WebExchangeDataBinder`类匹配的查询参数和表单字段名称目标字段名`Object`。必要时在应用类型转换后填充匹配字段。有关数据绑定（和验证）的更多信息，请参见 [验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)。有关自定义数据绑定的更多信息，请参见 [`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-initbinder)。

数据绑定可能导致错误。默认情况下，`WebExchangeBindException`引发a，但是，要检查controller方法中的此类错误，可以在`BindingResult`旁边紧接着添加一个参数`@ModelAttribute`，如以下示例所示：

爪哇

科特林

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

|      | 添加一个`BindingResult`。 |
| ---- | ------------------------- |
|      |                           |

您可以通过添加`javax.validation.Valid`注释或Spring的`@Validated`注释在数据绑定之后自动应用验证 （另请参见 [Bean验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation)和 [Spring验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)）。以下示例使用`@Valid`注释：

爪哇

科特林

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

|      | 使用`@Valid`一个模型属性参数。 |
| ---- | ------------------------------ |
|      |                                |

与Spring MVC不同，Spring WebFlux支持模型中的Reactive类型，例如 `Mono<Account>`或`io.reactivex.Single<Account>`。您可以声明`@ModelAttribute`带有或不带有Reactive类型包装器的参数，并将根据需要将其解析为实际值。但是，请注意，要使用`BindingResult` 参数，您必须在`@ModelAttribute`参数之前声明它，而无需使用Reactive类型包装器，如先前所示。另外，您可以通过Reactive处理任何错误，如以下示例所示：

爪哇

科特林

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public Mono<String> processSubmit(@Valid @ModelAttribute("pet") Mono<Pet> petMono) {
    return petMono
        .flatMap(pet -> {
            // ...
        })
        .onErrorResume(ex -> {
            // ...
        });
}
```

请注意，使用of`@ModelAttribute`是可选的-例如，设置其属性。默认情况下，任何不是简单值类型（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ）且未被其他任何参数解析器解析的参数都将被视为`@ModelAttribute`。

##### `@SessionAttributes`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattributes)

`@SessionAttributes`用于`WebSession`在两次请求之间存储模型属性。它是类型级别的注释，用于声明特定控制器使用的会话属性。这通常列出应透明地存储在会话中以供后续访问请求的模型属性的名称或模型属性的类型。

考虑以下示例：

爪哇

科特林

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {
    // ...
}
```

|      | 使用`@SessionAttributes`注释。 |
| ---- | ------------------------------ |
|      |                                |

在第一个请求中，将名称为的模型属性`pet`添加到模型后，它会自动升级到并保存在中`WebSession`。它会一直保留在那里，直到另一个控制器方法使用`SessionStatus`方法参数来清除存储，如以下示例所示：

爪哇

科特林

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) { 
        if (errors.hasErrors()) {
            // ...
        }
            status.setComplete();
            // ...
        }
    }
}
```

|      | 使用`@SessionAttributes`注释。 |
| ---- | ------------------------------ |
|      | 使用`SessionStatus`变量。      |

##### `@SessionAttribute`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattribute)

如果您需要访问全局存在（即，在控制器外部，例如，通过过滤器）进行管理且可能存在或不存在的预先存在的会话属性，则可以`@SessionAttribute`在方法参数上使用注释，如下所示示例显示：

爪哇

科特林

```java
@GetMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

|      | 使用`@SessionAttribute`。 |
| ---- | ------------------------- |
|      |                           |

对于需要添加或删除会话属性的用例，请考虑注入 `WebSession`到controller方法中。

要将模型属性临时存储在会话中作为控制器工作流的一部分，请考虑使用`SessionAttributes`，如中所述 [`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-sessionattributes)。

##### `@RequestAttribute`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestattrib)

与相似`@SessionAttribute`，您可以使用`@RequestAttribute`注释访问先前创建的预先存在的请求属性（例如通过`WebFilter`），如以下示例所示：

爪哇

科特林

```java
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 
    // ...
}
```

|      | 使用`@RequestAttribute`。 |
| ---- | ------------------------- |
|      |                           |

##### 多部分内容

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart-forms)

如[Multipart Data中所述](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multipart)，`ServerWebExchange`提供对多部分内容的访问。在控制器中处理文件上传表单（例如，从浏览器）的最佳方法是通过将数据绑定到[命令对象](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-method-args)，如以下示例所示：

爪哇

科特林

```java
class MyForm {

    private String name;

    private MultipartFile file;

    // ...

}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {
        // ...
    }

}
```

您还可以在RESTful服务方案中从非浏览器客户端提交多部分请求。以下示例将文件与JSON一起使用：

```
POST / someUrl
内容类型：多部分/混合

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
内容处置：表单数据；name =“元数据”
内容类型：application / json; 字符集= UTF-8
内容传输编码：8bit

{
    “ name”：“值”
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
内容处置：表单数据；name =“文件数据”; filename =“ file.properties”
内容类型：text / xml
内容传输编码：8bit
...文件数据...
```

您可以使用来访问各个部分`@RequestPart`，如以下示例所示：

爪哇

科特林

```java
@PostMapping("/")
public String handle(@RequestPart("meta-data") Part metadata, 
        @RequestPart("file-data") FilePart file) { 
    // ...
}
```

|      | 使用`@RequestPart`获得的元数据。 |
| ---- | -------------------------------- |
|      | 使用`@RequestPart`来获取文件。   |

要反序列化原始零件的内容（例如，转换为JSON，类似于`@RequestBody`），可以声明一个具体的target `Object`，而不是`Part`，如下面的示例所示：

爪哇

科特林

```java
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata) { 
    // ...
}
```

|      | 使用`@RequestPart`获得的元数据。 |
| ---- | -------------------------------- |
|      |                                  |

可以`@RequestPart`与`javax.validation.Valid`或Spring的 `@Validated`注释结合使用，这将导致应用标准Bean验证。验证错误会导致导致`WebExchangeBindException`400（BAD_REQUEST）响应。异常包含`BindingResult`带有错误详细信息的异常，也可以在控制器方法中通过以下方式处理：使用异步包装器声明参数，然后使用与错误相关的运算符：

爪哇

科特林

```java
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") Mono<MetaData> metadata) {
    // use one of the onError* operators...
}
```

要以形式访问所有多部分数据`MultiValueMap`，可以使用`@RequestBody`，如以下示例所示：

爪哇

科特林

```java
@PostMapping("/")
public String handle(@RequestBody Mono<MultiValueMap<String, Part>> parts) { 
    // ...
}
```

|      | 使用`@RequestBody`。 |
| ---- | -------------------- |
|      |                      |

访问多部分数据顺序，以流的方式，可以使用`@RequestBody`与 `Flux<Part>`（或`Flow<Part>`在科特林）代替，如下面的示例所示：

爪哇

科特林

```java
@PostMapping("/")
public String handle(@RequestBody Flux<Part> parts) { 
    // ...
}
```

|      | 使用`@RequestBody`。 |
| ---- | -------------------- |
|      |                      |

##### `@RequestBody`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestbody)

您可以使用`@RequestBody`注释使请求正文`Object`通过[HttpMessageReader](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)读取并反序列化为 。下面的示例使用一个`@RequestBody`参数：

爪哇

科特林

```java
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

与Spring MVC不同，在WebFlux中，`@RequestBody`method参数支持响应类型以及完全无阻塞的读取和（客户端到服务器）流传输。

爪哇

科特林

```java
@PostMapping("/accounts")
public void handle(@RequestBody Mono<Account> account) {
    // ...
}
```

您可以使用[WebFlux Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)的[HTTP消息编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-message-codecs)选项来配置或自定义消息阅读器。

可以`@RequestBody`与`javax.validation.Valid`或Spring的 `@Validated`注释结合使用，这将导致应用标准Bean验证。验证错误会导致`WebExchangeBindException`，导致400（BAD_REQUEST）响应。异常包含一个`BindingResult`带有错误详细信息的异常，可以通过使用异步包装器声明该参数，然后使用与错误相关的运算符在controller方法中进行处理：

爪哇

科特林

```java
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Mono<Account> account) {
    // use one of the onError* operators...
}
```

##### `HttpEntity`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-httpentity)

`HttpEntity`与使用大致相同，[`@RequestBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-requestbody)但是基于公开请求标头和正文的容器对象。以下示例使用 `HttpEntity`：

爪哇

科特林

```java
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```

##### `@ResponseBody`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responsebody)

You can use the `@ResponseBody` annotation on a method to have the return serialized to the response body through an [HttpMessageWriter](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs). The following example shows how to do so:

Java

Kotlin

```java
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

`@ResponseBody` is also supported at the class level, in which case it is inherited by all controller methods. This is the effect of `@RestController`, which is nothing more than a meta-annotation marked with `@Controller` and `@ResponseBody`.

`@ResponseBody` supports reactive types, which means you can return Reactor or RxJava types and have the asynchronous values they produce rendered to the response. For additional details, see [Streaming](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-streaming) and [JSON rendering](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-jackson).

You can combine `@ResponseBody` methods with JSON serialization views. See [Jackson JSON](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-jackson) for details.

您可以使用[WebFlux Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)的[HTTP消息编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-message-codecs)选项来配置或自定义消息编写。

##### `ResponseEntity`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responseentity)

`ResponseEntity`就像[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-responsebody)但带有状态和标题。例如：

爪哇

科特林

```java
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

WebFlux支持使用单值[响应类型](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)来`ResponseEntity`异步生成和/或为主体生成单值和多值响应类型。

##### 杰克逊JSON

Spring提供了对Jackson JSON库的支持。

###### JSON视图

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-jackson)

Spring WebFlux为[Jackson的序列化视图](https://www.baeldung.com/jackson-json-view-annotation)提供了内置支持，该[视图](https://www.baeldung.com/jackson-json-view-annotation)仅可呈现.NET 中所有字段的一部分`Object`。要将其与 `@ResponseBody`或`ResponseEntity`控制器方法一起使用，可以使用Jackson的 `@JsonView`注释来激活序列化视图类，如以下示例所示：

爪哇

科特林

```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

|      | `@JsonView`允许一组视图类，但每个控制器方法只能指定一个。如果需要激活多个视图，请使用复合界面。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.4.4。 `Model`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-methods)

您可以使用`@ModelAttribute`注释：

- 在[方法](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-method-args)中的[方法参数上](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-modelattrib-method-args)，`@RequestMapping`用于从模型创建或访问对象，并通过将该对象绑定到请求 `WebDataBinder`。
- 作为`@Controller`或`@ControllerAdvice`类中的方法级注释，有助于在任何`@RequestMapping`方法调用之前初始化模型。
- 在一个`@RequestMapping`方法来标记它的返回值作为一个模型属性。

本节讨论`@ModelAttribute`方法，或前面列表中的第二项。控制器可以有多种`@ModelAttribute`方法。`@RequestMapping`在同一个控制器中，所有这些方法均在方法之前被调用。`@ModelAttribute` 也可以通过跨控制器共享一种方法`@ControllerAdvice`。有关更多详细信息，请参见“[控制器建议](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-controller-advice)”部分 。

`@ModelAttribute`方法具有灵活的方法签名。它们支持许多与`@RequestMapping`方法相同的参数（除了`@ModelAttribute`自身以及与请求主体相关的任何东西）。

以下示例使用一种`@ModelAttribute`方法：

爪哇

科特林

```java
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountRepository.findAccount(number));
    // add more ...
}
```

以下示例仅添加一个属性：

爪哇

科特林

```java
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountRepository.findAccount(number);
}
```

|      | 如果未明确指定名称，则根据类型选择默认名称，如javadoc中对的解释[`Conventions`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/Conventions.html)。您始终可以通过使用重载`addAttribute`方法或通过on的name属性`@ModelAttribute`（用于返回值）来分配显式名称。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

与Spring MVC不同，Spring WebFlux在模型中显式支持响应类型（例如`Mono<Account>`或`io.reactivex.Single<Account>`）。可以在`@RequestMapping`调用时将此类异步模型属性透明地解析（并更新模型）为其实际值，只要`@ModelAttribute`声明的参数没有包装即可，如以下示例所示：

爪哇

科特林

```java
@ModelAttribute
public void addAccount(@RequestParam String number) {
    Mono<Account> accountMono = accountRepository.findAccount(number);
    model.addAttribute("account", accountMono);
}

@PostMapping("/accounts")
public String handle(@ModelAttribute Account account, BindingResult errors) {
    // ...
}
```

另外，任何具有Reactive类型包装器的模型属性都将在视图渲染之前解析为其实际值（并更新了模型）。

您还可以`@ModelAttribute`在方法上用作方法级注释`@RequestMapping` ，在这种情况下，方法的返回值将`@RequestMapping`解释为模型属性。通常不需要这样做，因为这是HTML控制器的默认行为，除非返回值是a `String`，否则它将被解释为视图名称。`@ModelAttribute`还可以帮助自定义模型属性名称，如以下示例所示：

爪哇

科特林

```java
@GetMapping("/accounts/{id}")
@ModelAttribute("myAccount")
public Account handle() {
    // ...
    return account;
}
```

#### 1.4.5。 `DataBinder`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-initbinder)

`@Controller`或`@ControllerAdvice`类可以具有`@InitBinder`用于初始化的实例的方法`WebDataBinder`。这些依次用于：

- 将请求参数（即表单数据或查询）绑定到模型对象。
- 将`String`基于请求的请求值（例如请求参数，路径变量，标头，Cookie等）转换为控制器方法参数的目标类型。
- `String`呈现HTML表单时，将模型对象值格式化为值。

`@InitBinder`方法可以注册控制器特异性`java.beans.PropertyEditor`或Spring`Converter`和`Formatter`组件。另外，您可以使用 [WebFlux Java配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-conversion)来注册`Converter`和 `Formatter`键入全局共享`FormattingConversionService`。

`@InitBinder``@RequestMapping`除了`@ModelAttribute`（命令对象）参数外，方法还支持许多与方法相同的参数。通常，它们使用`WebDataBinder`用于注册的参数和`void`返回值进行声明。以下示例使用`@InitBinder`注释：

爪哇

科特林

```java
@Controller
public class FormController {

    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

|      | 使用`@InitBinder`注释。 |
| ---- | ----------------------- |
|      |                         |

另外，当`Formatter`通过shared使用基于设置时 `FormattingConversionService`，可以重新使用相同的方法并注册特定于控制器的`Formatter`实例，如以下示例所示：

爪哇

科特林

```java
@Controller
public class FormController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd")); 
    }

    // ...
}
```

|      | 添加自定义格式器（`DateFormatter`在本例中为a）。 |
| ---- | ------------------------------------------------ |
|      |                                                  |

#### 1.4.6。管理异常

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler)

`@Controller`和[@ControllerAdvice](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-controller-advice)类可以具有 `@ExceptionHandler`处理控制器方法异常的方法。下面的示例包括这样的处理程序方法：

爪哇

科特林

```java
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler 
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```

|      | 声明一个`@ExceptionHandler`。 |
| ---- | ----------------------------- |
|      |                               |

该异常可以与正在传播的顶级异常（即，直接`IOException`引发的异常）匹配，也可以与顶级包装器异常内的直接原因（例如，`IOException`包装在内`IllegalStateException`）相匹配 。

对于匹配的异常类型，最好将目标异常声明为方法参数，如前面的示例所示。或者，注释声明可以缩小异常类型以使其匹配。我们通常建议在参数签名中尽可能具体，并在`@ControllerAdvice`优先级上以相应的顺序声明您的主根异常映射 。有关详细信息，请参见[MVC部分](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler)。

|      | `@ExceptionHandler`WebFlux中 的方法支持与方法相同的方法参数和返回值`@RequestMapping`，但与请求正文和`@ModelAttribute`方法相关的参数除外。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

支持`@ExceptionHandler`在Spring WebFlux方法是由提供 `HandlerAdapter`的`@RequestMapping`方法。请参阅[`DispatcherHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-dispatcher-handler) 以获取更多详细信息。

##### REST API例外

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-rest-exceptions)

REST服务的常见要求是在响应正文中包含错误详细信息。Spring框架不会自动这样做，因为响应主体中错误详细信息的表示是特定于应用程序的。但是， `@RestController`可以使用`@ExceptionHandler`带有`ResponseEntity`返回值的方法来设置响应的状态和主体。也可以在`@ControllerAdvice`类中声明此类方法以将其全局应用。

|      | 请注意，Spring WebFlux与Spring MVC没有等效项 `ResponseEntityExceptionHandler`，因为WebFlux仅引发`ResponseStatusException` （或其子类），并且不需要将其转换为HTTP状态代码。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.4.7。控制器建议

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice)

Typically, the `@ExceptionHandler`, `@InitBinder`, and `@ModelAttribute` methods apply within the `@Controller` class (or class hierarchy) in which they are declared. If you want such methods to apply more globally (across controllers), you can declare them in a class annotated with `@ControllerAdvice` or `@RestControllerAdvice`.

`@ControllerAdvice` is annotated with `@Component`, which means that such classes can be registered as Spring beans through [component scanning](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-instantiating-container-scan). `@RestControllerAdvice` is a composed annotation that is annotated with both `@ControllerAdvice` and `@ResponseBody`, which essentially means `@ExceptionHandler` methods are rendered to the response body through message conversion (versus view resolution or template rendering).

On startup, the infrastructure classes for `@RequestMapping` and `@ExceptionHandler` methods detect Spring beans annotated with `@ControllerAdvice` and then apply their methods at runtime. Global `@ExceptionHandler` methods (from a `@ControllerAdvice`) are applied *after* local ones (from the `@Controller`). By contrast, global `@ModelAttribute` and `@InitBinder` methods are applied *before* local ones.

By default, `@ControllerAdvice` methods apply to every request (that is, all controllers), but you can narrow that down to a subset of controllers by using attributes on the annotation, as the following example shows:

Java

Kotlin

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

The selectors in the preceding example are evaluated at runtime and may negatively impact performance if used extensively. See the [`@ControllerAdvice`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html) javadoc for more details.

### 1.5. Functional Endpoints

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-fn)

Spring WebFlux包含WebFlux.fn，这是一个轻量级的函数编程模型，其中的函数用于路由和处理请求，而契约则是为不变性而设计的。它是基于注释的编程模型的替代方案，但可以在相同的[Reactive Core](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-spring-web)基础上运行。

#### 1.5.1。总览

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-fn-overview)

在WebFlux.fn中，HTTP请求使用`HandlerFunction`：处理，该函数接受 `ServerRequest`并返回延迟`ServerResponse`（即`Mono<ServerResponse>`）。请求和响应对象都具有不可变的协定，这些协定为JDK 8提供了对HTTP请求和响应的友好访问。 与基于注释的编程模型中方法`HandlerFunction`的主体等效`@RequestMapping`。

传入的请求使用：路由到处理函数，`RouterFunction`该函数接受`ServerRequest`并返回一个延迟的`HandlerFunction`（即`Mono<HandlerFunction>`）。当路由器功能匹配时，返回处理程序功能。否则为空Mono。 `RouterFunction`与`@RequestMapping`注解等效，但主要区别在于路由器功能不仅提供数据，还提供行为。

`RouterFunctions.route()` 提供了一个有助于构建路由器的路由器构建器，如以下示例所示：

爪哇

科特林

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.RequestPredicates.*;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> route = route()
    .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson)
    .GET("/person", accept(APPLICATION_JSON), handler::listPeople)
    .POST("/person", handler::createPerson)
    .build();


public class PersonHandler {

    // ...

    public Mono<ServerResponse> listPeople(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> createPerson(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> getPerson(ServerRequest request) {
        // ...
    }
}
```

运行a的一种方法`RouterFunction`是将其转换为an`HttpHandler`并通过内置[服务器适配器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler)之一进行安装：

- `RouterFunctions.toHttpHandler(RouterFunction)`
- `RouterFunctions.toHttpHandler(RouterFunction, HandlerStrategies)`

大多数应用程序可以通过WebFlux Java配置[运行](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn-running)，请参阅“[运行服务器”](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn-running)。

#### 1.5.2。处理函数

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-fn-handler-functions)

`ServerRequest` and `ServerResponse` are immutable interfaces that offer JDK 8-friendly access to the HTTP request and response. Both request and response provide [响应流](https://www.reactive-streams.org/) back pressure against the body streams. The request body is represented with a Reactor `Flux` or `Mono`. The response body is represented with any 响应流 `Publisher`, including `Flux` and `Mono`. For more on that, see [Reactive Libraries](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries).

##### ServerRequest

`ServerRequest` provides access to the HTTP method, URI, headers, and query parameters, while access to the body is provided through the `body` methods.

The following example extracts the request body to a `Mono<String>`:

Java

Kotlin

```java
Mono<String> string = request.bodyToMono(String.class);
```

以下示例将主体提取到`Flux<Person>`（或`Flow<Person>`Kotlin中的）主体，在该主体中，`Person`对象以某种序列化形式（例如JSON或XML ）进行解码：

爪哇

科特林

```java
Flux<Person> people = request.bodyToFlux(Person.class);
```

前面的示例是使用更通用的快捷方式，该快捷方式`ServerRequest.body(BodyExtractor)`接受`BodyExtractor`功能策略接口。实用程序类 `BodyExtractors`提供对许多实例的访问。例如，前面的示例也可以编写如下：

爪哇

科特林

```java
Mono<String> string = request.body(BodyExtractors.toMono(String.class));
Flux<Person> people = request.body(BodyExtractors.toFlux(Person.class));
```

下面的示例演示如何访问表单数据：

爪哇

科特林

```java
Mono<MultiValueMap<String, String> map = request.formData();
```

以下示例显示了如何以地图的形式访问多部分数据：

爪哇

科特林

```java
Mono<MultiValueMap<String, Part> map = request.multipartData();
```

下面的示例演示如何以流方式一次访问多个部分：

爪哇

科特林

```java
Flux<Part> parts = request.body(BodyExtractors.toParts());
```

##### 服务器响应

`ServerResponse`提供对HTTP响应的访问，并且由于它是不可变的，因此可以使用一种`build`方法来创建它。您可以使用构建器来设置响应状态，添加响应标题或提供正文。以下示例使用JSON内容创建200（确定）响应：

爪哇

科特林

```java
Mono<Person> person = ...
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person, Person.class);
```

下面的示例演示如何构建`Location`不带标头的201（已创建）响应：

爪哇

科特林

```java
URI location = ...
ServerResponse.created(location).build();
```

根据所使用的编解码器，可以传递提示参数以自定义主体的序列化或反序列化方式。例如，要指定一个[Jackson JSON视图](https://www.baeldung.com/jackson-json-view-annotation)：

爪哇

科特林

```java
ServerResponse.ok().hint(Jackson2CodecSupport.JSON_VIEW_HINT, MyJacksonView.class).body(...);
```

##### 处理程序类

我们可以将处理程序函数编写为lambda，如以下示例所示：

爪哇

科特林

```java
HandlerFunction<ServerResponse> helloWorld =
  request -> ServerResponse.ok().bodyValue("Hello World");
```

这很方便，但是在应用程序中我们需要多个功能，并且多个内联lambda可能会变得凌乱。因此，将相关的处理程序功能分组到一个处理程序类中很有用，该类具有与`@Controller`基于注释的应用程序类似的作用。例如，以下类公开了Reactive`Person`存储库：

爪哇

科特林

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;

public class PersonHandler {

    private final PersonRepository repository;

    public PersonHandler(PersonRepository repository) {
        this.repository = repository;
    }

    public Mono<ServerResponse> listPeople(ServerRequest request) { 
        Flux<Person> people = repository.allPeople();
        return ok().contentType(APPLICATION_JSON).body(people, Person.class);
    }

    public Mono<ServerResponse> createPerson(ServerRequest request) { 
        Mono<Person> person = request.bodyToMono(Person.class);
        return ok().build(repository.savePerson(person));
    }

    public Mono<ServerResponse> getPerson(ServerRequest request) { 
        int personId = Integer.valueOf(request.pathVariable("id"));
        return repository.getPerson(personId)
            .flatMap(person -> ok().contentType(APPLICATION_JSON).bodyValue(person))
            .switchIfEmpty(ServerResponse.notFound().build());
    }
}
```

|      | `listPeople`是一个处理程序函数，`Person`以JSON格式返回存储库中找到的所有对象。 |
| ---- | ------------------------------------------------------------ |
|      | `createPerson`是一个处理程序功能，用于存储`Person`请求正文中包含的新内容。请注意，`PersonRepository.savePerson(Person)`返回`Mono<Void>`：一个空的`Mono`，当从请求中读取并存储此人时，它将发出完成信号。因此`build(Publisher<Void>)`，当接收到该完成信号时（即，`Person`已保存时），我们使用该 方法发送响应。 |
|      | `getPerson`是一个处理程序函数，该函数返回由`id`path变量标识的一个人。我们`Person`从存储库中检索到该内容，并创建一个JSON响应（如果找到）。如果未找到，我们`switchIfEmpty(Mono<T>)`将返回404 Not Found响应。 |

##### 验证方式

功能端点可以使用Spring的[验证工具](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)将验证应用于请求主体。例如，给定一个定制的Spring [Validator](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation)实现`Person`：

爪哇

科特林

```java
public class PersonHandler {

    private final Validator validator = new PersonValidator(); 

    // ...

    public Mono<ServerResponse> createPerson(ServerRequest request) {
        Mono<Person> person = request.bodyToMono(Person.class).doOnNext(this::validate); 
        return ok().build(repository.savePerson(person));
    }

    private void validate(Person person) {
        Errors errors = new BeanPropertyBindingResult(person, "person");
        validator.validate(person, errors);
        if (errors.hasErrors()) {
            throw new ServerWebInputException(errors.toString()); 
        }
    }
}
```

|      | 创建`Validator`实例。 |
| ---- | --------------------- |
|      | 应用验证。            |
|      | 引发400响应的异常。   |

处理程序还可以通过创建和注入`Validator`基于的全局实例来使用标准bean验证API（JSR-303）`LocalValidatorFactoryBean`。请参阅[春季验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation)。

#### 1.5.3。 `RouterFunction`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-fn-router-functions)

路由器功能用于将请求路由到相应的`HandlerFunction`。通常，您不是自己编写路由器功能，而是使用`RouterFunctions`实用程序类上的方法 创建一个。 `RouterFunctions.route()`（无参数）为您提供了一个流畅的生成器来创建路由器功能，而`RouterFunctions.route(RequestPredicate, HandlerFunction)`提供了一种直接的方式来创建路由器。

通常，建议使用`route()`构建器，因为它为典型的映射场景提供了便捷的捷径，而无需发现静态导入。例如，路由器功能构建器提供了`GET(String, HandlerFunction)`为GET请求创建映射的方法。和`POST(String, HandlerFunction)`POST。

除了基于HTTP方法的映射外，路由构建器还提供了一种在映射到请求时引入其他谓词的方法。对于每个HTTP方法，都有一个以a`RequestPredicate`作为参数的重载变体，不过可以表达其他约束。

##### 谓词

您可以编写自己的`RequestPredicate`，但是`RequestPredicates`实用程序类根据请求路径，HTTP方法，内容类型等提供常用的实现。以下示例使用请求谓词基于`Accept` 标头创建约束：

爪哇

科特林

```java
RouterFunction<ServerResponse> route = RouterFunctions.route()
    .GET("/hello-world", accept(MediaType.TEXT_PLAIN),
        request -> ServerResponse.ok().bodyValue("Hello World")).build();
```

您可以使用以下命令组合多个请求谓词：

- `RequestPredicate.and(RequestPredicate)` -两者必须匹配。
- `RequestPredicate.or(RequestPredicate)` -两者都可以匹配。

的许多谓词`RequestPredicates`组成。例如，`RequestPredicates.GET(String)`由`RequestPredicates.method(HttpMethod)` 和组成`RequestPredicates.path(String)`。上面显示的示例还使用两个请求谓词，因为构建器在`RequestPredicates.GET`内部使用 并将其与`accept`谓词组合在一起。

##### 路线

路由器功能按顺序评估：如果第一个路由不匹配，则评估第二个路由，依此类推。因此，在通用路由之前声明更具体的路由是有意义的。请注意，此行为不同于基于注释的编程模型，在该模型中，将自动选择“最特定”的控制器方法。

使用路由器功能生成器时，所有定义的路由都组成一个`RouterFunction`从中返回的路由 `build()`。还有其他方法可以将多个路由器功能组合在一起：

- `add(RouterFunction)`在`RouterFunctions.route()`建造者上
- `RouterFunction.and(RouterFunction)`
- `RouterFunction.andRoute(RequestPredicate, HandlerFunction)` —`RouterFunction.and()`嵌套的快捷方式 `RouterFunctions.route()`。

以下示例显示了四种路线的组成：

爪哇

科特林

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.RequestPredicates.*;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> otherRoute = ...

RouterFunction<ServerResponse> route = route()
    .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson) 
    .GET("/person", accept(APPLICATION_JSON), handler::listPeople) 
    .POST("/person", handler::createPerson) 
    .add(otherRoute) 
    .build();
```

|      | `GET /person/{id}`与`Accept`匹配JSON头被路由到 `PersonHandler.getPerson` |
| ---- | ------------------------------------------------------------ |
|      | `GET /person`与`Accept`匹配JSON头被路由到 `PersonHandler.listPeople` |
|      | `POST /person`没有其他谓词的映射到 `PersonHandler.createPerson`，并且 |
|      | `otherRoute` 是在其他地方创建并添加到所建立路由的路由器功能。 |

##### 嵌套路线

一组路由器功能通常具有共享谓词，例如共享路径。在上面的示例中，共享谓词将是与其中`/person`三个路由使用的match匹配的路径谓词。使用注释时，您可以通过使用`@RequestMapping`映射到的类型级别的注释来 删除此重复项`/person`。在WebFlux.fn中，可以通过`path`路由器功能构建器上的方法共享路径谓词。例如，可以通过使用嵌套路由以以下方式改进上面示例的最后几行：

爪哇

科特林

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", builder -> builder 
        .GET("/{id}", accept(APPLICATION_JSON), handler::getPerson)
        .GET(accept(APPLICATION_JSON), handler::listPeople)
        .POST("/person", handler::createPerson))
    .build();
```

|      | 请注意，的第二个参数`path`是使用路由器构建器的使用者。 |
| ---- | ------------------------------------------------------ |
|      |                                                        |

尽管基于路径的嵌套是最常见的，但是您可以通过使用`nest`构建器上的方法来嵌套在任何种类的谓词上。上面的内容仍然包含一些共享头`Accept`谓词形式的重复项。通过结合使用该`nest`方法，我们可以进一步改进`accept`：

爪哇

科特林

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET(handler::listPeople))
        .POST("/person", handler::createPerson))
    .build();
```

#### 1.5.4。运行服务器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-fn-running)

如何在HTTP服务器中运行路由器功能？一个简单的选项是`HttpHandler`使用以下方法之一将路由器功能转换为：

- `RouterFunctions.toHttpHandler(RouterFunction)`
- `RouterFunctions.toHttpHandler(RouterFunction, HandlerStrategies)`

然后，您可以`HttpHandler`按照[HttpHandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-httphandler)的说明使用特定服务器返回的服务器适配器 。

A more typical option, also used by Spring Boot, is to run with a [`DispatcherHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-dispatcher-handler)-based setup through the [WebFlux Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config), which uses Spring configuration to declare the components required to process requests. The WebFlux Java configuration declares the following infrastructure components to support functional endpoints:

- `RouterFunctionMapping`: Detects one or more `RouterFunction<?>` beans in the Spring configuration, combines them through `RouterFunction.andOther`, and routes requests to the resulting composed `RouterFunction`.
- `HandlerFunctionAdapter`: Simple adapter that lets `DispatcherHandler` invoke a `HandlerFunction` that was mapped to a request.
- `ServerResponseResultHandler`: Handles the result from the invocation of a `HandlerFunction` by invoking the `writeTo` method of the `ServerResponse`.

前面的组件使功能端点适合于`DispatcherHandler`请求处理生命周期，并且（如果有）声明的控制器也可以（可能）与带注释的控制器并排运行。这也是Spring Boot WebFlux启动器启用功能端点的方式。

以下示例显示了WebFlux Java配置（有关如何运行它，请参见 [DispatcherHandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-dispatcher-handler)）：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Bean
    public RouterFunction<?> routerFunctionA() {
        // ...
    }

    @Bean
    public RouterFunction<?> routerFunctionB() {
        // ...
    }

    // ...

    @Override
    public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
        // configure message conversion...
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // configure CORS...
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // configure view resolution for HTML rendering...
    }
}
```

#### 1.5.5。过滤处理程序功能

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-fn-handler-filter-function)

您可以通过使用过滤处理器功能`before`，`after`或`filter`在路由功能生成器方法。使用注释，您可以通过使用实现类似的功能`@ControllerAdvice`，一个`ServletFilter`，或两者兼而有之。该过滤器将应用于构建器构建的所有路由。这意味着在嵌套路由中定义的过滤器不适用于“顶级”路由。例如，考虑以下示例：

爪哇

科特林

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET(handler::listPeople)
            .before(request -> ServerRequest.from(request) 
                .header("X-RequestHeader", "Value")
                .build()))
        .POST("/person", handler::createPerson))
    .after((request, response) -> logResponse(response)) 
    .build();
```

|      | `before`添加自定义请求标头的过滤器仅应用于两个GET路由。 |
| ---- | ------------------------------------------------------- |
|      | `after`记录响应的过滤器将应用于所有路由，包括嵌套路由。 |

在`filter`路由器上构建器方法需要`HandlerFilterFunction`：一个函数，接受`ServerRequest`和`HandlerFunction`并返回`ServerResponse`。handler函数参数代表链中的下一个元素。这通常是路由到的处理程序，但是如果应用了多个，它也可以是另一个过滤器。

现在，我们可以在路由中添加一个简单的安全过滤器，假设我们有一个`SecurityManager`可以确定是否允许特定路径的。以下示例显示了如何执行此操作：

爪哇

科特林

```java
SecurityManager securityManager = ...

RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET(handler::listPeople))
        .POST("/person", handler::createPerson))
    .filter((request, next) -> {
        if (securityManager.allowAccessTo(request.path())) {
            return next.handle(request);
        }
        else {
            return ServerResponse.status(UNAUTHORIZED).build();
        }
    })
    .build();
```

前面的示例演示了调用`next.handle(ServerRequest)`是可选的。我们只允许在允许访问时运行处理程序函数。

除了使用`filter`路由器功能构建器上的方法之外，还可以通过将过滤器应用于现有路由器功能`RouterFunction.filter(HandlerFilterFunction)`。

|      | 通过专用CORS支持功能端点 [`CorsWebFilter`](https://docs.spring.io/spring-framework/docs/current/reference/html/webflux-cors.html#webflux-cors-webfilter)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.6。URI链接

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-uri-building)

本节描述了Spring框架中用于准备URI的各种选项。

#### 1.6.1。UriComponents

Spring MVC和Spring WebFlux

`UriComponentsBuilder` 有助于从带有变量的URI模板构建URI，如以下示例所示：

爪哇

科特林

```java
UriComponents uriComponents = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")  
        .queryParam("q", "{q}")  
        .encode() 
        .build(); 

URI uri = uriComponents.expand("Westin", "123").toUri();  
```

|      | 带有URI模板的静态工厂方法。      |
| ---- | -------------------------------- |
|      | 添加或替换URI组件。              |
|      | 请求对URI模板和URI变量进行编码。 |
|      | 建立一个`UriComponents`。        |
|      | 展开变量并获得`URI`。            |

可以将前面的示例合并为一个链，并通过进行缩短`buildAndExpand`，如以下示例所示：

爪哇

科特林

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("Westin", "123")
        .toUri();
```

您可以通过直接转到URI（这意味着编码）来进一步缩短它，如以下示例所示：

爪哇

科特林

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

您可以使用完整的URI模板进一步缩短它，如以下示例所示：

爪哇

科特林

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}?q={q}")
        .build("Westin", "123");
```

#### 1.6.2。UriBuilder

Spring MVC和Spring WebFlux

[`UriComponentsBuilder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#web-uricomponents)实施`UriBuilder`。您可以`UriBuilder`依次使用创建 一个`UriBuilderFactory`。一起，`UriBuilderFactory`并 `UriBuilder`提供一个可插入的机构以从URI模板，基于共享的配置，构建的URI诸如基本URL，编码偏好，以及其他细节。

您可以配置`RestTemplate`和`WebClient`使用`UriBuilderFactory` 自定义的URI的准备。`DefaultUriBuilderFactory`是内部`UriBuilderFactory`使用`UriComponentsBuilder`并公开共享配置选项的默认实现。

以下示例显示了如何配置`RestTemplate`：

爪哇

科特林

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);
```

以下示例配置了`WebClient`：

爪哇

科特林

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

另外，您也可以`DefaultUriBuilderFactory`直接使用。它与使用类似， `UriComponentsBuilder`但不是静态工厂方法，它是一个包含配置和首选项的实际实例，如以下示例所示：

爪哇

科特林

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory uriBuilderFactory = new DefaultUriBuilderFactory(baseUrl);

URI uri = uriBuilderFactory.uriString("/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

#### 1.6.3。URI编码

Spring MVC和Spring WebFlux

`UriComponentsBuilder` 在两个级别公开编码选项：

- [UriComponentsBuilder＃encode（）](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/UriComponentsBuilder.html#encode--)：首先对URI模板进行预编码，然后在扩展时严格对URI变量进行编码。
- [UriComponents＃encode（）](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/UriComponents.html#encode--)：扩展URI变量*后，*[对](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/util/UriComponents.html#encode--)URI组件*进行*编码。

这两个选项都使用转义的八位字节替换非ASCII和非法字符。但是，第一个选项还会替换出现在URI变量中的具有保留含义的字符。

|      | 考虑“;”，这在路径上是合法的，但具有保留的含义。第一个选项代替“;” URI变量中带有“％3B”，但URI模板中没有。相比之下，第二个选项永远不会替换“;”，因为它是路径中的合法字符。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在大多数情况下，第一个选项可能会产生预期的结果，因为它将URI变量视为要完全编码的不透明数据，而选项2仅在URI变量有意包含保留字符的情况下才有用。

以下示例使用第一个选项：

爪哇

科特林

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("New York", "foo+bar")
        .toUri();

// Result is "/hotel%20list/New%20York?q=foo%2Bbar"
```

您可以通过直接转到URI（这意味着编码）来缩短前面的示例，如以下示例所示：

爪哇

科特林

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .build("New York", "foo+bar")
```

您可以使用完整的URI模板进一步缩短它，如以下示例所示：

爪哇

科特林

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}?q={q}")
        .build("New York", "foo+bar")
```

在`WebClient`与`RestTemplate`扩大和编码URI通过内部模板`UriBuilderFactory`策略。两者都可以使用自定义策略进行配置。如下例所示：

爪哇

科特林

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl)
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

// Customize the RestTemplate..
RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);

// Customize the WebClient..
WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

该`DefaultUriBuilderFactory`实现在`UriComponentsBuilder`内部使用以扩展和编码URI模板。作为工厂，它提供了一个位置，可以根据以下一种编码模式来配置编码方法：

- `TEMPLATE_AND_VALUES`：使用`UriComponentsBuilder#encode()`，对应于先前列表中的第一个选项，对URI模板进行预编码，并在扩展时严格编码URI变量。
- `VALUES_ONLY`：不对URI模板进行编码，而是在将URI变量`UriUtils#encodeUriUriVariables`扩展到模板之前对其进行严格编码。
- `URI_COMPONENT`: Uses `UriComponents#encode()`, corresponding to the second option in the earlier list, to encode URI component value *after* URI variables are expanded.
- `NONE`: No encoding is applied.

The `RestTemplate` is set to `EncodingMode.URI_COMPONENT` for historic reasons and for backwards compatibility. The `WebClient` relies on the default value in `DefaultUriBuilderFactory`, which was changed from `EncodingMode.URI_COMPONENT` in 5.0.x to `EncodingMode.TEMPLATE_AND_VALUES` in 5.1.

### 1.7. CORS

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors)

Spring WebFlux lets you handle CORS (Cross-Origin Resource Sharing). This section describes how to do so.

#### 1.7.1. Introduction

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-intro)

出于安全原因，浏览器禁止AJAX调用当前来源以外的资源。例如，您可以在一个标签页中拥有您的银行帐户，而在另一个标签页中拥有evil.com。来自evil.com的脚本不应使用您的凭据向您的银行API发出AJAX请求，例如，从您的帐户中提取资金！

跨域资源共享（CORS）是 由[大多数浏览器](https://caniuse.com/#feat=cors)实现的[W3C规范](https://www.w3.org/TR/cors/)，可让您指定授权哪种类型的跨域请求，而不是使用基于IFRAME或JSONP的安全性较低且功能较弱的变通办法。

#### 1.7.2。处理中

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-processing)

CORS规范区分飞行前，简单和实际请求。要了解CORS的工作原理，您可以阅读 [本文](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)以及其他内容，或者参阅规范以获取更多详细信息。

Spring WebFlux`HandlerMapping`实现提供对CORS的内置支持。成功将请求映射到处理程序后，会`HandlerMapping`检查给定请求和处理程序的CORS配置并采取进一步的措施。飞行前请求直接处理，而简单和实际的CORS请求被拦截，验证并设置了所需的CORS响应标头。

为了启用跨域请求（即，`Origin`标头存在并且与请求的主机不同），您需要具有一些显式声明的CORS配置。如果找不到匹配的CORS配置，则飞行前请求将被拒绝。没有将CORS标头添加到简单和实际CORS请求的响应中，因此，浏览器拒绝了它们。

每个`HandlerMapping`都可以 使用基于URL模式的映射进行 单独[配置](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/handler/AbstractHandlerMapping.html#setCorsConfigurations-java.util.Map-)`CorsConfiguration`。在大多数情况下，应用程序使用WebFlux Java配置声明此类映射，从而导致将单个全局映射传递给所有`HandlerMapping`实现。

您可以将全局CORS配置`HandlerMapping`与更细粒度的处理程序级CORS配置结合使用。例如，带注释的控制器可以使用类或方法级的`@CrossOrigin`注释（其他处理程序可以实现 `CorsConfigurationSource`）。

组合全局和本地配置的规则通常是相加的，例如，所有全局和所有本地来源。对于只能接受单个值的那些属性，例如`allowCredentials`和`maxAge`，本地将覆盖全局值。请参阅 [`CorsConfiguration#combine(CorsConfiguration)`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/cors/CorsConfiguration.html#combine-org.springframework.web.cors.CorsConfiguration-) 以获取更多详细信息。

|      | 要从源中了解更多信息或进行高级自定义，请参阅：`CorsConfiguration``CorsProcessor` 和 `DefaultCorsProcessor``AbstractHandlerMapping` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.7.3。 `@CrossOrigin`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-controller)

该[`@CrossOrigin`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html) 注释能够对带注释的控制器方法跨域请求，如下面的示例所示：

爪哇

科特林

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Mono<Account> retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public Mono<Void> remove(@PathVariable Long id) {
        // ...
    }
}
```

默认情况下，`@CrossOrigin`允许：

- 所有起源。
- 所有标题。
- 控制器方法映射到的所有HTTP方法。

`allowCredentials`默认情况下不会启用，因为这会建立一个信任级别，该级别公开敏感的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。启用后，`allowOrigins`必须将其设置为一个或多个特定域（而不是特殊值`"*"`），或者`allowOriginPatterns`可以使用该属性来匹配动态的一组原点。

`maxAge` 设置为30分钟。

`@CrossOrigin`在类级别也受支持，并且由所有方法继承。下面的示例指定一个特定的域并将其设置`maxAge`为一个小时：

爪哇

科特林

```java
@CrossOrigin(origins = "https://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @GetMapping("/{id}")
    public Mono<Account> retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public Mono<Void> remove(@PathVariable Long id) {
        // ...
    }
}
```

您可以`@CrossOrigin`在类和方法级别上使用，如以下示例所示：

爪哇

科特林

```java
@CrossOrigin(maxAge = 3600) 
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("https://domain2.com") 
    @GetMapping("/{id}")
    public Mono<Account> retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public Mono<Void> remove(@PathVariable Long id) {
        // ...
    }
}
```

|      | 使用`@CrossOrigin`类级别。   |
| ---- | ---------------------------- |
|      | 使用`@CrossOrigin`方法级别。 |

#### 1.7.4。全局配置

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-global)

除了细粒度的控制器方法级配置外，您可能还想定义一些全局CORS配置。您可以`CorsConfiguration` 在任何上分别设置基于URL的映射`HandlerMapping`。但是，大多数应用程序都使用WebFlux Java配置来执行此操作。

默认情况下，全局配置启用以下功能：

- 所有起源。
- 所有标题。
- `GET`，`HEAD`和`POST`方法。

`allowedCredentials`默认情况下不会启用，因为这会建立一个信任级别，该级别公开敏感的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。启用后，`allowOrigins`必须将其设置为一个或多个特定域（而不是特殊值`"*"`），或者`allowOriginPatterns`可以使用该属性来匹配动态的一组原点。

`maxAge` 设置为30分钟。

要在WebFlux Java配置中启用CORS，可以使用`CorsRegistry`回调，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

#### 1.7.5。CORS`WebFilter`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-filter)

您可以通过内置应用CORS支持 [`CorsWebFilter`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/cors/reactive/CorsWebFilter.html)，这非常适合[功能性端点](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn)。

|      | 如果您尝试在`CorsFilter`Spring Security中使用，请记住Spring Security [内置了](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors) 对CORS的[支持](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要配置过滤器，可以声明一个`CorsWebFilter`bean并将其传递 `CorsConfigurationSource`给其构造函数，如以下示例所示：

爪哇

科特林

```java
@Bean
CorsWebFilter corsFilter() {

    CorsConfiguration config = new CorsConfiguration();

    // Possibly...
    // config.applyPermitDefaultValues()

    config.setAllowCredentials(true);
    config.addAllowedOrigin("https://domain1.com");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);

    return new CorsWebFilter(source);
}
```

### 1.8。网络安全

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-web-security)

在[Spring Security的](https://projects.spring.io/spring-security/)项目提供了保护Web应用程序免受恶意攻击的支持。请参阅Spring Security参考文档，包括：

- [WebFlux安全](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#jc-webflux)
- [WebFlux测试支持](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#test-webflux)
- [CSRF保护](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#csrf)
- [安全响应标头](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#headers)

### 1.9。查看技术

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view)

Spring WebFlux中视图技术的使用是可插入的。是否决定使用Thymeleaf，FreeMarker或其他某种视图技术，主要取决于配置更改。本章介绍与Spring WebFlux集成的视图技术。我们假设您已经熟悉[View Resolution](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-viewresolution)。

#### 1.9.1。胸腺

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-thymeleaf)

Thymeleaf是一种现代的服务器端Java模板引擎，它强调可以通过双击在浏览器中预览的自然HTML模板，这对于独立处理UI模板（例如，由设计人员）非常有用，而无需使用正在运行的服务器。Thymeleaf提供了广泛的功能集，并且正在积极地开发和维护。有关更完整的介绍，请参见 [Thymeleaf](https://www.thymeleaf.org/)项目主页。

Thymeleaf与Spring WebFlux的集成由Thymeleaf项目管理。配置涉及几个bean声明，如 `SpringResourceTemplateResolver`，`SpringWebFluxTemplateEngine`和 `ThymeleafReactiveViewResolver`。有关更多详细信息，请参见 [Thymeleaf + Spring](https://www.thymeleaf.org/documentation.html)和WebFlux集成 [公告](http://forum.thymeleaf.org/Thymeleaf-3-0-8-JUST-PUBLISHED-td4030687.html)。

#### 1.9.2。FreeMarker

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-freemarker)

[Apache FreeMarker](https://freemarker.apache.org/)是一个模板引擎，用于生成从HTML到电子邮件等的任何类型的文本输出。Spring框架具有内置的集成，可以将Spring WebFlux与FreeMarker模板一起使用。

##### 查看配置

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-freemarker-contextconfig)

以下示例显示了如何将FreeMarker配置为一种视图技术：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.freeMarker();
    }

    // Configure FreeMarker...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("classpath:/templates/freemarker");
        return configurer;
    }
}
```

您的模板需要存储在由所指定的目录`FreeMarkerConfigurer`中，如上例所示。给定上述配置，如果您的控制器返回视图名称`welcome`，则解析器将查找 `classpath:/templates/freemarker/welcome.ftl`模板。

##### FreeMarker配置

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-views-freemarker)

您可以通过`Configuration`在bean上设置适当的bean属性，将FreeMarker的“设置”和“ SharedVariables”直接传递给FreeMarker 对象（由Spring管理）`FreeMarkerConfigurer`。该`freemarkerSettings`属性需要一个`java.util.Properties`对象，而该`freemarkerVariables`属性需要一个 `java.util.Map`。以下示例显示了如何使用`FreeMarkerConfigurer`：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    // ...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        Map<String, Object> variables = new HashMap<>();
        variables.put("xml_escape", new XmlEscape());

        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("classpath:/templates");
        configurer.setFreemarkerVariables(variables);
        return configurer;
    }
}
```

有关设置和变量应用于`Configuration`对象的详细信息，请参见FreeMarker文档。

##### 表格处理

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-freemarker-forms)

Spring provides a tag library for use in JSPs that contains, among others, a `<spring:bind/>` element. This element primarily lets forms display values from form-backing objects and show the results of failed validations from a `Validator` in the web or business tier. Spring also has support for the same functionality in FreeMarker, with additional convenience macros for generating form input elements themselves.

###### The Bind Macros

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-bind-macros)

A standard set of macros are maintained within the `spring-webflux.jar` file for FreeMarker, so they are always available to a suitably configured application.

Spring模板库中定义的某些宏被视为内部（私有）宏，但是在宏定义中不存在这种范围，使所有宏对调用代码和用户模板可见。以下各节仅关注您需要从模板内直接调用的宏。如果您希望直接查看宏代码，则将调用该文件`spring.ftl`并将其包含在 `org.springframework.web.reactive.result.view.freemarker`包中。

有关绑定支持的更多详细信息，请参见Spring MVC的[简单绑定](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-simple-binding)。

###### 表格巨集

有关Spring对FreeMarker模板的表单宏支持的详细信息，请查阅Spring MVC文档的以下部分。

- [输入宏](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-views-form-macros)
- [输入栏位](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-views-form-macros-input)
- [选择字段](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-views-form-macros-select)
- [HTML转义](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-views-form-macros-html-escaping)

#### 1.9.3。脚本视图

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-script)

Spring框架具有内置的集成，可以将Spring WebFlux与可以在[JSR-223](https://www.jcp.org/en/jsr/detail?id=223) Java脚本引擎之上运行的任何模板库一起使用 。下表显示了我们在不同脚本引擎上测试过的模板库：

| 脚本库                                                       | 脚本引擎                                               |
| :----------------------------------------------------------- | :----------------------------------------------------- |
| [车把](https://handlebarsjs.com/)                            | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [胡子](https://mustache.github.io/)                          | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [反应](https://facebook.github.io/react/)                    | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [EJS](https://www.embeddedjs.com/)                           | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [ERB](https://www.stuartellis.name/articles/erb/)            | [红宝石](https://www.jruby.org/)                       |
| [字符串模板](https://docs.python.org/2/library/string.html#template-strings) | [吉顿](https://www.jython.org/)                        |
| [Kotlin脚本模板](https://github.com/sdeleuze/kotlin-script-templating) | [科特林](https://kotlinlang.org/)                      |

|      | 集成任何其他脚本引擎的基本规则是，它必须实现 `ScriptEngine`和`Invocable`接口。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 要求

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-script-dependencies)

您需要在类路径上具有脚本引擎，其细节因脚本引擎而异：

- 在[犀牛](https://openjdk.java.net/projects/nashorn/)的JavaScript引擎提供的Java 8+。强烈建议使用可用的最新更新版本。
- 应该将[JRuby](https://www.jruby.org/)添加为对Ruby支持的依赖。
- 应该将[Jython](https://www.jython.org/)添加为对Python支持的依赖项。
- `org.jetbrains.kotlin:kotlin-script-util`依赖关系和`META-INF/services/javax.script.ScriptEngineFactory` 包含`org.jetbrains.kotlin.script.jsr223.KotlinJsr223JvmLocalScriptEngineFactory` 一行的文件应添加以支持Kotlin脚本。有关更多详细信息，请参 [见此示例](https://github.com/sdeleuze/kotlin-script-templating)。

您需要具有脚本模板库。针对Javascript的一种方法是通过[WebJars](https://www.webjars.org/)。

##### 脚本模板

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-script-integrate)

您可以声明一个`ScriptTemplateConfigurer`bean，以指定要使用的脚本引擎，要加载的脚本文件，调用呈现模板的函数等等。以下示例使用Mustache模板和Nashorn JavaScript引擎：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("mustache.js");
        configurer.setRenderObject("Mustache");
        configurer.setRenderFunction("render");
        return configurer;
    }
}
```

该`render`函数使用以下参数调用：

- `String template`：模板内容
- `Map model`：视图模型
- `RenderingContext renderingContext`: The [`RenderingContext`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/servlet/view/script/RenderingContext.html) that gives access to the application context, the locale, the template loader, and the URL (since 5.0)

`Mustache.render()` is natively compatible with this signature, so you can call it directly.

If your templating technology requires some customization, you can provide a script that implements a custom render function. For example, [Handlerbars](https://handlebarsjs.com/) needs to compile templates before using them and requires a [polyfill](https://en.wikipedia.org/wiki/Polyfill) in order to emulate some browser facilities not available in the server-side script engine. The following example shows how to set a custom render function:

Java

Kotlin

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("polyfill.js", "handlebars.js", "render.js");
        configurer.setRenderFunction("render");
        configurer.setSharedEngine(false);
        return configurer;
    }
}
```

|      | Setting the `sharedEngine` property to `false` is required when using non-thread-safe script engines with templating libraries not designed for concurrency, such as Handlebars or React running on Nashorn. In that case, Java SE 8 update 60 is required, due to [this bug](https://bugs.openjdk.java.net/browse/JDK-8076099), but it is generally recommended to use a recent Java SE patch release in any case. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`polyfill.js` defines only the `window` object needed by Handlebars to run properly, as the following snippet shows:

```javascript
var window = {};
```

此基本`render.js`实现在使用模板之前先对其进行编译。生产就绪的实现还应该存储和重用缓存的模板或预编译的模板。这可以在脚本端以及您需要的任何自定义（例如，管理模板引擎配置）上完成。以下示例显示了如何编译模板：

```javascript
function render(template, model) {
    var compiledTemplate = Handlebars.compile(template);
    return compiledTemplate(model);
}
```

请查看Spring Framework单元测试， [Java](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux/src/test/java/org/springframework/web/reactive/result/view/script)和 [资源](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux/src/test/resources/org/springframework/web/reactive/result/view/script)，以获取更多配置示例。

#### 1.9.4。JSON和XML

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view-jackson)

出于[内容协商的](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multiple-representations)目的，根据客户端请求的内容类型，能够在使用HTML模板呈现模型或以其他格式（例如JSON或XML）呈现模型之间进行切换非常有用。为了支持这样做，春天WebFlux提供`HttpMessageWriterView`，您可以在任何可用的使用插件 [编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)的`spring-web`，比如`Jackson2JsonEncoder`，`Jackson2SmileEncoder`或者`Jaxb2XmlEncoder`。

与其他视图技术不同，`HttpMessageWriterView`不需要视图，`ViewResolver` 而是将其[配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-view-resolvers)为默认视图。您可以配置一个或多个这样的默认视图，包装不同的`HttpMessageWriter`实例或`Encoder`实例。在运行时使用与请求的内容类型匹配的内容。

在大多数情况下，模型包含多个属性。要确定要序列化的对象，可以配置`HttpMessageWriterView`要用于渲染的model属性的名称。如果模型仅包含一个属性，则使用该属性。

### 1.10。HTTP缓存

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching)

HTTP缓存可以显着提高Web应用程序的性能。HTTP缓存围绕`Cache-Control`响应标头和后续条件请求标头（例如`Last-Modified`和）展开`ETag`。`Cache-Control`建议私有（例如浏览器）和公共（例如代理）缓存如何缓存和重复使用响应。一个`ETag`头用于使没有身体可能导致一个304（NOT_MODIFIED）一个条件请求，如果内容没有改变。`ETag`可以看作是`Last-Modified`标题的更复杂的后继者。

本节描述了Spring WebFlux中与HTTP缓存相关的选项。

#### 1.10.1。 `CacheControl`

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-cachecontrol)

[`CacheControl`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/CacheControl.html)提供对配置与`Cache-Control`标头相关的设置的支持，并在许多地方作为参数被接受：

- [控制器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-caching-etag-lastmodified)
- [静态资源](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-caching-static-resources)

尽管[RFC 7234](https://tools.ietf.org/html/rfc7234#section-5.2.2)描述了`Cache-Control`响应标头的所有可能的指令，但是该`CacheControl`类型采用面向用例的方法，重点关注常见方案，如以下示例所示：

爪哇

科特林

```java
// Cache for an hour - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// Prevent caching - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// Cache for ten days in public and private caches,
// public caches should not transform the response
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS).noTransform().cachePublic();
```

#### 1.10.2。控制器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-etag-lastmodified)

控制器可以添加对HTTP缓存的显式支持。我们建议您这样做，因为需要先计算资源的 `lastModified`or`ETag`值，然后才能将其与条件请求标头进行比较。控制器可以将`ETag`和`Cache-Control` 设置添加到`ResponseEntity`，如以下示例所示：

爪哇

科特林

```java
@GetMapping("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
            .ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
            .eTag(version) // lastModified is also available
            .body(book);
}
```

如果与条件请求标头的比较表明内容未更改，则前面的示例发送带有空主体的304（NOT_MODIFIED）响应。否则， `ETag`和`Cache-Control`标头将添加到响应中。

您还可以在控制器中针对条件请求标头进行检查，如以下示例所示：

爪哇

科特林

```java
@RequestMapping
public String myHandleMethod(ServerWebExchange exchange, Model model) {

    long eTag = ... 

    if (exchange.checkNotModified(eTag)) {
        return null; 
    }

    model.addAttribute(...); 
    return "myViewName";
}
```

|      | 特定于应用程序的计算。                            |
| ---- | ------------------------------------------------- |
|      | 响应已设置为304（NOT_MODIFIED）。无需进一步处理。 |
|      | 继续进行请求处理。                                |

有三种变体，用于根据`eTag`值和`lastModified` /或值检查条件请求。对于条件`GET`和`HEAD`请求，可以将响应设置为304（NOT_MODIFIED）。对于有条件的`POST`，`PUT`和`DELETE`，可以改为将响应设置为412（PRECONDITION_FAILED）以防止并发修改。

#### 1.10.3。静态资源

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-static-resources)

您应使用`Cache-Control`和条件响应标头来提供静态资源，以实现最佳性能。请参阅“配置[静态资源](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-static-resources)”部分。

### 1.11。WebFlux配置

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config)

WebFlux Java配置声明使用带注释的控制器或功能端点来声明处理请求所必需的组件，并且它提供了用于自定义配置的API。这意味着您不需要了解Java配置创建的底层bean。但是，如果您想了解它们，则可以`WebFluxConfigurationSupport`在[Special Bean Types中](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-special-bean-types)看到它们，或进一步了解它们的含义。

要获得配置API中没有的更高级的自定义设置，您可以通过[高级配置模式](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-advanced-java)来完全控制 [配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-advanced-java)。

#### 1.11.1。启用WebFlux配置

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-enable)

您可以`@EnableWebFlux`在Java配置中使用注释，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig {
}
```

前面的示例注册了许多Spring WebFlux[基础结构Bean，](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-special-bean-types)并适应了类路径上可用的依赖关系-JSON ，XML等。

#### 1.11.2。WebFlux配置API

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-customize)

在Java配置中，您可以实现该`WebFluxConfigurer`接口，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    // Implement configuration methods...
}
```

#### 1.11.3。转换，格式化

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-conversion)

默认情况下，将安装各种数字和日期类型的格式化程序，并支持通过`@NumberFormat`和`@DateTimeFormat`在字段上进行自定义。

要在Java配置中注册自定义格式器和转换器，请使用以下命令：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }

}
```

默认情况下，Spring WebFlux在解析和格式化日期值时会考虑请求区域设置。这适用于使用“输入”表单字段将日期表示为字符串的表单。但是，对于“日期”和“时间”表单字段，浏览器使用HTML规范中定义的固定格式。在这种情况下，日期和时间格式可以按以下方式自定义：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setUseIsoFormat(true);
        registrar.registerFormatters(registry);
    }
}
```

|      | 有关何时使用实现的更多信息，请参见[`FormatterRegistrar`SPI](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format-FormatterRegistrar-SPI) 和。 `FormattingConversionServiceFactoryBean``FormatterRegistrar` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.11.4。验证方式

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-validation)

默认情况下，如果[Bean验证](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation-overview)存在于类路径（例如，Hibernate验证）时，`LocalValidatorFactoryBean` 被登记为一全球[验证器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validator)，用于在使用`@Valid`和 `@Validated`上`@Controller`方法的参数。

在Java配置中，您可以自定义全局`Validator`实例，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public Validator getValidator(); {
        // ...
    }

}
```

请注意，您还可以`Validator`在本地注册实现，如以下示例所示：

爪哇

科特林

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }

}
```

|      | 如果需要在`LocalValidatorFactoryBean`某处进行注入，请创建一个bean并标记为Bean，`@Primary`以避免与MVC配置中声明的bean发生冲突。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.11.5。内容类型解析器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-content-negotiation)

您可以配置Spring WebFlux如何根据请求确定`@Controller`实例的请求媒体类型 。默认情况下，仅`Accept`检查标题，但您也可以启用基于查询参数的策略。

以下示例显示如何自定义请求的内容类型解析：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureContentTypeResolver(RequestedContentTypeResolverBuilder builder) {
        // ...
    }
}
```

#### 1.11.6。HTTP消息编解码器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)

以下示例显示如何自定义如何读取和写入请求和响应正文：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
        configurer.defaultCodecs().maxInMemorySize(512 * 1024);
    }
}
```

`ServerCodecConfigurer`提供一组默认的读取器和写入器。您可以使用它来添加更多的读取器和写入器，自定义默认的读取器或完全替换默认的读取器和写入器。

对于Jackson JSON和XML，请考虑使用[`Jackson2ObjectMapperBuilder`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html)，它使用 以下属性自定义Jackson的默认属性：

- [`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html#FAIL_ON_UNKNOWN_PROPERTIES) 被禁用。
- [`MapperFeature.DEFAULT_VIEW_INCLUSION`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html#DEFAULT_VIEW_INCLUSION) 被禁用。

如果在类路径中检测到以下知名模块，它将自动注册以下知名模块：

- [`jackson-datatype-joda`](https://github.com/FasterXML/jackson-datatype-joda)：支持Joda-Time类型。
- [`jackson-datatype-jsr310`](https://github.com/FasterXML/jackson-datatype-jsr310)：支持Java 8日期和时间API类型。
- [`jackson-datatype-jdk8`](https://github.com/FasterXML/jackson-datatype-jdk8)：支持其他Java 8类型，例如`Optional`。
- [`jackson-module-kotlin`](https://github.com/FasterXML/jackson-module-kotlin)：支持Kotlin类和数据类。

#### 1.11.7。查看解析器

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-view-resolvers)

下面的示例显示如何配置视图分辨率：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // ...
    }
}
```

在`ViewResolverRegistry`对与Spring框架的集成视图技术的捷径。以下示例使用FreeMarker（这也需要配置基础FreeMarker视图技术）：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {


    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.freeMarker();
    }

    // Configure Freemarker...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("classpath:/templates");
        return configurer;
    }
}
```

您还可以插入任何`ViewResolver`实现，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {


    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        ViewResolver resolver = ... ;
        registry.viewResolver(resolver);
    }
}
```

为了支持[内容协商](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-multiple-representations)，并通过视图分辨率（除HTML）渲染等格式，可以根据配置一个或多个默认视图`HttpMessageWriterView`实现，它接受任何可用的 [编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)的`spring-web`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {


    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.freeMarker();

        Jackson2JsonEncoder encoder = new Jackson2JsonEncoder();
        registry.defaultViews(new HttpMessageWriterView(encoder));
    }

    // ...
}
```

有关与Spring WebFlux集成的视图技术的更多信息，请参见[View Technologies](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view)。

#### 1.11.8。静态资源

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-static-resources)

此选项提供了一种方便的方法来从[`Resource`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/io/Resource.html)基于位置的列表中提供静态资源 。

在下一个示例中，给定一个以开头的请求，`/resources`相对路径用于查找和提供相对于`/static`类路径的静态资源。资源的有效期为一年，以确保最大程度地利用浏览器缓存并减少浏览器发出的HTTP请求。`Last-Modified`标头也将被评估，如果存在，`304`则返回状态码。以下列表显示了示例：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCacheControl(CacheControl.maxAge(365, TimeUnit.DAYS));
    }

}
```

资源处理程序还支持一系列 [`ResourceResolver`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/resource/ResourceResolver.html)实现和 [`ResourceTransformer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/resource/ResourceTransformer.html)实现，可用于创建使用优化资源的工具链。

您可以使用`VersionResourceResolver`基于内容，固定应用程序版本或其他信息计算出的MD5哈希值的for版本资源URL。一 `ContentVersionStrategy`（MD5哈希）是一个很好的选择有一些明显的例外（比如用模块加载程序使用的JavaScript资源）。

以下示例显示了如何`VersionResourceResolver`在Java配置中使用：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }

}
```

您可以`ResourceUrlProvider`用来重写URL并应用完整的解析器和转换器链（例如，插入版本）。WebFlux配置提供了一个，`ResourceUrlProvider` 以便可以将其注入其他应用程序。

与Spring MVC不同，目前，在WebFlux中，由于没有视图技术可以利用解析器和转换器的无阻塞链，因此无法透明地重写静态资源URL。当仅提供本地资源时，解决方法是`ResourceUrlProvider`直接使用 （例如，通过自定义元素）并进行阻止。

请注意，在同时使用和`EncodedResourceResolver`（例如，Gzip，Brotli编码）和时 `VersionedResourceResolver`，必须按该顺序注册它们，以确保始终基于未编码文件可靠地计算基于内容的版本。

`WebJarsResourceResolver`当`org.webjars:webjars-locator-core`类路径中存在库时，也会通过 自动注册 [WebJars](https://www.webjars.org/documentation)。解析程序可以重写URL以包括jar的版本，还可以与没有版本的传入URL匹配，例如from`/jquery/jquery.min.js`到 `/jquery/1.2.0/jquery.min.js`。

#### 1.11.9。路径匹配

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-path-matching)

您可以自定义与路径匹配有关的选项。有关各个选项的详细信息，请参见 [`PathMatchConfigurer`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/web/reactive/config/PathMatchConfigurer.html)javadoc。以下示例显示如何使用`PathMatchConfigurer`：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setUseCaseSensitiveMatch(true)
            .setUseTrailingSlashMatch(false)
            .addPathPrefix("/api",
                    HandlerTypePredicate.forAnnotation(RestController.class));
    }
}
```

|      | Spring WebFlux依赖于请求路径的解析表示形式， `RequestPath`用于访问解码的路径段值，并删除了分号内容（即路径或矩阵变量）。这意味着，与Spring MVC不同，您无需指示是否解码请求路径，也无需指示是否出于路径匹配目的而删除分号内容。Spring WebFlux还不支持后缀模式匹配，这与Spring MVC不同，在Spring MVC中，我们也[建议](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-suffix-pattern-match)不要依赖它。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.11.10。WebSocket服务

WebFlux Java配置声明了一个`WebSocketHandlerAdapter`bean，该bean支持WebSocket处理程序的调用。这意味着要处理WebSocket握手请求，剩下要做的就是通过将映射`WebSocketHandler`到URL `SimpleUrlHandlerMapping`。

在某些情况下，可能有必要`WebSocketHandlerAdapter`使用提供的`WebSocketService`服务来创建bean，该服务允许配置WebSocket服务器属性。例如：

爪哇

科特林

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public WebSocketService getWebSocketService() {
        TomcatRequestUpgradeStrategy strategy = new TomcatRequestUpgradeStrategy();
        strategy.setMaxSessionIdleTimeout(0L);
        return new HandshakeWebSocketService(strategy);
    }
}
```

#### 1.11.11。高级配置模式

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-advanced-java)

`@EnableWebFlux`导入以下`DelegatingWebFluxConfiguration`内容：

- 为WebFlux应用程序提供默认的Spring配置
- 检测并委托给`WebFluxConfigurer`实现以自定义该配置。

对于高级模式，您可以`@EnableWebFlux`直接从中删除和扩展 `DelegatingWebFluxConfiguration`而不是实现`WebFluxConfigurer`，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class WebConfig extends DelegatingWebFluxConfiguration {

    // ...
}
```

您可以将现有方法保留在中`WebConfig`，但是现在您还可以覆盖基类中的bean声明，并且`WebMvcConfigurer`在类路径上仍然具有任何其他实现。

### 1.12。HTTP / 2

[Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-http2)

Reactor Netty，Tomcat，Jetty和Undertow支持HTTP / 2。但是，有一些与服务器配置有关的注意事项。有关更多详细信息，请参见 [HTTP / 2 Wiki页面](https://github.com/spring-projects/spring-framework/wiki/HTTP-2-support)。

## 2. WebClient

Spring WebFlux包括一个用于执行HTTP请求的客户端。`WebClient`有一个基于Reactor的功能性，流畅的API，请参阅[Reactive Libraries](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-reactive-libraries)，它可以以声明方式构成异步逻辑，而无需处理线程或并发。它是完全非阻塞的，它支持流传输，并且依赖于相同的[编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)，该[编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)还用于在服务器端对请求和响应内容进行编码和解码。

`WebClient`需要一个HTTP客户端库来执行请求。内置支持以下内容：

- [reactive-stack净值](https://github.com/reactor/reactor-netty)
- [码头ReactiveHttpClient](https://github.com/jetty-project/jetty-reactive-httpclient)
- [Apache HttpComponents](https://hc.apache.org/index.html)
- 其他可以通过插入`ClientHttpConnector`。

### 2.1。组态

创建a的最简单方法`WebClient`是通过静态工厂方法之一：

- `WebClient.create()`
- `WebClient.create(String baseUrl)`

您还可以使用`WebClient.builder()`其他选项：

- `uriBuilderFactory`：自定义`UriBuilderFactory`用作基本URL。
- `defaultUriVariables`：扩展URI模板时使用的默认值。
- `defaultHeader`：每个请求的标题。
- `defaultCookie`：针对每个请求的Cookie。
- `defaultRequest`：`Consumer`自定义每个请求。
- `filter`：针对每个请求的客户端过滤器。
- `exchangeStrategies`：HTTP消息读取器/写入器定制。
- `clientConnector`：HTTP客户端库设置。

例如：

爪哇

科特林

```java
WebClient client = WebClient.builder()
        .codecs(configurer -> ... )
        .build();
```

一旦建立，a`WebClient`是不可变的。但是，您可以克隆它并按如下所示构建修改后的副本：

爪哇

科特林

```java
WebClient client1 = WebClient.builder()
        .filter(filterA).filter(filterB).build();

WebClient client2 = client1.mutate()
        .filter(filterC).filter(filterD).build();

// client1 has filterA, filterB

// client2 has filterA, filterB, filterC, filterD
```

#### 2.1.1。最大内存大小

Codecs have [limits](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs-limits) for buffering data in memory to avoid application memory issues. By the default those are set to 256KB. If that’s not enough you’ll get the following error:

```
org.springframework.core.io.buffer.DataBufferLimitException: Exceeded limit on max bytes to buffer
```

To change the limit for default codecs, use the following:

Java

Kotlin

```java
WebClient webClient = WebClient.builder()
        .codecs(configurer -> configurer.defaultCodecs().maxInMemorySize(2 * 1024 * 1024))
        .build();
```

#### 2.1.2. Reactor Netty

To customize Reactor Netty settings, provide a pre-configured `HttpClient`:

Java

Kotlin

```java
HttpClient httpClient = HttpClient.create().secure(sslSpec -> ...);

WebClient webClient = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

##### Resources

By default, `HttpClient` participates in the global Reactor Netty resources held in `reactor.netty.http.HttpResources`, including event loop threads and a connection pool. This is the recommended mode, since fixed, shared resources are preferred for event loop concurrency. In this mode global resources remain active until the process exits.

如果服务器为该进程计时，则通常无需显式关闭。但是，如果服务器可以启动或停止进程内（例如，作为WAR部署的Spring MVC应用程序），则可以声明`ReactorResourceFactory`具有`globalResources=true`（默认值）类型的Spring托管Bean， 以确保Reactor Netty全局资源是当Spring关闭时`ApplicationContext`关闭，如以下示例所示：

爪哇

科特林

```java
@Bean
public ReactorResourceFactory reactorResourceFactory() {
    return new ReactorResourceFactory();
}
```

您也可以选择不参与全局Reactor Netty资源。但是，在这种模式下，确保所有Reactor Netty客户端和服务器实例使用共享资源是您的重担，如以下示例所示：

爪哇

科特林

```java
@Bean
public ReactorResourceFactory resourceFactory() {
    ReactorResourceFactory factory = new ReactorResourceFactory();
    factory.setUseGlobalResources(false); 
    return factory;
}

@Bean
public WebClient webClient() {

    Function<HttpClient, HttpClient> mapper = client -> {
        // Further customizations...
    };

    ClientHttpConnector connector =
            new ReactorClientHttpConnector(resourceFactory(), mapper); 

    return WebClient.builder().clientConnector(connector).build(); 
}
```

|      | 创建独立于全局资源的资源。                                 |
| ---- | ---------------------------------------------------------- |
|      | 将`ReactorClientHttpConnector`构造函数与资源工厂一起使用。 |
|      | 将连接器插入`WebClient.Builder`。                          |

##### 超时时间

要配置连接超时：

爪哇

科特林

```java
import io.netty.channel.ChannelOption;

HttpClient httpClient = HttpClient.create()
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000);

WebClient webClient = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

要配置读取或写入超时：

爪哇

科特林

```java
import io.netty.handler.timeout.ReadTimeoutHandler;
import io.netty.handler.timeout.WriteTimeoutHandler;

HttpClient httpClient = HttpClient.create()
        .doOnConnected(conn -> conn
                .addHandlerLast(new ReadTimeoutHandler(10))
                .addHandlerLast(new WriteTimeoutHandler(10)));

// Create WebClient...
```

要为所有请求配置响应超时：

爪哇

科特林

```java
HttpClient httpClient = HttpClient.create()
        .responseTimeout(Duration.ofSeconds(2));

// Create WebClient...
```

要为特定请求配置响应超时：

爪哇

科特林

```java
WebClient.create().get()
        .uri("https://example.org/path")
        .httpRequest(httpRequest -> {
            HttpClientRequest reactorRequest = httpRequest.getNativeRequest();
            reactorRequest.responseTimeout(Duration.ofSeconds(2));
        })
        .retrieve()
        .bodyToMono(String.class);
```

#### 2.1.3。码头

以下示例显示如何自定义Jetty`HttpClient`设置：

爪哇

科特林

```java
HttpClient httpClient = new HttpClient();
httpClient.setCookieStore(...);

WebClient webClient = WebClient.builder()
        .clientConnector(new JettyClientHttpConnector(httpClient))
        .build();
```

默认情况下，`HttpClient`创建自己的资源（`Executor`，`ByteBufferPool`，`Scheduler`），其保持有效，直到进程退出或者`stop()`被调用。

您可以在Jetty客户端（和服务器）的多个实例之间共享资源，并`ApplicationContext`通过声明类型`JettyResourceFactory`为Spring托管的bean来确保在关闭Spring时关闭资源，如以下示例所示：

爪哇

科特林

```java
@Bean
public JettyResourceFactory resourceFactory() {
    return new JettyResourceFactory();
}

@Bean
public WebClient webClient() {

    HttpClient httpClient = new HttpClient();
    // Further customizations...

    ClientHttpConnector connector =
            new JettyClientHttpConnector(httpClient, resourceFactory()); 

    return WebClient.builder().clientConnector(connector).build(); 
}
```

|      | 将`JettyClientHttpConnector`构造函数与资源工厂一起使用。 |
| ---- | -------------------------------------------------------- |
|      | 将连接器插入`WebClient.Builder`。                        |

#### 2.1.4。HttpComponents

以下示例显示了如何自定义Apache HttpComponents`HttpClient`设置：

爪哇

科特林

```java
HttpAsyncClientBuilder clientBuilder = HttpAsyncClients.custom();
clientBuilder.setDefaultRequestConfig(...);
CloseableHttpAsyncClient client = clientBuilder.build();
ClientHttpConnector connector = new HttpComponentsClientHttpConnector(client);

WebClient webClient = WebClient.builder().clientConnector(connector).build();
```

### 2.2。 `retrieve()`

该`retrieve()`方法可用于声明如何提取响应。例如：

爪哇

科特林

```java
WebClient client = WebClient.create("https://example.org");

Mono<ResponseEntity<Person>> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .toEntity(Person.class);
```

或只得到身体：

爪哇

科特林

```java
WebClient client = WebClient.create("https://example.org");

Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .bodyToMono(Person.class);
```

要获取解码对象流：

爪哇

科特林

```java
Flux<Quote> result = client.get()
        .uri("/quotes").accept(MediaType.TEXT_EVENT_STREAM)
        .retrieve()
        .bodyToFlux(Quote.class);
```

默认情况下，4xx或5xx响应会产生`WebClientResponseException`，包括特定HTTP状态代码的子类。要自定义错误响应的`onStatus`处理，请按以下方式使用处理程序：

爪哇

科特林

```java
Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError, response -> ...)
        .onStatus(HttpStatus::is5xxServerError, response -> ...)
        .bodyToMono(Person.class);
```

### 2.3。交换

的`exchangeToMono()`和`exchangeToFlux()`的方法（或`awaitExchange { }`和`exchangeToFlow { }`在科特林）对于需要更多的控制更高级的情况下是有用的，例如，以不同的方式取决于响应状态的响应进行解码：

爪哇

科特林

```java
Mono<Object> entityMono = client.get()
       .uri("/persons/1")
       .accept(MediaType.APPLICATION_JSON)
       .exchangeToMono(response -> {
           if (response.statusCode().equals(HttpStatus.OK)) {
               return response.bodyToMono(Person.class);
           }
           else if (response.statusCode().is4xxClientError()) {
               return response.bodyToMono(ErrorContainer.class);
           }
           else {
               return Mono.error(response.createException());
           }
       });
```

使用上述方法时，在返回`Mono`或`Flux`完成后，将检查响应主体，如果未消耗响应主体，则将其释放以防止内存和连接泄漏。因此，无法在下游进一步解码响应。如果需要，由提供的函数声明如何解码响应。

### 2.4。请求正文

可以使用`ReactiveAdapterRegistry`，如`Mono`或Kotlin Coroutines处理的任何异步类型对请求主体进行编码，如`Deferred`以下示例所示：

爪哇

科特林

```java
Mono<Person> personMono = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .body(personMono, Person.class)
        .retrieve()
        .bodyToMono(Void.class);
```

您还可以对对象流进行编码，如以下示例所示：

爪哇

科特林

```java
Flux<Person> personFlux = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_STREAM_JSON)
        .body(personFlux, Person.class)
        .retrieve()
        .bodyToMono(Void.class);
```

或者，如果您具有实际值，则可以使用`bodyValue`快捷方式，如以下示例所示：

爪哇

科特林

```java
Person person = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue(person)
        .retrieve()
        .bodyToMono(Void.class);
```

#### 2.4.1。表格数据

要发送表单数据，您可以提供一个`MultiValueMap<String, String>`作为正文。请注意，内容是`application/x-www-form-urlencoded`由自动设置为的 `FormHttpMessageWriter`。以下示例显示如何使用`MultiValueMap<String, String>`：

爪哇

科特林

```java
MultiValueMap<String, String> formData = ... ;

Mono<Void> result = client.post()
        .uri("/path", id)
        .bodyValue(formData)
        .retrieve()
        .bodyToMono(Void.class);
```

您还可以使用来在线提供表单数据`BodyInserters`，如以下示例所示：

爪哇

科特林

```java
import static org.springframework.web.reactive.function.BodyInserters.*;

Mono<Void> result = client.post()
        .uri("/path", id)
        .body(fromFormData("k1", "v1").with("k2", "v2"))
        .retrieve()
        .bodyToMono(Void.class);
```

#### 2.4.2。多部分数据

要发送多部分数据，您需要提供一个，`MultiValueMap<String, ?>`其值要么`Object`是代表零件内容的`HttpEntity`实例，要么是代表零件内容和标题的实例。`MultipartBodyBuilder`提供了方便的API来准备多部分请求。以下示例显示了如何创建`MultiValueMap<String, ?>`：

爪哇

科特林

```java
MultipartBodyBuilder builder = new MultipartBodyBuilder();
builder.part("fieldPart", "fieldValue");
builder.part("filePart1", new FileSystemResource("...logo.png"));
builder.part("jsonPart", new Person("Jason"));
builder.part("myPart", part); // Part from a server request

MultiValueMap<String, HttpEntity<?>> parts = builder.build();
```

在大多数情况下，您不必`Content-Type`为每个零件指定。内容类型是根据`HttpMessageWriter`选择的要自动序列化的内容确定的，如果是`Resource`，则根据文件扩展名来确定。如有必要，您可以`MediaType`通过重载的构建器`part`方法之一显式提供用于每个零件的。

一旦`MultiValueMap`准备，最简单的方法将它传递到`WebClient`是通过`body`方法，如下面的示例所示：

爪哇

科特林

```java
MultipartBodyBuilder builder = ...;

Mono<Void> result = client.post()
        .uri("/path", id)
        .body(builder.build())
        .retrieve()
        .bodyToMono(Void.class);
```

如果`MultiValueMap`包含至少一个非`String`值（也可以表示常规表单数据）（即`application/x-www-form-urlencoded`），则无需将设置`Content-Type`为`multipart/form-data`。使用时总是如此 `MultipartBodyBuilder`，以确保`HttpEntity`包装。

作为的替代`MultipartBodyBuilder`，您还可以通过内建的方式以内联方式提供多部分内容`BodyInserters`，如以下示例所示：

爪哇

科特林

```java
import static org.springframework.web.reactive.function.BodyInserters.*;

Mono<Void> result = client.post()
        .uri("/path", id)
        .body(fromMultipartData("fieldPart", "value").with("filePart", resource))
        .retrieve()
        .bodyToMono(Void.class);
```

### 2.5。筛选器

您可以`ExchangeFilterFunction`通过来注册客户端过滤器（）`WebClient.Builder` ，以拦截和修改请求，如以下示例所示：

爪哇

科特林

```java
WebClient client = WebClient.builder()
        .filter((request, next) -> {

            ClientRequest filtered = ClientRequest.from(request)
                    .header("foo", "bar")
                    .build();

            return next.exchange(filtered);
        })
        .build();
```

这可以用于跨领域的关注，例如身份验证。以下示例使用过滤器通过静态工厂方法进行基本身份验证：

爪哇

科特林

```java
import static org.springframework.web.reactive.function.client.ExchangeFilterFunctions.basicAuthentication;

WebClient client = WebClient.builder()
        .filter(basicAuthentication("user", "password"))
        .build();
```

您可以`WebClient`通过使用另一个实例作为起点来创建新实例。这样可以在不影响原稿的情况下插入或卸下过滤器`WebClient`。以下是在索引0处插入基本身份验证过滤器的示例：

爪哇

科特林

```java
import static org.springframework.web.reactive.function.client.ExchangeFilterFunctions.basicAuthentication;

WebClient client = webClient.mutate()
        .filters(filterList -> {
            filterList.add(0, basicAuthentication("user", "password"));
        })
        .build();
```

### 2.6。属性

您可以向请求添加属性。如果要通过筛选器链传递信息并影响给定请求的筛选器行为，这将很方便。例如：

爪哇

科特林

```java
WebClient client = WebClient.builder()
        .filter((request, next) -> {
            Optional<Object> usr = request.attribute("myAttribute");
            // ...
        })
        .build();

client.get().uri("https://example.org/")
        .attribute("myAttribute", "...")
        .retrieve()
        .bodyToMono(Void.class);

    }
```

### 2.7。语境

[属性](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client-attributes)提供了一种将信息传递到筛选器链的便捷方法，但是它们仅影响当前请求。如果您想传递传播到嵌套的其他请求的信息（例如通过`flatMap`或在之后通过）执行`concatMap`，则需要使用Reactor `Context`。

`WebClient`公开`Context`用于为给定请求填充Reactor的方法。此信息可用于过滤当前请求，并且还传播到后续请求或参与下游处理链的其他Reactive客户端。例如：

爪哇

```java
WebClient client = WebClient.builder()
        .filter((request, next) ->
                Mono.deferContextual(contextView -> {
                    String value = contextView.get("foo");
                    // ...
                }))
        .build();

client.get().uri("https://example.org/")
        .context(context -> context.put("foo", ...))
        .retrieve()
        .bodyToMono(String.class)
        .flatMap(body -> {
                // perform nested request (context propagates automatically)...
        });
```

请注意，您还可以`defaultRequest` 在和级别上指定如何通过方法填充上下文，该方法`WebClient.Builder`适用于所有请求。例如，这可以用于将信息从`ThreadLocal`存储传递到Spring MVC应用程序中的Reactor处理链上。

### 2.8。同步使用

`WebClient` 可以通过在末尾进行阻塞来以同步方式使用：

爪哇

科特林

```java
Person person = client.get().uri("/person/{id}", i).retrieve()
    .bodyToMono(Person.class)
    .block();

List<Person> persons = client.get().uri("/persons").retrieve()
    .bodyToFlux(Person.class)
    .collectList()
    .block();
```

但是，如果需要进行多个调用，则避免单独阻塞每个响应，而等待合并的结果会更有效：

爪哇

科特林

```java
Mono<Person> personMono = client.get().uri("/person/{id}", personId)
        .retrieve().bodyToMono(Person.class);

Mono<List<Hobby>> hobbiesMono = client.get().uri("/person/{id}/hobbies", personId)
        .retrieve().bodyToFlux(Hobby.class).collectList();

Map<String, Object> data = Mono.zip(personMono, hobbiesMono, (person, hobbies) -> {
            Map<String, String> map = new LinkedHashMap<>();
            map.put("person", person);
            map.put("hobbies", hobbies);
            return map;
        })
        .block();
```

以上仅是一个示例。还有许多其他模式和运算符可用于构建响应式管道，该管道可进行许多远程调用（可能是嵌套的，相互依赖的），而不会阻塞到最后。

|      | 使用`Flux`或`Mono`，您永远不必阻塞Spring MVC或Spring WebFlux控制器。只需从controller方法返回返回的反应类型。相同的原则适用于Kotlin Coroutines和Spring WebFlux，只需使用暂停功能或`Flow`在控制器方法中返回即可。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.9。测验

要测试使用的代码，`WebClient`可以使用模拟Web服务器，例如 [OkHttp MockWebServer](https://github.com/square/okhttp#mockwebserver)。要查看其用法示例，请查看 [`WebClientIntegrationTests`](https://github.com/spring-projects/spring-framework/blob/master/spring-webflux/src/test/java/org/springframework/web/reactive/function/client/WebClientIntegrationTests.java) Spring Framework测试套件或[`static-server`](https://github.com/square/okhttp/tree/master/samples/static-server) OkHttp存储库中的 示例。

## 3. WebSockets

[与Servlet堆栈中的相同](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket)

参考文档的此部分涵盖对reactive-stackWebSocket消息传递的支持。

### 3.1。WebSocket介绍

WebSocket协议[RFC 6455](https://tools.ietf.org/html/rfc6455)提供了一种标准化的方法，可以通过单个TCP连接在客户端和服务器之间建立全双工的双向通信通道。它是与HTTP不同的TCP协议，但旨在通过端口80和443在HTTP上工作，并允许重复使用现有的防火墙规则。

WebSocket交互始于一个HTTP请求，该请求使用HTTP`Upgrade`标头进行升级，或者在这种情况下切换到WebSocket协议。以下示例显示了这种交互：

```yaml
GET /spring-websocket-portfolio/portfolio HTTP/1.1
Host: localhost:8080
Upgrade: websocket 
Connection: Upgrade 
Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
Sec-WebSocket-Protocol: v10.stomp, v11.stomp
Sec-WebSocket-Version: 13
Origin: http://localhost:8080
```

|      | 该`Upgrade`头。     |
| ---- | ------------------- |
|      | 使用`Upgrade`连接。 |

具有WebSocket支持的服务器代替通常的200状态代码，返回类似于以下内容的输出：

```yaml
HTTP/1.1 101 Switching Protocols 
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
Sec-WebSocket-Protocol: v10.stomp
```

|      | 协议切换 |
| ---- | -------- |
|      |          |

成功握手后，HTTP升级请求的基础TCP套接字将保持打开状态，客户端和服务器均可继续发送和接收消息。

WebSockets的工作原理的完整介绍超出了本文档的范围。请参阅RFC 6455，HTML5的WebSocket章节，或Web上的许多简介和教程中的任何一个。

请注意，如果WebSocket服务器在Web服务器（例如nginx）后面运行，则可能需要对其进行配置，以将WebSocket升级请求传递到WebSocket服务器。同样，如果应用程序在云环境中运行，请检查与WebSocket支持相关的云提供商的说明。

#### 3.1.1。HTTP与WebSocket

尽管WebSocket被设计为与HTTP兼容并以HTTP请求开头，但重要的是要了解这两个协议导致了截然不同的体系结构和应用程序编程模型。

在HTTP和REST中，应用程序被建模为许多URL。为了与应用程序交互，客户端访问那些URL，即请求-响应样式。服务器根据HTTP URL，方法和标头将请求路由到适当的处理程序。

相比之下，在WebSockets中，初始连接通常只有一个URL。随后，所有应用程序消息都在同一TCP连接上流动。这指向了一个完全不同的异步，事件驱动的消息传递体系结构。

WebSocket也是一种低级传输协议，与HTTP不同，它不对消息的内容规定任何语义。这意味着除非客户端和服务器就消息语义达成一致，否则就无法路由或处理消息。

WebSocket客户端和服务器可以通过`Sec-WebSocket-Protocol`HTTP握手请求上的标头来协商更高级别的消息传递协议（例如STOMP）的使用。在这种情况下，他们需要提出自己的约定。

#### 3.1.2。何时使用WebSockets

WebSockets可以使网页具有动态性和交互性。但是，在许多情况下，结合使用Ajax和HTTP流或长时间轮询可以提供一种简单有效的解决方案。

例如，新闻，邮件和社交订阅源需要动态更新，但是每几分钟进行一次更新可能是完全可以的。另一方面，协作，游戏和金融应用程序需要更接近实时。

仅延迟并不是决定因素。如果消息量相对较少（例如，监视网络故障），则HTTP流或轮询可以提供有效的解决方案。低延迟，高频率和高音量的结合才是使用WebSocket的最佳案例。

还要记住，在Internet上，控件之外的限制性代理可能会阻止WebSocket交互，这可能是因为未将它们配置为传递 `Upgrade`标头，或者是因为它们关闭了长期处于空闲状态的连接。这意味着与面向公众的应用程序相比，将WebSocket用于防火墙内部的应用程序是一个更直接的决定。

### 3.2。WebSocket API

[与Servlet堆栈中的相同](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-server)

Spring框架提供了一个WebSocket API，可用于编写处理WebSocket消息的客户端和服务器端应用程序。

#### 3.2.1。服务器

[与Servlet堆栈中的相同](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-server-handler)

要创建WebSocket服务器，您可以先创建一个`WebSocketHandler`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.WebSocketSession;

public class MyWebSocketHandler implements WebSocketHandler {

    @Override
    public Mono<Void> handle(WebSocketSession session) {
        // ...
    }
}
```

然后，您可以将其映射到URL：

爪哇

科特林

```java
@Configuration
class WebConfig {

    @Bean
    public HandlerMapping handlerMapping() {
        Map<String, WebSocketHandler> map = new HashMap<>();
        map.put("/path", new MyWebSocketHandler());
        int order = -1; // before annotated controllers

        return new SimpleUrlHandlerMapping(map, order);
    }
}
```

如果使用[WebFlux Config](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config)，则无需执行其他操作；否则，如果不使用WebFlux Config，则需要声明a `WebSocketHandlerAdapter`，如下所示：

爪哇

科特林

```java
@Configuration
class WebConfig {

    // ...

    @Bean
    public WebSocketHandlerAdapter handlerAdapter() {
        return new WebSocketHandlerAdapter();
    }
}
```

#### 3.2.2。 `WebSocketHandler`

接受和返回的`handle`方法 指示会话的应用程序处理何时完成。通过两个流处理会话，一个流用于入站消息，一个流用于出站消息。下表描述了两种处理流的方法：`WebSocketHandler``WebSocketSession``Mono<Void>`

| `WebSocketSession` 方法                        | 描述                                                         |
| :--------------------------------------------- | :----------------------------------------------------------- |
| `Flux<WebSocketMessage> receive()`             | 提供对入站消息流的访问，并在关闭连接时完成。                 |
| `Mono<Void> send(Publisher<WebSocketMessage>)` | 获取传出消息的源，编写消息，并`Mono<Void>`在源完成并完成写入时返回完成的消息。 |

一个`WebSocketHandler`必须组成入站和出站流为一个统一的流程，并返回一个`Mono<Void>`反映该流的完成。根据应用程序要求，统一流程在以下情况下完成：

- 入站或出站消息流完成。
- 入站流完成（即，连接已关闭），而出站流是无限的。
- 在选定的点，通过的`close`方法`WebSocketSession`。

将入站和出站消息流组合在一起时，无需检查连接是否已打开，因为响应式流发出结束活动的信号。入站流接收完成或错误信号，而出站流接收取消信号。

处理程序最基本的实现是处理入站流的实现。以下示例显示了这样的实现：

爪哇

科特林

```java
class ExampleHandler implements WebSocketHandler {

    @Override
    public Mono<Void> handle(WebSocketSession session) {
        return session.receive()            
                .doOnNext(message -> {
                    // ...                  
                })
                .concatMap(message -> {
                    // ...                  
                })
                .then();                    
    }
}
```

|      | 访问入站消息流。                    |
| ---- | ----------------------------------- |
|      | 对每条消息进行处理。                |
|      | 执行使用消息内容的嵌套异步操作。    |
|      | `Mono<Void>`接收完成后返回完成的a。 |

|      | 对于嵌套的异步操作，您可能需要调用`message.retain()`使用池化数据缓冲区的基础服务器（例如，Netty）。否则，在您有机会读取数据之前，可能会释放数据缓冲区。有关更多背景信息，请参见 [数据缓冲区和编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下实现将入站和出站流组合在一起：

爪哇

科特林

```java
class ExampleHandler implements WebSocketHandler {

    @Override
    public Mono<Void> handle(WebSocketSession session) {

        Flux<WebSocketMessage> output = session.receive()               
                .doOnNext(message -> {
                    // ...
                })
                .concatMap(message -> {
                    // ...
                })
                .map(value -> session.textMessage("Echo " + value));    

        return session.send(output);                                    
    }
}
```

|      | 处理入站消息流。                             |
| ---- | -------------------------------------------- |
|      | 创建出站消息，产生合并流。                   |
|      | `Mono<Void>`当我们继续接收时，返回未完成的。 |

入站和出站流可以是独立的，并且只能为了完成而加入，如以下示例所示：

爪哇

科特林

```java
class ExampleHandler implements WebSocketHandler {

    @Override
    public Mono<Void> handle(WebSocketSession session) {

        Mono<Void> input = session.receive()                                
                .doOnNext(message -> {
                    // ...
                })
                .concatMap(message -> {
                    // ...
                })
                .then();

        Flux<String> source = ... ;
        Mono<Void> output = session.send(source.map(session::textMessage)); 

        return Mono.zip(input, output).then();                              
    }
}
```

|      | 处理入站消息流。                                 |
| ---- | ------------------------------------------------ |
|      | 发送外发消息。                                   |
|      | 加入流并返回其中一个`Mono<Void>`流结束时完成的。 |

#### 3.2.3。 `DataBuffer`

`DataBuffer`是WebFlux中字节缓冲区的表示形式。参考资料的Spring Core部分在“[数据缓冲区和编解码器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers)”部分中有更多介绍 。要理解的关键点是，在诸如Netty之类的某些服务器上，字节缓冲被池化并计数引用，并且在使用时必须将其释放以避免内存泄漏。

在Netty上运行时，`DataBufferUtils.retain(dataBuffer)`如果应用程序希望保留输入数据缓冲区，则必须使用它们以确保不释放它们，并在使用`DataBufferUtils.release(dataBuffer)`完缓冲区时随后使用。

#### 3.2.4。握手

[与Servlet堆栈中的相同](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-server-handshake)

`WebSocketHandlerAdapter`代表参加`WebSocketService`。默认情况下，它是的实例`HandshakeWebSocketService`，该实例对WebSocket请求执行基本检查，然后将其`RequestUpgradeStrategy`用于所使用的服务器。当前，内置了对Reactor Netty，Tomcat，Jetty和Undertow的支持。

`HandshakeWebSocketService`公开一个`sessionAttributePredicate`允许设置的属性，`Predicate<String>`以从提取属性`WebSession`并将其插入的属性`WebSocketSession`。

#### 3.2.5。服务器配置

[与Servlet堆栈中的相同](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-server-runtime-configuration)

的`RequestUpgradeStrategy`每个服务器公开配置的具体到下面的WebSocket服务器引擎。使用WebFlux Java配置时，您可以自定义这些属性，如[WebFlux配置](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-config-websocket-service)的相应部分所示。 ，否则，如果不使用WebFlux配置，请使用以下属性：

爪哇

科特林

```java
@Configuration
class WebConfig {

    @Bean
    public WebSocketHandlerAdapter handlerAdapter() {
        return new WebSocketHandlerAdapter(webSocketService());
    }

    @Bean
    public WebSocketService webSocketService() {
        TomcatRequestUpgradeStrategy strategy = new TomcatRequestUpgradeStrategy();
        strategy.setMaxSessionIdleTimeout(0L);
        return new HandshakeWebSocketService(strategy);
    }
}
```

检查服务器的升级策略，以查看可用的选项。当前，只有Tomcat和Jetty公开了此类选项。

#### 3.2.6。CORS

[与Servlet堆栈中的相同](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-server-allowed-origins)

配置CORS并限制对WebSocket端点的访问的最简单方法是使用您的`WebSocketHandler`实现`CorsConfigurationSource`并返回 `CorsConfiguration`带有允许的来源，标头和其他详细信息的。如果您不能这样做，也可以在`corsConfigurations`上设置该属性，`SimpleUrlHandler`以通过URL模式指定CORS设置。如果同时指定了两者，则使用上的`combine`方法将它们合并 `CorsConfiguration`。

#### 3.2.7。客户

Spring WebFlux提供了`WebSocketClient`Reactor Netty，Tomcat，Jetty，Undertow和标准Java（即JSR-356）实现的抽象。

|      | Tomcat客户端实际上是标准Java客户端的扩展，在`WebSocketSession`处理方面具有一些额外的功能，以利用特定于Tomcat的API暂停接收消息以产生反压。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要启动WebSocket会话，可以创建客户端的实例并使用其`execute` 方法：

爪哇

科特林

```java
WebSocketClient client = new ReactorNettyWebSocketClient();

URI url = new URI("ws://localhost:8080/path");
client.execute(url, session ->
        session.receive()
                .doOnNext(System.out::println)
                .then());
```

一些客户端（例如Jetty）`Lifecycle`需要实施并需要停止和启动，然后才能使用它们。所有客户端都具有与基础WebSocket客户端的配置有关的构造器选项。

## 4.测试

[在Spring MVC中相同](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#testing)

该`spring-test`模块提供的模拟实现中`ServerHttpRequest`， `ServerHttpResponse`和`ServerWebExchange`。有关模拟对象的讨论，请参见[Spring Web Reactive](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#mock-objects-web-reactive)。

[`WebTestClient`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#webtestclient)以这些模拟请求和响应对象为基础，可提供对不使用HTTP服务器的WebFlux应用程序进行测试的支持。您也可以将其`WebTestClient`用于端到端集成测试。

## 5. RSocket

本节描述了Spring Framework对RSocket协议的支持。

### 5.1。总览

RSocket是一种应用协议，用于使用以下交互模型之一通过TCP，WebSocket和其他字节流传输进行多路复用，双工通信：

- `Request-Response` —发送一封邮件并收到一封邮件。
- `Request-Stream` —发送一条消息并接收消息流。
- `Channel` —双向发送消息流。
- `Fire-and-Forget` —发送单向消息。

一旦建立了初始连接，由于双方变得对称，并且每一方都可以发起上述交互之一，因此“客户端”与“服务器”的区别将消失。这就是为什么在协议中将参与方称为“请求者”和“响应者”，而将上述交互称为“请求流”或简称为“请求”的原因。

这些是RSocket协议的关键功能和优势：

- [响应流](https://www.reactive-streams.org/)跨网络边界的语义-用于诸如`Request-Stream`和的流请求`Channel`，背压信号在请求者和响应者之间传播，从而允许请求者放慢源处的响应者的速度，从而减少了对网络层拥塞控制的依赖，并减少了在网络级别或任何级别。
- 请求限制-在`LEASE`可以从两端发送的帧之后，此功能称为“租赁”，以限制给定时间内另一端允许的请求总数。租约会定期更新。
- 会话恢复-这是为断开连接而设计的，需要维护一些状态。状态管理对于应用程序是透明的，并且可以与背压结合使用，从而可以在可能的情况下停止生产者并减少所需的状态量。
- 大邮件的碎片化和重组。
- Keepalive（心跳）。

RSocket具有多种语言的[实现](https://github.com/rsocket)。该 [Java库](https://github.com/rsocket/rsocket-java)是建立在[工程reactive-stack](https://projectreactor.io/)及[reactive-stack的Netty](https://github.com/reactor/reactor-netty)的运输。这意味着来自应用程序中的响应流 Publisher的信号通过RSocket在网络上透明地传播。

#### 5.1.1。协议书

RSocket的好处之一是它具有明确定义的行为，并且易于阅读的[规范](https://rsocket.io/docs/Protocol)以及某些协议 [扩展](https://github.com/rsocket/rsocket/tree/master/Extensions)。因此，独立于语言实现和更高级别的框架API，阅读规范是一个好主意。本节提供简要概述，以建立一些上下文。

**连接中**

最初，客户端通过一些低级流传输（例如TCP或WebSocket）连接到服务器，并将`SETUP`帧发送到服务器以设置连接参数。

服务器可以拒绝该`SETUP`帧，但是通常在发送（对于客户端）和接收（对于服务器）之后，双方都可以开始发出请求，除非`SETUP` 指示使用租赁语义来限制请求的数量，在这种情况下双方必须等待`LEASE`另一端发出的请求才能允许发出请求。

**发出请求**

一旦建立了连接，双方可以通过帧中的一个发起的请求`REQUEST_RESPONSE`，`REQUEST_STREAM`，`REQUEST_CHANNEL`，或`REQUEST_FNF`。这些帧中的每一个都将一条消息从请求者传送到响应者。

然后`PAYLOAD`，响应者可以返回带有响应消息的帧，并且在`REQUEST_CHANNEL`请求者的情况下，还可以发送`PAYLOAD`带有更多请求消息的帧。

当请求涉及诸如`Request-Stream`和的消息流时`Channel`，响应者必须遵守来自请求者的需求信号。需求表示为许多消息。初始需求在`REQUEST_STREAM`和 `REQUEST_CHANNEL`框架中指定。后续需求通过`REQUEST_N`帧发出信号。

每一端还可以通过该`METADATA_PUSH`帧发送与任何单个请求无关，而是与整个连接有关的元数据通知。

**讯息格式**

RSocket消息包含数据和元数据。元数据可用于发送路由，安全令牌等。数据和元数据的格式可以不同。每种类型的Mime类型都在`SETUP`框架中声明，并应用于给定连接上的所有请求。

虽然所有的消息可以具有元数据，如典型的元数据的路由是每个请求，因此仅包含在一个请求中的第一消息中，即，与帧中的一个 `REQUEST_RESPONSE`，`REQUEST_STREAM`，`REQUEST_CHANNEL`，或`REQUEST_FNF`。

协议扩展定义了用于应用程序的通用元数据格式：

- [复合元数据](https://github.com/rsocket/rsocket/blob/master/Extensions/CompositeMetadata.md)-多个独立格式化的元数据条目。
- [路由](https://github.com/rsocket/rsocket/blob/master/Extensions/Routing.md) -请求的路由。

#### 5.1.2。Java实现

RSocket的[Java实现](https://github.com/rsocket/rsocket-java)基于[Project Reactor](https://projectreactor.io/)构建 。TCP和WebSocket的传输基于[Reactor Netty](https://github.com/reactor/reactor-netty)构建。作为响应流库，Reactor简化了实现协议的工作。对于应用来说是天作之合来使用`Flux`和`Mono`用声明运营商和透明背压支持。

RSocket Java中的API故意是最小且基本的。它专注于协议功能，而将应用程序编程模型（例如RPC代码生成与其他）作为一个更高级别的独立关注点。

主合同 [io.rsocket.RSocket](https://github.com/rsocket/rsocket-java/blob/master/rsocket-core/src/main/java/io/rsocket/RSocket.java) 对四种请求交互类型进行建模，分别`Mono`表示对单个消息的承诺，消息`Flux`流以及`io.rsocket.Payload`可以访问数据和元数据作为字节缓冲区的实际消息。该`RSocket`合同是对称使用。对于请求，应用程序被赋予`RSocket`执行请求的权利。为了响应，应用程序实现`RSocket`了处理请求。

这并不意味着要进行全面介绍。在大多数情况下，Spring应用程序将不必直接使用其API。但是，独立于Spring查看或试验RSocket可能很重要。RSocket Java存储库包含许多 [示例应用程序](https://github.com/rsocket/rsocket-java/tree/master/rsocket-examples)，以演示其API和协议功能。

#### 5.1.3。弹簧支撑

该`spring-messaging`模块包含以下内容：

- [RSocketRequester](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-requester) —流利的API，通过`io.rsocket.RSocket` 带有数据和元数据的编码/解码来发出请求。
- [带注释的响应](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-annot-responders)`@MessageMapping`程序 — 用于响应的带注释的处理程序方法。

该`spring-web`模块包含`Encoder`和`Decoder`实现，比如杰克逊CBOR / JSON和的Protobuf该方法RSocket应用程序可能会需要。它还包含 `PathPatternParser`可以插入以进行有效路由匹配的。

Spring Boot 2.2支持通过TCP或WebSocket站立RSocket服务器，包括在WebFlux服务器中通过WebSocket公开RSocket的选项。还为`RSocketRequester.Builder`和提供客户端支持和自动配置`RSocketStrategies`。有关 更多详细信息，请参见Spring Boot参考中的 [RSocket部分](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-rsocket)。

Spring Security 5.2提供了RSocket支持。

Spring Integration 5.2提供了入站和出站网关以与RSocket客户端和服务器进行交互。有关更多详细信息，请参见《 Spring Integration参考手册》。

Spring Cloud Gateway支持RSocket连接。

### 5.2。RSocketRequester

`RSocketRequester`提供流利的API来执行RSocket请求，接受和返回数据和元数据的对象，而不是低级数据缓冲区。它可以对称地用于从客户端发出请求和从服务器发出请求。

#### 5.2.1。客户要求者

`RSocketRequester`在客户端获得一个是连接到服务器，这涉及到发送`SETUP`带有连接设置的RSocket框架。`RSocketRequester`提供了一个有助于准备框架的`io.rsocket.core.RSocketConnector`包括连接设置的构建器`SETUP`。

这是使用默认设置进行连接的最基本方法：

爪哇

科特林

```java
RSocketRequester requester = RSocketRequester.builder().tcp("localhost", 7000);

URI url = URI.create("https://example.org:8080/rsocket");
RSocketRequester requester = RSocketRequester.builder().webSocket(url);
```

上面没有立即连接。发出请求时，将透明地建立并使用共享连接。

##### 连接设置

`RSocketRequester.Builder`提供以下内容以定制初始`SETUP`框架：

- `dataMimeType(MimeType)` —设置连接数据的MIME类型。
- `metadataMimeType(MimeType)` —设置连接上元数据的mime类型。
- `setupData(Object)` —要包含在中的数据`SETUP`。
- `setupRoute(String, Object…)` —将元数据路由到中`SETUP`。
- `setupMetadata(Object, MimeType)` —要包含在中的其他元数据`SETUP`。

对于数据，默认的mime类型是从第一个configure派生的`Decoder`。对于元数据，默认的mime类型是 [复合元数据](https://github.com/rsocket/rsocket/blob/master/Extensions/CompositeMetadata.md)，它允许每个请求有多个元数据值和mime类型对。通常，两者都不需要更改。

`SETUP`框架中的数据和元数据是可选的。在服务器端，@ [ConnectMapping](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-annot-connectmapping)方法可用于处理连接的开始和`SETUP`框架的内容。元数据可用于连接级别的安全性。

##### 策略

`RSocketRequester.Builder` accepts `RSocketStrategies` to configure the requester. You’ll need to use this to provide encoders and decoders for (de)-serialization of data and metadata values. By default only the basic codecs from `spring-core` for `String`, `byte[]`, and `ByteBuffer` are registered. Adding `spring-web` provides access to more that can be registered as follows:

Java

Kotlin

```java
RSocketStrategies strategies = RSocketStrategies.builder()
    .encoders(encoders -> encoders.add(new Jackson2CborEncoder()))
    .decoders(decoders -> decoders.add(new Jackson2CborDecoder()))
    .build();

RSocketRequester requester = RSocketRequester.builder()
    .rsocketStrategies(strategies)
    .tcp("localhost", 7000);
```

`RSocketStrategies` is designed for re-use. In some scenarios, e.g. client and server in the same application, it may be preferable to declare it in Spring configuration.

##### Client Responders

`RSocketRequester.Builder` can be used to configure responders to requests from the server.

You can use annotated handlers for client-side responding based on the same infrastructure that’s used on a server, but registered programmatically as follows:

Java

Kotlin

```java
RSocketStrategies strategies = RSocketStrategies.builder()
    .routeMatcher(new PathPatternRouteMatcher())  
    .build();

SocketAcceptor responder =
    RSocketMessageHandler.responder(strategies, new ClientHandler()); 

RSocketRequester requester = RSocketRequester.builder()
    .rsocketConnector(connector -> connector.acceptor(responder)) 
    .tcp("localhost", 7000);
```

|      | Use `PathPatternRouteMatcher`, if `spring-web` is present, for efficient route matching. |
| ---- | ------------------------------------------------------------ |
|      | 使用`@MessageMaping`和/或`@ConnectMapping`方法从类中创建响应者。 |
|      | 注册响应者。                                                 |

请注意，以上只是设计用于客户端响应程序的程序化注册的快捷方式。对于替代方案，其中客户端响应者处于Spring配置中，您仍然可以声明`RSocketMessageHandler`为Spring Bean，然后按如下所示进行应用：

爪哇

科特林

```java
ApplicationContext context = ... ;
RSocketMessageHandler handler = context.getBean(RSocketMessageHandler.class);

RSocketRequester requester = RSocketRequester.builder()
    .rsocketConnector(connector -> connector.acceptor(handler.responder()))
    .tcp("localhost", 7000);
```

对于上述可能还需要使用`setHandlerPredicate`在`RSocketMessageHandler`切换到不同的策略用于检测客户端响应，例如，诸如基于自定义注释`@RSocketClientResponder`VS默认`@Controller`。在客户端和服务器或同一应用程序中有多个客户端的情况下，这是必需的。

有关编程模型的更多信息，请参见带[注释的响应者](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-annot-responders)。

##### 高级

`RSocketRequesterBuilder`提供了一个回调，以公开基础 `io.rsocket.core.RSocketConnector`的更多配置选项，包括保持活动间隔，会话恢复，拦截器等。您可以按以下方式在该级别上配置选项：

爪哇

科特林

```java
RSocketRequester requester = RSocketRequester.builder()
    .rsocketConnector(connector -> {
        // ...
    })
    .tcp("localhost", 7000);
```

#### 5.2.2。服务器请求者

从服务器向连接的客户端发出请求是从服务器获取连接客户端的请求者的问题。

在带[注释的响应者中](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-annot-responders)，`@ConnectMapping`和`@MessageMapping`方法支持 `RSocketRequester`参数。使用它来访问连接的请求者。请记住，`@ConnectMapping`方法本质上是`SETUP`框架的处理程序，必须在请求开始之前对其进行处理。因此，必须从一开始就将请求与处理分离。例如：

爪哇

科特林

```java
@ConnectMapping
Mono<Void> handle(RSocketRequester requester) {
    requester.route("status").data("5")
        .retrieveFlux(StatusReport.class)
        .subscribe(bar -> { 
            // ...
        });
    return ... 
}
```

|      | 独立于处理，异步启动请求。       |
| ---- | -------------------------------- |
|      | 执行处理并返回完成`Mono<Void>`。 |

#### 5.2.3。要求

有了[客户端](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-requester-client)或 [服务器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-requester-server)请求者后，可以按以下方式发出请求：

爪哇

科特林

```java
ViewBox viewBox = ... ;

Flux<AirportLocation> locations = requester.route("locate.radars.within") 
        .data(viewBox) 
        .retrieveFlux(AirportLocation.class); 
```

|      | 指定要包含在请求消息的元数据中的路由。 |
| ---- | -------------------------------------- |
|      | 提供请求消息的数据。                   |
|      | 声明预期的响应。                       |

交互类型由输入和输出的基数隐式确定。上面的示例是`Request-Stream`因为发送一个值并接收值流。在大多数情况下，只要输入和输出的选择与RSocket交互类型以及响应者期望的输入和输出的类型相匹配，就无需考虑这一点。无效组合的唯一示例是多对一。

该`data(Object)`方法还接受任何Reactive流`Publisher`，包括 `Flux`和`Mono`，以及在中注册的任何其他值的生产者 `ReactiveAdapterRegistry`。对于`Publisher`诸如`Flux`产生相同类型值的多值，请考虑使用重载`data`方法之一，以避免`Encoder`对每个元素进行类型检查和查找：

```java
data(Object producer, Class<?> elementClass);
data(Object producer, ParameterizedTypeReference<?> elementTypeRef);
```

该`data(Object)`步骤是可选的。跳过不发送数据的请求：

爪哇

科特林

```java
Mono<AirportLocation> location = requester.route("find.radar.EWR"))
    .retrieveMono(AirportLocation.class);
```

如果使用[复合元数据](https://github.com/rsocket/rsocket/blob/master/Extensions/CompositeMetadata.md)（默认值）并且注册的值支持这些值，则可以添加额外的元数据值 `Encoder`。例如：

爪哇

科特林

```java
String securityToken = ... ;
ViewBox viewBox = ... ;
MimeType mimeType = MimeType.valueOf("message/x.rsocket.authentication.bearer.v0");

Flux<AirportLocation> locations = requester.route("locate.radars.within")
        .metadata(securityToken, mimeType)
        .data(viewBox)
        .retrieveFlux(AirportLocation.class);
```

对于`Fire-and-Forget`使用`send()`方法，返回`Mono<Void>`。请注意，`Mono` 仅表示成功发送了消息，而不表示已处理。

对于`Metadata-Push`使用`sendMetadata()`方法有`Mono<Void>`返回值。

### 5.3。带注释的响应者

RSocket响应器可以通过`@MessageMapping`和`@ConnectMapping`方法实现。 `@MessageMapping`方法处理单个请求，而`@ConnectMapping`方法处理连接级事件（设置和元数据推送）。对称支持带注释的响应者，用于从服务器端响应和从客户端端响应。

#### 5.3.1。服务器响应者

要在服务器端使用带注释的响应者，请添加`RSocketMessageHandler`到您的Spring配置中以和 方法检测`@Controller`bean ：`@MessageMapping``@ConnectMapping`

爪哇

科特林

```java
@Configuration
static class ServerConfig {

    @Bean
    public RSocketMessageHandler rsocketMessageHandler() {
        RSocketMessageHandler handler = new RSocketMessageHandler();
        handler.routeMatcher(new PathPatternRouteMatcher());
        return handler;
    }
}
```

然后通过Java RSocket API启动RSocket服务器，并`RSocketMessageHandler`为响应者插入 ，如下所示：

爪哇

科特林

```java
ApplicationContext context = ... ;
RSocketMessageHandler handler = context.getBean(RSocketMessageHandler.class);

CloseableChannel server =
    RSocketServer.create(handler.responder())
        .bind(TcpServerTransport.create("localhost", 7000))
        .block();
```

`RSocketMessageHandler`默认情况下支持 [复合](https://github.com/rsocket/rsocket/blob/master/Extensions/CompositeMetadata.md)和 [路由](https://github.com/rsocket/rsocket/blob/master/Extensions/Routing.md)元数据。如果需要切换到其他mime类型或注册其他元数据mime类型，则可以设置其 [MetadataExtractor](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-metadata-extractor)。

您需要设置支持元数据和数据格式所需的`Encoder`和`Decoder`实例。您可能需要`spring-web`用于编解码器实现的模块。

默认情况下`SimpleRouteMatcher`用于通过匹配路由`AntPathMatcher`。我们建议插入`PathPatternRouteMatcher`from`spring-web`以进行有效的路由匹配。RSocket路由可以是分层的，但不是URL路径。两个路由匹配器都配置为使用“。” 默认为分隔符，并且没有HTTP URL那样的URL解码。

`RSocketMessageHandler`可以进行配置`RSocketStrategies`，如果您需要在同一过程中在客户端和服务器之间共享配置，则可以通过它进行配置：

爪哇

科特林

```java
@Configuration
static class ServerConfig {

    @Bean
    public RSocketMessageHandler rsocketMessageHandler() {
        RSocketMessageHandler handler = new RSocketMessageHandler();
        handler.setRSocketStrategies(rsocketStrategies());
        return handler;
    }

    @Bean
    public RSocketStrategies rsocketStrategies() {
        return RSocketStrategies.builder()
            .encoders(encoders -> encoders.add(new Jackson2CborEncoder()))
            .decoders(decoders -> decoders.add(new Jackson2CborDecoder()))
            .routeMatcher(new PathPatternRouteMatcher())
            .build();
    }
}
```

#### 5.3.2。客户回应者

客户端中带注释的响应者需要在中配置 `RSocketRequester.Builder`。有关详细信息，请参阅 [客户响应者](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-requester-client-responder)。

#### 5.3.3。@MessageMapping

一旦[服务器](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-annot-responders-server)或 [客户机](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-annot-responders-client)应答器的配置就位， `@MessageMapping`方法可以被使用如下：

爪哇

科特林

```java
@Controller
public class RadarsController {

    @MessageMapping("locate.radars.within")
    public Flux<AirportLocation> radars(MapRequest request) {
        // ...
    }
}
```

上述`@MessageMapping`方法响应具有路由“ locate.radars.within”的请求-流交互。它支持灵活的方法签名，并可以选择使用以下方法参数：

| 方法参数                       | 描述                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| `@Payload`                     | 请求的有效负载。这可以是异步类型（例如`Mono`或）的具体值 `Flux`。**注意：**注释的使用是可选的。并非简单类型并且不是其他任何受支持参数的方法参数都假定为预期的有效负载。 |
| `RSocketRequester`             | 向远端发出请求的请求者。                                     |
| `@DestinationVariable`         | 根据映射模式中的变量从路线提取的值，例如 `@MessageMapping("find.radar.{id}")`。 |
| `@Header`                      | 如在所描述的元数据值注册为萃取[MetadataExtractor](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-metadata-extractor)。 |
| `@Headers Map<String, Object>` | 如[MetadataExtractor中](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-metadata-extractor)所述，注册所有用于提取的元数据值。 |

返回值应为一个或多个要序列化为响应有效负载的对象。可以是异步类型，例如`Mono`或`Flux`，具体值，也可以`void`是无值异步类型，例如`Mono<Void>`。

`@MessageMapping`方法支持的RSocket交互类型由输入（即`@Payload`参数）和输出的基数确定，其中基数表示以下含义：

| 基数 | 描述                                                         |
| :--- | :----------------------------------------------------------- |
| 1个  | 显式值或单值异步类型（例如）`Mono<T>`。                      |
| 许多 | 多值异步类型，例如`Flux<T>`。                                |
| 0    | 对于输入，这意味着该方法没有`@Payload`参数。对于输出，这是`void`一个或无值异步类型，例如`Mono<Void>`。 |

下表显示了所有输入和输出基数组合以及相应的交互类型：

| 输入基数 | 输出基数   | 互动类型           |
| :------- | :--------- | :----------------- |
| 0，1     | 0          | 一劳永逸，请求响应 |
| 0，1     | 1个        | 请求-响应          |
| 0，1     | 许多       | 请求流             |
| 许多     | 0、1，很多 | 请求通道           |

#### 5.3.4。@ConnectMapping

`@ConnectMapping``SETUP`在RSocket连接的开始处处理该框架，并通过该`METADATA_PUSH`框架（即 `metadataPush(Payload)`在中）处理所有后续的元数据推送通知`io.rsocket.RSocket`。

`@ConnectMapping`方法支持与[@MessageMapping](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-annot-messagemapping)相同的参数， 但基于`SETUP`和 `METADATA_PUSH`帧中的元数据和数据。`@ConnectMapping`可以使用某种模式将处理范围缩小到元数据中具有路由的特定连接，或者如果未声明任何模式，则所有连接都匹配。

`@ConnectMapping`方法不能返回数据，必须用`void`或 `Mono<Void>`作为返回值进行声明。如果处理返回新连接错误，则连接被拒绝。不得阻止向`RSocketRequester`连接请求的处理。有关详细信息，请参见 [服务器请求](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#rsocket-requester-server)程序。

### 5.4。元数据提取器

响应者必须解释元数据。 [复合元数据](https://github.com/rsocket/rsocket/blob/master/Extensions/CompositeMetadata.md)允许独立格式化的元数据值（例如，用于路由，安全性，跟踪），每个元数据值都有自己的mime类型。应用程序需要一种配置要支持的元数据MIME类型的方法，以及一种访问提取值的方法。

`MetadataExtractor`是获取序列化元数据并返回已解码名称/值对的合同，然后可以按名称（例如，通过带`@Header` 注释的处理程序方法）像标题一样对其进行访问。

`DefaultMetadataExtractor`可以提供`Decoder`实例来解码元数据。开箱即用，它具有对[“ message / x.rsocket.routing.v0”的](https://github.com/rsocket/rsocket/blob/master/Extensions/Routing.md)内置支持， 它可以解码 `String`并保存在“ route”键下。对于任何其他mime类型，您需要提供a`Decoder`并注册mime类型，如下所示：

爪哇

科特林

```java
DefaultMetadataExtractor extractor = new DefaultMetadataExtractor(metadataDecoders);
extractor.metadataToExtract(fooMimeType, Foo.class, "foo");
```

复合元数据可以很好地组合独立的元数据值。但是，请求者可能不支持复合元数据，或者可以选择不使用它。为此， `DefaultMetadataExtractor`可能需要自定义逻辑将解码后的值映射到输出映射。这是将JSON用于元数据的示例：

爪哇

科特林

```java
DefaultMetadataExtractor extractor = new DefaultMetadataExtractor(metadataDecoders);
extractor.metadataToExtract(
    MimeType.valueOf("application/vnd.myapp.metadata+json"),
    new ParameterizedTypeReference<Map<String,String>>() {},
    (jsonMap, outputMap) -> {
        outputMap.putAll(jsonMap);
    });
```

通过配置`MetadataExtractor`时`RSocketStrategies`，您可以 `RSocketStrategies.Builder`使用配置的解码器创建提取器，并只需使用回调即可自定义注册，如下所示：

爪哇

科特林

```java
RSocketStrategies strategies = RSocketStrategies.builder()
    .metadataExtractorRegistry(registry -> {
        registry.metadataToExtract(fooMimeType, Foo.class, "foo");
        // ...
    })
    .build();
```

## 6.Reactive图书馆

`spring-webflux`在`reactor-core`内部依赖并使用它来构成异步逻辑并提供响应流支持。通常，WebFlux API返回`Flux`或 `Mono`（因为它们在内部使用）并且宽容地接受任何响应流 `Publisher`实现作为输入。使用`Flux`vs`Mono`是重要的，因为它有助于表达基数-例如，期望单个或多个异步值，并且这对于进行决策（例如，在编码或解码HTTP消息时）至关重要。

对于带注释的控制器，WebFlux透明地适应应用程序选择的Reactive库。这是在的帮助下完成的 [`ReactiveAdapterRegistry`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/core/ReactiveAdapterRegistry.html)，该工具为Reactive库和其他异步类型提供了可插入的支持。该注册表具有对RxJava 2/3，RxJava 1（通过RxJava 响应流桥）和的内置支持 `CompletableFuture`，但是您也可以注册其他内容。

|      | 从Spring Framework 5.3开始，不支持RxJava 1。 |
| ---- | -------------------------------------------- |
|      |                                              |

对于功能性API（例如，[功能性终结点](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn)，`WebClient`和等），适用WebFlux API的一般规则- `Flux`并`Mono`作为返回值和Reactive流 `Publisher`作为输入。如果提供`Publisher`，无论是自定义的还是来自其他响应式库的，都只能将其视为语义未知（0..N）的流。但是，如果语义是已知的，你可以把它包装`Flux`或`Mono.from(Publisher)`强似原料`Publisher`。

例如，给定的`Publisher`不是a `Mono`，Jackson JSON消息编写器期望多个值。如果媒体类型暗示无限流（例如`application/json+stream`），则将 分别写入和刷新值。否则，值将缓冲到列表中并呈现为JSON数组。

版本5.3.1
最后更新2020-11-10 08:42:36 UTC