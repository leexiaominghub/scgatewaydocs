# Spring Cloud Contract

 **2.2.4。发布**

[ ](https://github.com/spring-cloud/spring-cloud-contract)

- [总览](https://spring.io/projects/spring-cloud-contract#overview)
- [学习](https://spring.io/projects/spring-cloud-contract#learn)
- [例子](https://spring.io/projects/spring-cloud-contract#samples)

Spring Cloud Contract是一个总括项目解决方案，可帮助用户成功实施“消费者驱动合约”方法。当前，Spring Cloud Contract由Spring Cloud Contract Verifier项目组成。

Spring Cloud Contract Verifier是一种工具，支持基于JVM的应用程序的消费者驱动合约（CDC）开发。它附带了用Groovy或YAML编写的合约定义语言（DSL）。合约定义用于产生以下资源：

- 默认情况下，在对客户端代码进行集成测试（客户端测试）时，WireMock（HTTP服务器桩）将使用JSON桩定义。测试代码仍然必须是手工编写的，测试数据是由Spring Cloud Contract Verifier生成的。
- 消息传递路线（如果您使用的是一条）。我们正在与Spring Integration，Spring Cloud Stream和Apache Camel集成。但是，您可以根据需要设置自己的集成。
- 验收测试（在JUnit或Spock中默认为），用于验证API的服务器端实现是否符合合约（服务器测试）。完整测试由Spring Cloud Contract Verifier生成。

Spring Cloud Contract Verifier将TDD移至软件架构级别。

要查看Spring Cloud Contract如何支持其他语言，只需查看[此博客文章](https://spring.io/blog/2018/02/13/spring-cloud-contract-in-a-polyglot-world)。

## 特性

当尝试测试与其他服务通信的应用程序时，我们可以做两件事之一：

- 部署所有微服务并执行端到端测试
- 在单元/集成测试中模拟其他微服务

两者都有优点，也有很多缺点。让我们专注于后者。部署所有微服务并执行端到端测试

好处：

- 模拟生产
- 测试服务之间的真实通信

缺点：

- 要测试一个微服务，我们将必须部署6个微服务，几个数据库等。
- 进行测试的环境将被锁定为一组测试（即，在此期间没有其他人能够运行测试）。
- 渴望运行
- 反馈很晚
- 极难调试

在单元/集成测试中模拟其他微服务

好处：

- 非常快的反馈
- 没有基础设施要求

缺点：

- 服务的实现者会创建桩，因此它们可能与现实无关
- 您可以通过测试并通过失败的生产

为了解决上述问题，创建了带有Stub Runner的Spring Cloud Contract Verifier。他们的主要思想是在不建立整个微服务世界的情况下，为您提供非常快速的反馈。

Spring Cloud Contract Verifier的功能：

- 确保HTTP / Messaging桩（在开发客户端时使用）确实在执行实际的服务器端实现
- 促进验收测试驱动的开发方法和微服务架构风格
- 提供一种发布合约更改的方法，该更改在通讯的两侧立即可见
- 生成服务器端使用的样板测试代码

## Spring Boot配置

## 在生产者方面

要开始使用Spring Cloud Contract，您可以将具有REST的文件或以Groovy DSL或YAML表示的消息传递合约添加到由contractsDslDir属性设置的合约目录中。默认情况下，它是$ rootDir / src / test / resources / contracts。

然后，您可以将Spring Cloud Contract Verifier依赖项和插件添加到您的构建文件中，如以下示例所示：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-verifier</artifactId>
    <scope>test</scope>
</dependency>复制
```

以下清单显示了如何添加插件，该插件应放在文件的build / plugins部分中：

```xml
<plugin>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-maven-plugin</artifactId>
    <version>${spring-cloud-contract.version}</version>
    <extensions>true</extensions>
</plugin>复制
```

运行会`./mvnw clean install`自动生成测试，以验证应用程序是否符合添加的合约。默认情况下，将在下生成测试`org.springframework.cloud.contract.verifier.tests`。

由于尚不存在合约描述的功能的实现，因此测试失败。

要使它们通过，您必须添加处理HTTP请求或消息的正确实现。另外，必须将用于自动生成的测试的基本测试类添加到项目。该类由所有自动生成的测试扩展，并且应包含运行它们所需的所有设置信息（例如，`RestAssuredMockMvc`控制器设置或消息传递测试设置）。

以下来自pom.xml的示例显示了如何指定基本测试类：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-contract-maven-plugin</artifactId>
            <version>${spring-cloud-contract.version}</version>
            <extensions>true</extensions>
            <configuration>
                <baseClassForTests>com.example.contractTest.BaseTestClass</baseClassForTests>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>复制
```

信息：baseClassForTests元素使您可以指定基本测试类。它必须是spring-cloud-contract-maven-plugin中配置元素的子代。

一旦实现和测试基类就位，测试就会通过，并且将应用程序和桩工件都构建并安装在本地Maven存储库中。现在，您可以合并更改，并且可以在在线存储库中发布应用程序和桩工件。

## 在消费者方面

您可以在集成测试中使用Spring Cloud Contract Stub Runner获得模拟模拟实际服务的正在运行的WireMock实例或消息传递路由。

为此，将依赖项添加到Spring Cloud Contract Stub Runner，如以下示例所示：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
    <scope>test</scope>
</dependency>复制
```

您可以通过以下两种方式之一在Maven存储库中安装生产者端桩：

通过签出生产者端存储库并添加合约并通过运行以下命令来生成桩：

```bash
$ cd local-http-server-repo
$ ./mvnw clean install -DskipTests复制
```

由于生产者方合约实施尚未到位，因此跳过了测试，因此自动生成的合约测试失败。

通过从远程存储库获取已经存在的生产者服务桩。为此，将桩工件ID和工件存储库URL作为Spring Cloud Contract Stub Runner属性传递，如以下示例所示：

```yml
    stubrunner:
      ids: 'com.example:http-server-dsl:+:stubs:8080'
      repositoryRoot: https://repo.spring.io/libs-snapshot复制
```

现在，您可以使用注释您的测试课程`@AutoConfigureStubRunner`。在批注中，为Spring Cloud Contract Stub Runner提供group-id和artifact-id值，以为您运行协作者的桩，如以下示例所示：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.NONE)
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:stubs:6565"},
        stubsMode = StubRunnerProperties.StubsMode.LOCAL)
public class LoanApplicationServiceTests {复制
```

`REMOTE`从在线存储库下载桩和`LOCAL`进行脱机工作时，请使用stubsMode 。

现在，在集成测试中，您可以接收预期由协作服务发出的HTTP响应或消息的桩版本。