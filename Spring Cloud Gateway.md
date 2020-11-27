# Spring Cloud Gateway

**2.2.5。发布**

该项目提供了一个在Spring生态系统之上构建的API网关，包括：Spring 5，Spring Boot 2和Project Reactor。Spring Cloud Gateway旨在提供一种简单而有效的方法来路由到API，并为它们提供跨领域的关注点，例如：安全性，监视/指标和弹性。

## [1.如何包括Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-starter)

要将Spring Cloud Gateway包含在您的项目中，请使用具有org.springframework.cloud group ID`和artifact ID的启动器`spring-cloud-starter-gateway`。有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参见[Spring Cloud Project页面](https://projects.spring.io/spring-cloud/)。

如果包括启动器，但不希望启用网关，请设置`spring.cloud.gateway.enabled=false`。

|      | Spring Cloud Gateway是基于[Spring Boot 2.x](https://spring.io/projects/spring-boot#learn)，[Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html)和[Project Reactor](https://projectreactor.io/docs)[构建的](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html)。结果，当您使用Spring Cloud Gateway时，许多您熟悉的同步库（例如，Spring Data和Spring Security）和模式可能不适用。如果您不熟悉这些项目，建议您在使用Spring Cloud Gateway之前先阅读它们的文档以熟悉一些新概念。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | Spring Cloud Gateway需要Spring Boot和Spring Webflux提供的Netty运行时。它不能在传统的Servlet容器中或作为WAR构建时使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## [2.词汇表](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#glossary)

- **路线**：网关的基本构建块。它由ID，目标URI，谓词集合和过滤器集合定义。如果聚合谓词为true，则匹配路由。
- **谓词**：这是[Java 8函数谓词](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html)。输入类型是[Spring Framework`ServerWebExchange`](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/server/ServerWebExchange.html)。这使您可以匹配HTTP请求中的所有内容，例如标头或参数。
- **Filter**：这些是使用特定工厂构造的[Spring Framework`GatewayFilter`](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/server/GatewayFilter.html)实例。在这里，您可以在发送下游请求之前或之后修改请求和响应。

## [3.工作原理](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-how-it-works)

下图从总体上概述了Spring Cloud Gateway的工作方式：

![Spring Cloud网关图](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/images/spring_cloud_gateway_diagram.png)

客户端向Spring Cloud Gateway发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序。该处理程序通过特定于请求的过滤器链运行请求。筛选器由虚线分隔的原因是，筛选器可以在发送代理请求之前和之后运行逻辑。所有“前置”过滤器逻辑均被执行。然后发出代理请求。发出代理请求后，将运行“后”过滤器逻辑。

|      | 在没有端口的路由中定义的URI，HTTP和HTTPS URI的默认端口值分别为80和443。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## [4.配置路由谓词工厂和网关过滤工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#configuring-route-predicate-factories-and-gateway-filter-factories)

有两种配置谓词和过滤器的方法：快捷方式和完全扩展的参数。下面的大多数示例都使用快捷方式。

名称和自变量名称将`code`在每个部分的第一个或两个中列出。参数通常按快捷方式配置所需的顺序列出。

### [4.1。快捷方式配置](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#shortcut-configuration)

快捷方式配置由过滤器名称识别，后跟一个等号（`=`），然后是由逗号分隔的参数值（`,`）。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

上一个示例`Cookie`使用两个参数（cookie名称`mycookie`和match的值）定义了Route Predicate Factory `mycookievalue`。

### [4.2。完全展开的论点](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#fully-expanded-arguments)

完全展开的参数看起来更像带有名称/值对的标准Yaml配置。通常，将有一个`name`钥匙和一个`args`钥匙。的`args`关键是地图密钥值对的配置谓词或过滤器。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue
```

这是`Cookie`上面显示的谓词的快捷方式配置的完整配置。

## [5.路由谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-request-predicates-factories)

Spring Cloud Gateway将路由匹配作为Spring WebFlux`HandlerMapping`基础架构的一部分。Spring Cloud Gateway包括许多内置的路由谓词工厂。所有这些谓词都与HTTP请求的不同属性匹配。您可以将多个路由谓词工厂与逻辑`and`语句结合使用。

### [5.1。后路线谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-after-route-predicate-factory)

所述`After`路线谓词工厂有一个参数，一个`datetime`（其是Java `ZonedDateTime`）。该谓词匹配在指定日期时间之后发生的请求。下面的示例配置路由后谓词：

例子1. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

这条路线符合2017年1月20日17:42山区时间（丹佛）之后的任何请求。

### [5.2。之前路线谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-before-route-predicate-factory)

所述`Before`路线谓词工厂有一个参数，一个`datetime`（其是Java `ZonedDateTime`）。该谓词匹配在指定之前发生的请求`datetime`。下面的示例配置路由前谓词：

例子2. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

这条路线符合2017年1月20日17:42山区时间（丹佛）之前的任何要求。

### [5.3。路由谓词间工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-between-route-predicate-factory)

该`Between`路线谓词工厂有两个参数，`datetime1`并且`datetime2` 这是Java`ZonedDateTime`对象。该谓词匹配之后`datetime1`和之前发生的请求`datetime2`。该`datetime2`参数必须是后`datetime1`。以下示例配置了路由之间的谓词：

例子3. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

此路线与2017年1月20日山区时间（丹佛）之后和2017年1月21日17:42山区时间（丹佛）之后的任何请求相匹配。这对于维护时段可能很有用。

### [5.4。Cookie路线谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-cookie-route-predicate-factory)

所述`Cookie`路线谓词工厂采用两个参数，该cookie`name`和`regexp`（其是Java正则表达式）。该谓词匹配具有给定名称且其值与正则表达式匹配的cookie。以下示例配置cookie路由谓词工厂：

例子4. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

此路由匹配具有名称为`chocolate`与`ch.p`正则表达式匹配的cookie的请求。

### [5.5。标头路由谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-header-route-predicate-factory)

所述`Header`路线谓词工厂采用两个参数，报头`name`和一个`regexp`（其是Java正则表达式）。该谓词与具有给定名称的标头匹配，该标头的值与正则表达式匹配。以下示例配置标头路由谓词：

例子5. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

如果请求具有名为`X-Request-Id`其值与`\d+`正则表达式匹配的标头（即，其值为一个或多个数字），则此路由匹配。

### [5.6。主机路由谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-host-route-predicate-factory)

该`Host`路线谓词工厂需要一个参数：主机名的列表`patterns`。该模式是带有`.`分隔符的Ant样式的模式。谓词与`Host`匹配模式的标头匹配。以下示例配置主机路由谓词：

例子6. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

`{sub}.myhost.org`还支持URI模板变量（例如）。

如果请求具有这种路由匹配`Host`用的头值`www.somehost.org`或`beta.somehost.org`或`www.anotherhost.org`。

该谓词提取URI模板变量（例如`sub`，在前面的示例中定义的）作为名称和值的映射，并`ServerWebExchange.getAttributes()`使用中定义的键将其放在中`ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE`。这些值可供[工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-route-filters)使用[`GatewayFilter`](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-route-filters)

### [5.7。方法路线谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-method-route-predicate-factory)

所述`Method`路线谓词厂需要`methods`的参数，它是一个或多个参数：HTTP方法来匹配。以下示例配置方法route谓词：

例子7. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

如果请求方法是a`GET`或a，则此路由匹配`POST`。

### [5.8。路径路线谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-path-route-predicate-factory)

该`Path`路线谓词厂有两个参数：春天的列表`PathMatcher` `patterns`和一个可选的标志叫`matchOptionalTrailingSeparator`。以下示例配置路径路由谓词：

例子8. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

：这条路线，如果请求路径是，例如匹配`/red/1`或`/red/blue`或`/blue/green`。

该谓词提取URI模板变量（例如`segment`，在前面的示例中定义的）作为名称和值的映射，并`ServerWebExchange.getAttributes()`使用中定义的键将其放在中`ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE`。这些值可供[工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-route-filters)使用[`GatewayFilter`](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-route-filters)

可以使用实用程序方法（称为`get`）来简化对这些变量的访问。下面的示例演示如何使用该`get`方法：

```java
Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

String segment = uriVariables.get("segment");
```

### [5.9。查询路由谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-query-route-predicate-factory)

所述`Query`路线谓词工厂采用两个参数：所要求的`param`和可选的`regexp`（其是Java正则表达式）。以下示例配置查询路由谓词：

例子9. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=green
```

如果请求包含`green`查询参数，则前面的路由匹配。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree.
```

如果请求包含一个前述路线匹配`red`，其值相匹配的查询参数`gree.`的regexp，所以`green`和`greet`将匹配。

### [5.10。RemoteAddr路由谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-remoteaddr-route-predicate-factory)

所述`RemoteAddr`路线谓词工厂需要的列表（分钟尺寸1） `sources`，其是CIDR的表示法（IPv4或IPv6）的字符串，如`192.168.0.1/16`（其中`192.168.0.1`是一个IP地址和`16`一个子网掩码）。以下示例配置一个RemoteAddr路由谓词：

例子10. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

如果请求的远程地址为，则此路由匹配`192.168.1.10`。

### [5.11。重量路线谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-weight-route-predicate-factory)

该`Weight`路线谓词工厂有两个参数：`group`和`weight`（一个int）。权重是按组计算的。以下示例配置权重路由谓词：

例子11. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

这条路线会将约80％的流量转发至[weighthigh.org，](https://weighthigh.org/)并将约20％的流量[转发](https://weighlow.org/)至[weightlow.org。](https://weighlow.org/)

#### [5.11.1。修改远程地址的解析方式](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#modifying-the-way-remote-addresses-are-resolved)

默认情况下，RemoteAddr路由谓词工厂使用传入请求中的远程地址。如果Spring Cloud Gateway位于代理层后面，则可能与实际的客户端IP地址不匹配。

您可以通过设置custom来定制解析远程地址的方式`RemoteAddressResolver`。春季云网关附带了一个基于的关闭一个非默认的远程地址解析器[X -转发，对于头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)，`XForwardedRemoteAddressResolver`。

`XForwardedRemoteAddressResolver` 有两个静态构造函数方法，它们采用不同的安全性方法：

- `XForwardedRemoteAddressResolver::trustAll`返回一个`RemoteAddressResolver`始终采用`X-Forwarded-For`标头中找到的第一个IP地址的。这种方法容易受到欺骗的攻击，因为恶意客户端可能会为设置初始值，`X-Forwarded-For`解析器会接受该初始值。
- `XForwardedRemoteAddressResolver::maxTrustedIndex`采用与Spring Cloud Gateway前面运行的受信任基础结构数量相关的索引。例如，如果只能通过HAProxy访问Spring Cloud Gateway，则应使用值1。如果在访问Spring Cloud Gateway之前需要两跳可信基础架构，则应使用值2。

考虑以下标头值：

```
X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3
```

以下`maxTrustedIndex`值产生以下远程地址：

| `maxTrustedIndex`         | 结果                                           |
| :------------------------ | :--------------------------------------------- |
| [ `Integer.MIN_VALUE`，0] | （`IllegalArgumentException`在初始化期间无效） |
| 1个                       | 0.0.0.3                                        |
| 2                         | 0.0.0.2                                        |
| 3                         | 0.0.0.1                                        |
| [4，`Integer.MAX_VALUE`]  | 0.0.0.1                                        |

以下示例显示了如何使用Java实现相同的配置：

例子12. GatewayConfig.java

```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```

## [6.`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gatewayfilter-factories)

路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路由过滤器适用于特定路由。Spring Cloud Gateway包括许多内置的GatewayFilter工厂。

|      | 有关如何使用以下任何过滤器的更多详细示例，请查看[单元测试](https://github.com/spring-cloud/spring-cloud-gateway/tree/master/spring-cloud-gateway-core/src/test/java/org/springframework/cloud/gateway/filter/factory)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [6.1。该`AddRequestHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-addrequestheader-gatewayfilter-factory)

该`AddRequestHeader` `GatewayFilter`工厂需要`name`和`value`参数。以下示例配置了`AddRequestHeader` `GatewayFilter`：

例子13. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```

此清单将`X-Request-red:blue`标头添加到所有匹配请求的下游请求的标头中。

`AddRequestHeader`了解用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时扩展。以下示例配置了`AddRequestHeader` `GatewayFilter`使用变量的：

例子14. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```

### [6.2。该`AddRequestParameter` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-addrequestparameter-gatewayfilter-factory)

该`AddRequestParameter` `GatewayFilter`工厂需要`name`和`value`参数。以下示例配置了`AddRequestParameter` `GatewayFilter`：

例子15. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        filters:
        - AddRequestParameter=red, blue
```

这将添加`red=blue`到所有匹配请求的下游请求的查询字符串中。

`AddRequestParameter`了解用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时扩展。以下示例配置了`AddRequestParameter` `GatewayFilter`使用变量的：

例子16. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddRequestParameter=foo, bar-{segment}
```

### [6.3。该`AddResponseHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-addresponseheader-gatewayfilter-factory)

该`AddResponseHeader` `GatewayFilter`工厂需要`name`和`value`参数。以下示例配置了`AddResponseHeader` `GatewayFilter`：

例子17. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        filters:
        - AddResponseHeader=X-Response-Red, Blue
```

这会将`X-Response-Foo:Bar`标头添加到所有匹配请求的下游响应的标头中。

`AddResponseHeader`知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时扩展。以下示例配置了`AddResponseHeader` `GatewayFilter`使用变量的：

例子18. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddResponseHeader=foo, bar-{segment}
```

### [6.4。该`DedupeResponseHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-deduperesponseheader-gatewayfilter-factory)

DedupeResponseHeader GatewayFilter工厂采用一个`name`参数和一个可选`strategy`参数。`name`可以包含以空格分隔的标题名称列表。以下示例配置了`DedupeResponseHeader` `GatewayFilter`：

例子19. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: dedupe_response_header_route
        uri: https://example.org
        filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

当网关CORS逻辑和下游逻辑都将它们添加时，这将删除`Access-Control-Allow-Credentials`和`Access-Control-Allow-Origin`响应头的重复值。

该`DedupeResponseHeader`过滤器还接受一个可选的`strategy`参数。可接受的值为`RETAIN_FIRST`（默认值）`RETAIN_LAST`，和`RETAIN_UNIQUE`。

### [6.5。Hystrix`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#hystrix)

|      | [Netflix已将Hystrix置于维护模式](https://cloud.spring.io/spring-cloud-netflix/multi/multi__modules_in_maintenance_mode.html)。我们建议您将[Spring Cloud CircuitBreaker网关过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#spring-cloud-circuitbreaker-filter-factory)与Resilience4J一起使用，因为在将来的版本中将不再支持Hystrix。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

[Hystrix](https://github.com/Netflix/Hystrix)是Netflix的一个库，用于实现[断路器模式](https://martinfowler.com/bliki/CircuitBreaker.html)。将`Hystrix` `GatewayFilter`让你介绍断路器和网关路由，保护您的服务从级联故障，让你提供下游故障时备用的响应。

要`Hystrix` `GatewayFilter`在项目中启用实例，请`spring-cloud-starter-netflix-hystrix`从[Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/)添加依赖项。

该`Hystrix` `GatewayFilter`工厂需要一个`name`参数，它是的名称`HystrixCommand`。以下示例配置Hystrix `GatewayFilter`：

例子20. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: https://example.org
        filters:
        - Hystrix=myCommandName
```

这会将其余的过滤器包装在`HystrixCommand`命令名称为的中`myCommandName`。

Hystrix过滤器还可以接受可选`fallbackUri`参数。当前，仅`forward:`支持计划的URI。如果调用后备，则请求将转发到与URI匹配的控制器。以下示例配置了这种后备：

例子21. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingserviceendpoint
        filters:
        - name: Hystrix
          args:
            name: fallbackcmd
            fallbackUri: forward:/incaseoffailureusethis
        - RewritePath=/consumingserviceendpoint, /backingserviceendpoint
```

`/incaseoffailureusethis`调用Hystrix后备时，它将转发到URI。请注意，此示例还演示了（可选）Spring Cloud Netflix Ribbon负载平衡（`lb`在目标URI上定义了前缀）。

主要方案是对`fallbackUri`网关应用程序中的内部控制器或处理程序使用。但是，您还可以将请求重新路由到外部应用程序中的控制器或处理程序，如下所示：

例子22. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: Hystrix
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```

在此示例中，`fallback`网关应用程序中没有端点或处理程序。但是，另一个应用程序中有一个，在处注册`localhost:9994`。

如果将请求转发给后备，则Hystrix网关过滤器还会提供`Throwable`引起请求的。它`ServerWebExchange`作为`ServerWebExchangeUtils.HYSTRIX_EXECUTION_EXCEPTION_ATTR`属性添加到中，您可以在处理网关应用程序中的后备时使用该属性。

对于外部控制器/处理程序方案，可以添加带有异常详细信息的标头。您可以在[FallbackHeaders GatewayFilter Factory部分中](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#fallback-headers)找到有关此操作的更多信息。

您可以使用应用程序属性使用全局默认值或逐条路由配置Hystrix设置（例如超时），如[Hystrix Wiki](https://github.com/Netflix/Hystrix/wiki/Configuration)上所述。

要为前面显示的示例路由设置五秒钟的超时时间，可以使用以下配置：

例子23. application.yml

```yaml
hystrix.command.fallbackcmd.execution.isolation.thread.timeoutInMilliseconds: 5000
```

### [6.6。Spring Cloud CircuitBreaker GatewayFilter工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#spring-cloud-circuitbreaker-filter-factory)

Spring Cloud CircuitBreaker GatewayFilter工厂使用Spring Cloud CircuitBreaker API将网关路由包装在断路器中。Spring Cloud CircuitBreaker支持可与Spring Cloud Gateway一起使用的两个库Hystrix和Resilience4J。由于Netflix已将Hystrix置于仅维护模式，因此建议您使用Resilience4J。

要启用Spring Cloud CircuitBreaker过滤器，您需要将`spring-cloud-starter-circuitbreaker-reactor-resilience4j`或`spring-cloud-starter-netflix-hystrix`放置在类路径上。以下示例配置了Spring Cloud CircuitBreaker `GatewayFilter`：

例子24. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: https://example.org
        filters:
        - CircuitBreaker=myCircuitBreaker
```

要配置断路器，请参阅所使用的基础断路器实现的配置。

- [Resilience4J文档](https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/spring-cloud-circuitbreaker.html)
- [Hystrix文档](https://cloud.spring.io/spring-cloud-netflix/reference/html/)

Spring Cloud CircuitBreaker过滤器也可以接受可选`fallbackUri`参数。当前，仅`forward:`支持计划的URI。如果调用后备，则请求将转发到与URI匹配的控制器。以下示例配置了这种后备：

例子25. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
        - RewritePath=/consumingServiceEndpoint, /backingServiceEndpoint
```

以下清单在Java中做同样的事情：

例子26. Application.java

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```

`/inCaseofFailureUseThis`当调用断路器回退时，此示例将转发到URI。请注意，此示例还演示了（可选）Spring Cloud Netflix Ribbon负载平衡（由`lb`目标URI上的前缀定义）。

主要方案是使用`fallbackUri`定义网关应用程序中的内部控制器或处理程序。但是，您还可以将请求重新路由到外部应用程序中的控制器或处理程序，如下所示：

例子27. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```

在此示例中，`fallback`网关应用程序中没有端点或处理程序。但是，另一个应用程序中有一个，在处注册`localhost:9994`。

如果将请求转发给后备，则Spring Cloud CircuitBreaker网关过滤器还会提供`Throwable`引起请求的。它`ServerWebExchange`作为`ServerWebExchangeUtils.CIRCUITBREAKER_EXECUTION_EXCEPTION_ATTR`属性添加到中，该属性可以在处理网关应用程序中的后备时使用。

对于外部控制器/处理程序方案，可以添加带有异常详细信息的标头。您可以在[FallbackHeaders GatewayFilter Factory部分中](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#fallback-headers)找到有关此操作的更多信息。

#### [6.6.1。根据状态码使断路器跳闸](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#circuit-breaker-status-codes)

在某些情况下，您可能希望根据断路器包装的路线返回的状态代码来使断路器跳闸。断路器配置对象获取状态代码列表，如果返回该状态代码，将导致断路器跳闸。设置要使断路器跳闸的状态码时，可以使用带状态码值的整数或`HttpStatus`枚举的字符串表示形式。

例子28. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
            statusCodes:
              - 500
              - "NOT_FOUND"
```

例子29. Application.java

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis").addStatusCode("INTERNAL_SERVER_ERROR"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```

### [6.7。该`FallbackHeaders` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#fallback-headers)

通过`FallbackHeaders`工厂，您可以在转发到`fallbackUri`外部应用程序中的请求的标头中添加Hystrix或Spring Cloud CircuitBreaker执行异常详细信息，如以下情况所示：

例子30. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

在此示例中，在运行断路器时发生执行异常之后，该请求将转发到在上`fallback`运行的应用程序中的端点或处理程序`localhost:9994`。`FallbackHeaders`过滤器会将具有异常类型，消息和（如果有）根本原因异常类型和消息的标头添加到该请求。

您可以通过设置以下参数的值（以其默认值显示）来覆盖配置中标头的名称：

- `executionExceptionTypeHeaderName`（`"Execution-Exception-Type"`）
- `executionExceptionMessageHeaderName`（`"Execution-Exception-Message"`）
- `rootCauseExceptionTypeHeaderName`（`"Root-Cause-Exception-Type"`）
- `rootCauseExceptionMessageHeaderName`（`"Root-Cause-Exception-Message"`）

有关断路器和网关的更多信息，请参见[Hystrix GatewayFilter Factory部分](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#hystrix)或[Spring Cloud CircuitBreaker Factory部分](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#spring-cloud-circuitbreaker-filter-factory)。

### [6.8。该`MapRequestHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-maprequestheader-gatewayfilter-factory)

该`MapRequestHeader` `GatewayFilter`工厂采用`fromHeader`和`toHeader`参数。它将创建一个新的命名标头（`toHeader`），然后`fromHeader`从传入的http请求中将其值从现有的命名标头（）中提取出来。如果输入标头不存在，则过滤器不起作用。如果新的命名头已经存在，则其值将使用新值进行扩充。以下示例配置了`MapRequestHeader`：

例子31. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: map_request_header_route
        uri: https://example.org
        filters:
        - MapRequestHeader=Blue, X-Request-Red
```

这会将`X-Request-Red:<values>`标头添加到下游请求中，并带有来自传入HTTP请求`Blue`标头的更新值。

### [6.9。该`PrefixPath` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-prefixpath-gatewayfilter-factory)

该`PrefixPath` `GatewayFilter`工厂采用单个`prefix`参数。以下示例配置了`PrefixPath` `GatewayFilter`：

例子32. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```

这将`/mypath`作为所有匹配请求的路径的前缀。因此，`/hello`将向发送请求`/mypath/hello`。

### [6.10。该`PreserveHostHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-preservehostheader-gatewayfilter-factory)

该`PreserveHostHeader` `GatewayFilter`工厂没有参数。此过滤器设置请求属性，路由过滤器将检查该请求属性以确定是否应发送原始主机头，而不是由HTTP客户端确定的主机头。以下示例配置了`PreserveHostHeader` `GatewayFilter`：

例子33. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: preserve_host_route
        uri: https://example.org
        filters:
        - PreserveHostHeader
```

### [6.11。该`RequestRateLimiter` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-requestratelimiter-gatewayfilter-factory)

该`RequestRateLimiter` `GatewayFilter`工厂采用的是`RateLimiter`实施以确定当前请求被允许继续进行。如果不是，`HTTP 429 - Too Many Requests`则返回状态（默认）。

该过滤器采用一个可选`keyResolver`参数和特定于速率限制器的参数（本节后面将介绍）。

`keyResolver`是实现`KeyResolver`接口的bean 。在配置中，使用SpEL按名称引用bean。 `#{@myKeyResolver}`是一个SpEL表达式，它引用名为的bean `myKeyResolver`。以下清单显示了该`KeyResolver`接口：

例子34. KeyResolver.java

```java
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

该`KeyResolver`接口允许可插拔策略派生用于限制请求的密钥。在未来的里程碑版本中，将有一些`KeyResolver`实现。

的默认实现`KeyResolver`是`PrincipalNameKeyResolver`，`Principal`从`ServerWebExchange`和调用检索`Principal.getName()`。

默认情况下，如果`KeyResolver`找不到密钥，则拒绝请求。您可以通过设置`spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key`（`true`或`false`）和`spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code`属性来调整此行为。

|      | 在`RequestRateLimiter`不与“快捷方式”符号配置。下面的示例*无效*：例子35. application.properties`＃无效的快捷方式配置 spring.cloud.gateway.routes [0] .filters [0] = RequestRateLimiter = 2，2，＃{@ userkeyresolver}` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### [6.11.1。Redis`RateLimiter`](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-redis-ratelimiter)

Redis实现基于[Stripe](https://stripe.com/blog/rate-limiters)所做的工作。它需要使用`spring-boot-starter-data-redis-reactive`Spring Boot启动器。

使用的算法是[令牌桶算法](https://en.wikipedia.org/wiki/Token_bucket)。

该`redis-rate-limiter.replenishRate`属性是希望用户每秒允许多少个请求，而没有任何丢弃的请求。这是令牌桶被填充的速率。

该`redis-rate-limiter.burstCapacity`属性是允许用户在一秒钟内执行的最大请求数。这是令牌桶可以容纳的令牌数。将此值设置为零将阻止所有请求。

该`redis-rate-limiter.requestedTokens`属性是一个请求要花费多少令牌。这是每个请求从存储桶中提取的令牌数，默认为`1`。

通过在`replenishRate`和中设置相同的值，可以达到稳定的速率`burstCapacity`。设置`burstCapacity`大于可以允许临时爆发`replenishRate`。在这种情况下，`replenishRate`由于两次连续的突发会导致请求丢弃（`HTTP 429 - Too Many Requests`），因此需要在两次突发之间允许有一定的时间限制（根据）。以下清单配置了一个`redis-rate-limiter`：

波纹管速率限制`1 request/s`功能通过设置完成`replenishRate`的请求的希望数，`requestedTokens`在几秒钟内的时间跨度`burstCapacity`来的产品`replenishRate`和`requestedTokens`结构，如设定`replenishRate=1`，`requestedTokens=60`以及`burstCapacity=60`将会导致的限制`1 request/min`。

例子36. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
            redis-rate-limiter.requestedTokens: 1
```

以下示例在Java中配置KeyResolver：

例子37. Config.java

```java
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了每个用户10的请求速率限制。允许20个突发，但是在下一秒中，只有10个请求可用。这`KeyResolver`是一个简单的获取`user`请求参数的参数（请注意，不建议在生产环境中使用该参数）。

您还可以将速率限制器定义为实现`RateLimiter`接口的Bean 。在配置中，可以使用SpEL按名称引用Bean。 `#{@myRateLimiter}`是一个SpEL表达式，它引用一个名为的bean `myRateLimiter`。以下清单定义了一个使用`KeyResolver`前面清单中定义的速率限制器：

例子38. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```

### [6.12。该`RedirectTo` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-redirectto-gatewayfilter-factory)

该`RedirectTo` `GatewayFilter`工厂有两个参数，`status`和`url`。该`status`参数应该是300系列重定向HTTP代码，例如301。该`url`参数应该是有效的URL。这是`Location`标题的值。对于相对重定向，您应该将其`uri: no://op`用作路由定义的uri。以下清单配置了一个`RedirectTo` `GatewayFilter`：

例子39. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - RedirectTo=302, https://acme.org
```

这将发送带有`Location:https://acme.org`标头的状态302以执行重定向。

### [6.13。该`RemoveRequestHeader`GatewayFilter厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-removerequestheader-gatewayfilter-factory)

该`RemoveRequestHeader` `GatewayFilter`工厂需要一个`name`参数。它是要删除的标题的名称。以下清单配置了一个`RemoveRequestHeader` `GatewayFilter`：

例子40. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```

这将删除`X-Request-Foo`标题，然后再将其发送到下游。

### [6.14。`RemoveResponseHeader` `GatewayFilter`厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#removeresponseheader-gatewayfilter-factory)

该`RemoveResponseHeader` `GatewayFilter`工厂需要一个`name`参数。它是要删除的标题的名称。以下清单配置了一个`RemoveResponseHeader` `GatewayFilter`：

例子41. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removeresponseheader_route
        uri: https://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```

这将从`X-Response-Foo`响应中删除标头，然后将其返回到网关客户端。

要删除任何类型的敏感标头，应为可能要执行此操作的所有路由配置此过滤器。此外，您可以使用一次配置此过滤器，`spring.cloud.gateway.default-filters`并将其应用于所有路由。

### [6.15。该`RemoveRequestParameter` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-removerequestparameter-gatewayfilter-factory)

该`RemoveRequestParameter` `GatewayFilter`工厂需要一个`name`参数。它是要删除的查询参数的名称。以下示例配置了`RemoveRequestParameter` `GatewayFilter`：

例子42. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestparameter_route
        uri: https://example.org
        filters:
        - RemoveRequestParameter=red
```

这将删除`red`参数，然后再将其发送到下游。

### [6.16。该`RewritePath` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-rewritepath-gatewayfilter-factory)

该`RewritePath` `GatewayFilter`工厂采用的路径`regexp`参数和`replacement`参数。这使用Java正则表达式提供了一种灵活的方式来重写请求路径。以下清单配置了一个`RewritePath` `GatewayFilter`：

例子43. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://example.org
        predicates:
        - Path=/red/**
        filters:
        - RewritePath=/red(?<segment>/?.*), $\{segment}
```

对于的请求路径`/red/blue`，这会将路径设置为`/blue`发出下游请求之前的路径。请注意，由于YAML规范，`$`应将替换`$\`为。

### [6.17。`RewriteLocationResponseHeader` `GatewayFilter`厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#rewritelocationresponseheader-gatewayfilter-factory)

该`RewriteLocationResponseHeader` `GatewayFilter`工厂修改的值`Location`响应头，通常摆脱于后端的具体细节。这需要`stripVersionMode`，`locationHeaderName`，`hostValue`，和`protocolsRegex`参数。以下清单配置了一个`RewriteLocationResponseHeader` `GatewayFilter`：

例子44. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritelocationresponseheader_route
        uri: http://example.org
        filters:
        - RewriteLocationResponseHeader=AS_IN_REQUEST, Location, ,
```

例如，对于的请求，的响应标头值将重写为。`POST api.example.com/some/object/name``Location``object-service.prod.example.net/v2/some/object/id``api.example.com/some/object/id`

该`stripVersionMode`参数具有以下可能的值：`NEVER_STRIP`，`AS_IN_REQUEST`（默认）和`ALWAYS_STRIP`。

- `NEVER_STRIP`：即使原始请求路径不包含任何版本，也不会剥离该版本。
- `AS_IN_REQUEST` 仅当原始请求路径不包含任何版本时，才剥离该版本。
- `ALWAYS_STRIP` 即使原始请求路径包含版本，也会始终剥离该版本。

的`hostValue`参数，如果提供，则使用替换`host:port`的响应的部分`Location`标头。如果未提供，`Host`则使用请求标头的值。

该`protocolsRegex`参数必须是`String`与协议名称匹配的有效regex 。如果不匹配，则过滤器不执行任何操作。默认值为`http|https|ftp|ftps`。

### [6.18。该`RewriteResponseHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-rewriteresponseheader-gatewayfilter-factory)

该`RewriteResponseHeader` `GatewayFilter`工厂需要`name`，`regexp`和`replacement`参数。它使用Java正则表达式以灵活的方式重写响应标头值。以下示例配置了`RewriteResponseHeader` `GatewayFilter`：

例子45. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
```

对于标头值为`/42?user=ford&password=omg!what&flag=true`，`/42?user=ford&password=***&flag=true`在发出下游请求后将其设置为。由于YAML规范，您必须使用`$\`表示`$`。

### [6.19。该`SaveSession` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-savesession-gatewayfilter-factory)

*在*向下游转发呼叫*之前*，`SaveSession` `GatewayFilter`工厂会强制执行`WebSession::save`操作。当将诸如[Spring Session之](https://projects.spring.io/spring-session/)类的东西与惰性数据存储一起使用时，这特别有用，您需要确保在进行转发呼叫之前已保存了会话状态。以下示例配置了：`SaveSession` `GatewayFilter`

例子46. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: https://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
```

如果您将[Spring Security](https://projects.spring.io/spring-security/)与Spring Session集成在一起，并希望确保安全性详细信息已转发到远程进程，那么这一点至关重要。

### [6.20。该`SecureHeaders` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-secureheaders-gatewayfilter-factory)

根据[此博客文章中](https://blog.appcanary.com/2017/http-security-headers.html)`SecureHeaders` `GatewayFilter`的建议，工厂为响应添加了许多标题。

添加了以下标头（以其默认值显示）：

- `X-Xss-Protection:1 (mode=block`）
- `Strict-Transport-Security (max-age=631138519`）
- `X-Frame-Options (DENY)`
- `X-Content-Type-Options (nosniff)`
- `Referrer-Policy (no-referrer)`
- `Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline)'`
- `X-Download-Options (noopen)`
- `X-Permitted-Cross-Domain-Policies (none)`

要更改默认值，请在`spring.cloud.gateway.filter.secure-headers`名称空间中设置适当的属性。可以使用以下属性：

- `xss-protection-header`
- `strict-transport-security`
- `x-frame-options`
- `x-content-type-options`
- `referrer-policy`
- `content-security-policy`
- `x-download-options`
- `x-permitted-cross-domain-policies`

要禁用默认值，请`spring.cloud.gateway.filter.secure-headers.disable`使用逗号分隔的值设置属性。以下示例显示了如何执行此操作：

```
spring.cloud.gateway.filter.secure-headers.disable=x-frame-options,strict-transport-security
```

|      | 必须使用安全标头的小写全名来禁用它。 |
| ---- | ------------------------------------ |
|      |                                      |

### [6.21。该`SetPath` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-setpath-gatewayfilter-factory)

该`SetPath` `GatewayFilter`工厂采用的路径`template`参数。通过允许路径的模板段，它提供了一种操作请求路径的简单方法。这使用了Spring Framework中的URI模板。允许多个匹配段。以下示例配置了`SetPath` `GatewayFilter`：

例子47. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

对于的请求路径`/red/blue`，这会将路径设置为`/blue`发出下游请求之前的路径。

### [6.22。该`SetRequestHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-setrequestheader-gatewayfilter-factory)

该`SetRequestHeader` `GatewayFilter`工厂采用`name`和`value`参数。以下清单配置了一个`SetRequestHeader` `GatewayFilter`：

例子48. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

这`GatewayFilter`将用给定名称替换（而不是添加）所有标头。因此，如果下游服务器以响应`X-Request-Red:1234`，则将替换为`X-Request-Red:Blue`，这是下游服务将收到的内容。

`SetRequestHeader`知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时扩展。以下示例配置了`SetRequestHeader` `GatewayFilter`使用变量的：

例子49. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetRequestHeader=foo, bar-{segment}
```

### [6.23。该`SetResponseHeader` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-setresponseheader-gatewayfilter-factory)

该`SetResponseHeader` `GatewayFilter`工厂采用`name`和`value`参数。以下清单配置了一个`SetResponseHeader` `GatewayFilter`：

例子50. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        filters:
        - SetResponseHeader=X-Response-Red, Blue
```

该GatewayFilter用给定名称替换（而不是添加）所有标头。因此，如果下游服务器以响应，则将`X-Response-Red:1234`替换为`X-Response-Red:Blue`，这是网关客户端将收到的内容。

`SetResponseHeader`知道用于匹配路径或主机的URI变量。URI变量可用于该值，并将在运行时扩展。以下示例配置了`SetResponseHeader` `GatewayFilter`使用变量的：

例子51. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetResponseHeader=foo, bar-{segment}
```

### [6.24。该`SetStatus` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-setstatus-gatewayfilter-factory)

该`SetStatus` `GatewayFilter`工厂采用单个参数，`status`。它必须是有效的Spring `HttpStatus`。它可以是整数值`404`或枚举的字符串表示形式：`NOT_FOUND`。以下清单配置了一个`SetStatus` `GatewayFilter`：

例子52. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusstring_route
        uri: https://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

无论哪种情况，响应的HTTP状态都设置为401。

您可以配置，`SetStatus` `GatewayFilter`以在响应的标头中从代理请求返回原始HTTP状态代码。如果将头配置为以下属性，则会将其添加到响应中：

例子53. application.yml

```yaml
spring:
  cloud:
    gateway:
      set-status:
        original-status-header-name: original-http-status
```

### [6.25。该`StripPrefix` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-stripprefix-gatewayfilter-factory)

该`StripPrefix` `GatewayFilter`工厂有一个参数，`parts`。该`parts`参数指示在向下游发送请求之前，要从请求中剥离的路径中的零件数。以下清单配置了一个`StripPrefix` `GatewayFilter`：

例子54. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

通过网关`/name/blue/red`发出请求时，发出的请求`nameservice`看起来像`nameservice/red`。

### [6.26。重试`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-retry-gatewayfilter-factory)

该`Retry` `GatewayFilter`工厂支持以下参数：

- `retries`：应尝试的重试次数。
- `statuses`：应重试的HTTP状态代码，使用表示`org.springframework.http.HttpStatus`。
- `methods`：应该重试的HTTP方法，以表示`org.springframework.http.HttpMethod`。
- `series`：要重试的一系列状态代码，使用表示`org.springframework.http.HttpStatus.Series`。
- `exceptions`：应重试的引发异常的列表。
- `backoff`：为重试配置的指数补偿。重试在退避间隔，即`firstBackoff * (factor ^ n)`，之后执行`n`。如果`maxBackoff`已配置，则应用的最大退避限制为`maxBackoff`。如果`basedOnPreviousValue`为true，则使用计算退避`prevBackoff * factor`。

`Retry`如果启用了以下默认过滤器配置：

- `retries`：3次
- `series`：5XX系列
- `methods`：GET方法
- `exceptions`：`IOException`和`TimeoutException`
- `backoff`：禁用

以下清单配置了Retry `GatewayFilter`：

例子55. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: http://localhost:8080/flakey
        predicates:
        - Host=*.retry.com
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
            methods: GET,POST
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms
              factor: 2
              basedOnPreviousValue: false
```

|      | 当使用带有`forward:`前缀URL的重试过滤器时，应仔细编写目标端点，以便在发生错误的情况下，它不会做任何可能导致响应发送到客户端并提交的操作。例如，如果目标端点是带注释的控制器，则目标控制器方法不应返回`ResponseEntity`错误状态代码。相反，它应该抛出一个`Exception`错误或发出一个错误信号（例如，通过`Mono.error(ex)`返回值），该错误可以配置为重试过滤器通过重试进行处理。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 当将重试过滤器与任何具有主体的HTTP方法一起使用时，主体将被缓存，并且网关将受到内存的限制。正文被缓存在由定义的请求属性中`ServerWebExchangeUtils.CACHED_REQUEST_BODY_ATTR`。对象的类型是`org.springframework.core.io.buffer.DataBuffer`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [6.27。该`RequestSize` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-requestsize-gatewayfilter-factory)

当请求大小大于允许的限制时，`RequestSize` `GatewayFilter`工厂可以限制请求到达下游服务。过滤器接受一个`maxSize`参数。的`maxSize is a `DataSize`类型，所以值可以被定义为一个数字，后跟一个可选的`DataUnit`后缀，例如“KB”或“MB”。字节的默认值为“ B”。它是请求的允许大小限制，以字节为单位。以下清单配置了一个`RequestSize` `GatewayFilter`：

例子56. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
        uri: http://localhost:8080/upload
        predicates:
        - Path=/upload
        filters:
        - name: RequestSize
          args:
            maxSize: 5000000
```

在`RequestSize` `GatewayFilter`工厂设置响应状态作为`413 Payload Too Large`与另外的报头`errorMessage`时，请求被由于尺寸拒绝。以下示例显示了这样的内容`errorMessage`：

```
errorMessage` : `Request size is larger than permissible limit. Request size is 6.0 MB where permissible limit is 5.0 MB
```

|      | 如果未在路由定义中作为过滤器参数提供，则默认请求大小将设置为5 MB。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [6.28。该`SetRequestHost` `GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-setrequesthost-gatewayfilter-factory)

在某些情况下，可能需要覆盖主机头。在这种情况下，`SetRequestHost` `GatewayFilter`工厂可以用指定的值替换现有的主机头。过滤器接受一个`host`参数。以下清单配置了一个`SetRequestHost` `GatewayFilter`：

例子57. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: set_request_host_header_route
        uri: http://localhost:8080/headers
        predicates:
        - Path=/headers
        filters:
        - name: SetRequestHost
          args:
            host: example.org
```

该`SetRequestHost` `GatewayFilter`工厂替换主机头的值`example.org`。

### [6.29。修改请求正文`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#modify-a-request-body-gatewayfilter-factory)

您可以使用`ModifyRequestBody`过滤器过滤器在网关向下游发送请求主体之前对其进行修改。

|      | 只能使用Java DSL来配置此过滤器。 |
| ---- | -------------------------------- |
|      |                                  |

以下清单显示了如何修改请求正文`GatewayFilter`：

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}

static class Hello {
    String message;

    public Hello() { }

    public Hello(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

### [6.30。修改响应主体`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#modify-a-response-body-gatewayfilter-factory)

您可以使用`ModifyResponseBody`过滤器在将响应正文发送回客户端之前对其进行修改。

|      | 只能使用Java DSL来配置此过滤器。 |
| ---- | -------------------------------- |
|      |                                  |

以下清单显示了如何修改响应正文`GatewayFilter`：

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri))
        .build();
}
```

### [6.31。默认过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#default-filters)

要添加过滤器并将其应用于所有路由，可以使用`spring.cloud.gateway.default-filters`。此属性采用过滤器列表。以下清单定义了一组默认过滤器：

例子58. application.yml

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```

## [7.全局过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#global-filters)

该`GlobalFilter`接口具有相同的签名`GatewayFilter`。这些是特殊过滤器，有条件地应用于所有路由。

|      | 此接口及其用法可能会在将来的里程碑版本中更改。 |
| ---- | ---------------------------------------------- |
|      |                                                |

### [7.1。组合的全局过滤器和`GatewayFilter`订购](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-combined-global-filter-and-gatewayfilter-ordering)

当请求与路由匹配时，过滤Web处理程序会将的所有实例`GlobalFilter`和所有特定`GatewayFilter`于路由的实例添加到过滤器链中。该组合的过滤器链按`org.springframework.core.Ordered`接口排序，您可以通过实现该`getOrder()`方法进行设置。

由于Spring Cloud Gateway区分了执行过滤器逻辑的“前”阶段和“后”阶段（请参见[工作原理](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-how-it-works)），因此优先级最高的过滤器是“前”阶段的第一个，而“后”阶段的最后一个-相。

以下清单配置了一个过滤器链：

例子59. ExampleConfiguration.java

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

### [7.2。正向路由过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#forward-routing-filter)

将`ForwardRoutingFilter`在交换属性查找一个URI `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`。如果该网址具有`forward`方案（例如`forward:///localendpoint`），它将使用Spring`DispatcherHandler`处理请求。请求URL的路径部分被转发URL中的路径覆盖。未经修改的原始URL将附加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性中的列表。

### [7.3。该`LoadBalancerClient`过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-loadbalancerclient-filter)

将`LoadBalancerClientFilter`在交换属性查找一个URI命名`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`。如果URL的方案为`lb`（例如`lb://myservice`），它将使用Spring Cloud`LoadBalancerClient`将名称（`myservice`在这种情况下）解析为实际的主机和端口，并在同一属性中替换URI。未经修改的原始URL将附加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性中的列表。过滤器还会在`ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`属性中查找是否相等`lb`。如果是这样，则适用相同的规则。以下清单配置了一个`LoadBalancerClientFilter`：

例子60. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

|      | 默认情况下，如果在中找不到服务实例`LoadBalancer`，`503`则返回a。您可以`404`通过设置将网关配置为返回`spring.cloud.gateway.loadbalancer.use404=true`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 从覆盖返回 的`isSecure`值`ServiceInstance`将`LoadBalancer`覆盖对网关的请求中指定的方案。例如，如果请求通过进入网关，`HTTPS` 但`ServiceInstance`指示该请求不安全，则通过发出下游请求 `HTTP`。相反的情况也可以适用。但是，如果`GATEWAY_SCHEME_PREFIX_ATTR`在“网关”配置中为路由指定了该前缀，则将删除前缀，并且路由URL产生的方案将覆盖该`ServiceInstance`配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `LoadBalancerClientFilter``LoadBalancerClient`在引擎盖下使用阻挡色带。我们建议您[`ReactiveLoadBalancerClientFilter`改用](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#reactive-loadbalancer-client-filter)。您可以通过设置的值切换到它`spring.cloud.loadbalancer.ribbon.enabled`来`false`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [7.4。的`ReactiveLoadBalancerClientFilter`](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#reactive-loadbalancer-client-filter)

将`ReactiveLoadBalancerClientFilter`在交换属性查找一个URI命名`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`。如果URL有一个`lb`方案（例如`lb://myservice`），它将使用Spring Cloud`ReactorLoadBalancer`将名称（`myservice`在本示例中）解析为实际的主机和端口，并在同一属性中替换URI。未经修改的原始URL将附加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性中的列表。过滤器还会在`ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`属性中查找是否相等`lb`。如果是这样，则适用相同的规则。以下清单配置了一个`ReactiveLoadBalancerClientFilter`：

例子61. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

|      | 缺省情况下，当找不到服务实例时`ReactorLoadBalancer`，将`503`返回a。您可以`404`通过设置将网关配置为返回a `spring.cloud.gateway.loadbalancer.use404=true`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 从覆盖返回 的`isSecure`值`ServiceInstance`将`ReactiveLoadBalancerClientFilter`覆盖对网关的请求中指定的方案。例如，如果请求通过进入网关，`HTTPS`但`ServiceInstance`指示该请求不安全，则通过发出下游请求`HTTP`。相反的情况也可以适用。但是，如果`GATEWAY_SCHEME_PREFIX_ATTR`在“网关”配置中为路由指定了该前缀，则将删除前缀，并且路由URL产生的方案将覆盖该`ServiceInstance`配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [7.5。网络路由过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-netty-routing-filter)

如果位于`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`交换属性中的URL具有`http`或`https`方案，则将运行Netty路由筛选器。它使用Netty`HttpClient`发出下游代理请求。响应将放入`ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`exchange属性中，以供以后的过滤器使用。（也有一个实验`WebClientHttpRoutingFilter`，执行相同的功能，但不需要Netty。）

### [7.6。净写响应过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-netty-write-response-filter)

的`NettyWriteResponseFilter`，如果有一个运行的Netty`HttpClientResponse`在`ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`交换属性。它在所有其他筛选器完成后运行，并将代理响应写回到网关客户端响应。（也有一个实验`WebClientWriteResponseFilter`，执行相同的功能，但不需要Netty。）

### [7.7。该`RouteToRequestUrl`过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-routetorequesturl-filter)

如果交换属性中有一个`Route`对象`ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR`，则`RouteToRequestUrlFilter`运行。它基于请求URI创建一个新的URI，但使用`Route`对象的URI属性进行更新。新的URI放置在`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`exchange属性中。

如果URI具有方案前缀（例如）`lb:ws://serviceid`，则将`lb`方案从URI中剥离，并放入中以`ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`供稍后在过滤器链中使用。

### [7.8。Websocket路由过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-websocket-routing-filter)

如果位于`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`exchange属性中的URL具有`ws`或`wss`方案，则将运行websocket路由筛选器。它使用Spring WebSocket基础结构向下游转发websocket请求。

你可以用前缀的URI负载平衡的WebSockets `lb`，如`lb:ws://serviceid`。

|      | 如果将[SockJS](https://github.com/sockjs)用作常规HTTP的后备，则应配置常规HTTP路由以及websocket路由。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下面的清单配置了一个websocket路由过滤器：

例子62. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      # SockJS route
      - id: websocket_sockjs_route
        uri: http://localhost:3001
        predicates:
        - Path=/websocket/info/**
      # Normal Websocket route
      - id: websocket_route
        uri: ws://localhost:3001
        predicates:
        - Path=/websocket/**
```

### [7.9。网关指标过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-gateway-metrics-filter)

要启用网关指标，请添加spring-boot-starter-actuator作为项目依赖项。然后，默认情况下，只要属性`spring.cloud.gateway.metrics.enabled`未设置为，网关度量过滤器就会运行`false`。该过滤器添加了一个`gateway.requests`带有以下标记的计时器指标：

- `routeId`：路由ID。
- `routeUri`：API路由到的URI。
- `outcome`：结果，按[HttpStatus.Series](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpStatus.Series.html)分类。
- `status`：请求的HTTP状态返回给客户端。
- `httpStatusCode`：请求的HTTP状态返回给客户端。
- `httpMethod`：用于请求的HTTP方法。

然后可以从这些指标中[删除](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/images/gateway-grafana-dashboard.jpeg)这些指标，`/actuator/metrics/gateway.requests`并且可以轻松地将这些指标与Prometheus集成以创建[Grafana](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/images/gateway-grafana-dashboard.jpeg) [仪表板](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/gateway-grafana-dashboard.json)。

|      | 要启用Prometheus端点，请添加`micrometer-registry-prometheus`为项目依赖项。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [7.10。将交换标记为已路由](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#marking-an-exchange-as-routed)

网关对a进行路由之后`ServerWebExchange`，通过添加`gatewayAlreadyRouted` 交换属性将交换标记为“已路由” 。将请求标记为已路由后，其他路由筛选器将不会再次路由请求，实质上会跳过该筛选器。您可以使用多种便利的方法将交换标记为已路由或检查交换是否已路由。

- `ServerWebExchangeUtils.isAlreadyRouted`接收一个`ServerWebExchange`对象并检查它是否已被“路由”。
- `ServerWebExchangeUtils.setAlreadyRouted`接收一个`ServerWebExchange`对象并将其标记为“已路由”。

## [8. HttpHeadersFilters](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#httpheadersfilters)

HttpHeadersFilters会先应用于请求，然后再向下游发送请求，例如在中`NettyRoutingFilter`。

### [8.1。转发的标题过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#forwarded-headers-filter)

该`Forwarded`头过滤器创建一个`Forwarded`头发送到下游服务。它将`Host`当前请求的标头，方案和端口添加到任何现有的`Forwarded`标头中。

### [8.2。RemoveHopByHop标头过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#removehopbyhop-headers-filter)

该`RemoveHopByHop`头过滤去除转发的请求头。被删除的头的默认列表来自[IETF](https://tools.ietf.org/html/draft-ietf-httpbis-p1-messaging-14#section-7.1.3)。

默认删除的标题是：

- 连接
- 活着
- 代理验证
- 代理授权
- TE
- 预告片
- 传输编码
- 升级

要更改此设置，请将`spring.cloud.gateway.filter.remove-hop-by-hop.headers`属性设置为要删除的标题名称列表。

### [8.3。XForwarded标头过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#xforwarded-headers-filter)

该`XForwarded`头过滤器创建一个不同的`X-Forwarded-*`标题发送到下游服务。它使用`Host`当前请求的标头，方案，端口和路径来创建各种标头。

可以通过以下布尔属性（默认为true）控制单个标题的创建：

- `spring.cloud.gateway.x-forwarded.for-enabled`
- `spring.cloud.gateway.x-forwarded.host-enabled`
- `spring.cloud.gateway.x-forwarded.port-enabled`
- `spring.cloud.gateway.x-forwarded.proto-enabled`
- `spring.cloud.gateway.x-forwarded.prefix-enabled`

可以通过以下布尔属性（默认为true）控制追加多个标头：

- `spring.cloud.gateway.x-forwarded.for-append`
- `spring.cloud.gateway.x-forwarded.host-append`
- `spring.cloud.gateway.x-forwarded.port-append`
- `spring.cloud.gateway.x-forwarded.proto-append`
- `spring.cloud.gateway.x-forwarded.prefix-append`

## [9. TLS和SSL](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#tls-and-ssl)

遵循通常的Spring服务器配置，网关可以侦听HTTPS上的请求。以下示例显示了如何执行此操作：

例子63. application.yml

```yaml
server:
  ssl:
    enabled: true
    key-alias: scg
    key-store-password: scg1234
    key-store: classpath:scg-keystore.p12
    key-store-type: PKCS12
```

您可以将网关路由路由到HTTP和HTTPS后端。如果要路由到HTTPS后端，则可以使用以下配置将网关配置为信任所有下游证书：

例子64. application.yml

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          useInsecureTrustManager: true
```

使用不安全的信任管理器不适用于生产。对于生产部署，可以使用以下配置为网关配置一组可以信任的已知证书：

例子65. application.yml

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          trustedX509Certificates:
          - cert1.pem
          - cert2.pem
```

如果未为Spring Cloud Gateway提供受信任的证书，则使用默认的信任库（您可以通过设置`javax.net.ssl.trustStore`system属性来覆盖它）。

### [9.1。TLS握手](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#tls-handshake)

网关维护一个客户端池，该客户端池用于路由到后端。通过HTTPS进行通信时，客户端会启动TLS握手。许多超时与此握手相关联。您可以配置这些超时，可以如下配置（显示默认值）：

例子66. application.yml

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          handshake-timeout-millis: 10000
          close-notify-flush-timeout-millis: 3000
          close-notify-read-timeout-millis: 0
```

## [10.配置](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#configuration)

Spring Cloud Gateway的配置由一系列`RouteDefinitionLocator`实例驱动。以下清单显示了`RouteDefinitionLocator`接口的定义：

例子67. RouteDefinitionLocator.java

```java
public interface RouteDefinitionLocator {
    Flux<RouteDefinition> getRouteDefinitions();
}
```

默认情况下，`PropertiesRouteDefinitionLocator`使用Spring Boot的`@ConfigurationProperties`机制来加载属性。

较早的配置示例均使用快捷方式符号，该快捷方式符号使用位置参数而不是命名参数。以下两个示例是等效的：

例子68. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatus_route
        uri: https://example.org
        filters:
        - name: SetStatus
          args:
            status: 401
      - id: setstatusshortcut_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

对于网关的某些用法，属性是足够的，但是某些生产用例会受益于从外部源（例如数据库）加载配置。未来的里程碑版本将具有`RouteDefinitionLocator`基于Spring数据存储库的实现，例如Redis，MongoDB和Cassandra。

## [11.路由元数据配置](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#route-metadata-configuration)

您可以使用元数据为每个路由配置其他参数，如下所示：

例子69. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: route_with_metadata
        uri: https://example.org
        metadata:
          optionName: "OptionValue"
          compositeObject:
            name: "value"
          iAmNumber: 1
```

您可以从交易所获取所有元数据属性，如下所示：

```
Route route = exchange.getAttribute(GATEWAY_ROUTE_ATTR);
// get all metadata properties
route.getMetadata();
// get a single metadata property
route.getMetadata(someKey);
```

## [12. Http超时配置](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#http-timeouts-configuration)

可以为所有路由配置Http超时（响应和连接），并为每个特定路由覆盖Http超时。

### [12.1。全局超时](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#global-timeouts)

要配置全局http超时：
`connect-timeout`必须以毫秒为单位指定。
`response-timeout`必须指定为java.time.Duration

全局http超时示例

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

### [12.2。每个路由超时](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#per-route-timeouts)

要配置每个路由超时：
`connect-timeout`必须以毫秒为单位指定。
`response-timeout`必须以毫秒为单位指定。

通过配置每个路由的HTTP超时

```yaml
      - id: per_route_timeouts
        uri: https://example.org
        predicates:
          - name: Path
            args:
              pattern: /delay/{timeout}
        metadata:
          response-timeout: 200
          connect-timeout: 200
```

使用Java DSL的每个路由超时配置

```java
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.CONNECT_TIMEOUT_ATTR;
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.RESPONSE_TIMEOUT_ATTR;

      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder routeBuilder){
         return routeBuilder.routes()
               .route("test1", r -> {
                  return r.host("*.somehost.org").and().path("/somepath")
                        .filters(f -> f.addRequestHeader("header1", "header-value-1"))
                        .uri("http://someuri")
                        .metadata(RESPONSE_TIMEOUT_ATTR, 200)
                        .metadata(CONNECT_TIMEOUT_ATTR, 200);
               })
               .build();
      }
```

### [12.3。流利的Java Routes API](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#fluent-java-routes-api)

为了在Java中进行简单的配置，该`RouteLocatorBuilder`bean包含了一个流畅的API。以下清单显示了它的工作方式：

例子70. GatewaySampleApplication.java

```java
// static imports from GatewayFilters and RoutePredicates
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
    return builder.routes()
            .route(r -> r.host("**.abc.org").and().path("/image/png")
                .filters(f ->
                        f.addResponseHeader("X-TestHeader", "foobar"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.path("/image/webp")
                .filters(f ->
                        f.addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://httpbin.org:80")
                .metadata("key", "value")
            )
            .route(r -> r.order(-1)
                .host("**.throttle.org").and().path("/get")
                .filters(f -> f.filter(throttle.apply(1,
                        1,
                        10,
                        TimeUnit.SECONDS)))
                .uri("http://httpbin.org:80")
                .metadata("key", "value")
            )
            .build();
}
```

此样式还允许更多自定义谓词断言。`RouteDefinitionLocator`bean定义的谓词使用逻辑组合`and`。通过使用流利的Java API，你可以使用`and()`，`or()`以及`negate()`对运营`Predicate`类。

### [12.4。该`DiscoveryClient`路由定义定位器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#the-discoveryclient-route-definition-locator)

您可以将网关配置为基于在`DiscoveryClient`兼容服务注册表中注册的服务来创建路由。

要启用此功能，请设置`spring.cloud.gateway.discovery.locator.enabled=true`并确保`DiscoveryClient`在类路径上启用了某个实现（例如Netflix Eureka，Consul或Zookeeper）。

#### [12.4.1。配置`DiscoveryClient`路由的谓词和过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#configuring-predicates-and-filters-for-discoveryclient-routes)

默认情况下，网关为使用所创建的路由定义单个谓词和过滤器`DiscoveryClient`。

默认谓词是使用模式定义的路径谓词`/serviceId/**`，其中`serviceId`是来自的服务ID `DiscoveryClient`。

默认的过滤器是带有正则表达式`/serviceId/(?<remaining>.*)`和替换的重写路径过滤器`/${remaining}`。这会在将请求发送到下游之前从路径中剥离服务ID。

如果要自定义`DiscoveryClient`路线使用的谓词或过滤器，请设置`spring.cloud.gateway.discovery.locator.predicates[x]`和`spring.cloud.gateway.discovery.locator.filters[y]`。这样做时，如果要保留该功能，则需要确保包括前面显示的默认谓词和过滤器。下面的示例显示其外观：

例子71. application.properties

```
spring.cloud.gateway.discovery.locator.predicates [0]。名称：路径
spring.cloud.gateway.discovery.locator.predicates [0] .args [pattern]：“'/'+ serviceId +'/ **'” 
spring.cloud.gateway.discovery.locator.predicates [1]。名称：主机
spring.cloud.gateway.discovery.locator.predicates [1] .args [pattern]：“'**。foo.com'” 
spring。 cloud.gateway.discovery.locator.filters [0]。名称：Hystrix 
spring.cloud.gateway.discovery.locator.filters [0] .args [name]：serviceId 
spring.cloud.gateway.discovery.locator.filters [1 ] .name：RewritePath 
spring.cloud.gateway.discovery.locator.filters [1] .args [regexp]：“'/'+ serviceId +'/(??< 
remaining>. *）'” spring.cloud.gateway。 Discovery.locator.filters [1] .args [replacement]：“'/ $ {剩余}'”
```

## [13. Reactor Netty访问日志](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#reactor-netty-access-logs)

要启用Reactor Netty访问日志，请设置`-Dreactor.netty.http.server.accessLogEnabled=true`。

|      | 它必须是Java System属性，而不是Spring Boot属性。 |
| ---- | ------------------------------------------------ |
|      |                                                  |

您可以将日志记录系统配置为具有单独的访问日志文件。以下示例创建一个Logback配置：

例子72. logback.xml

```xml
    <appender name="accessLog" class="ch.qos.logback.core.FileAppender">
        <file>access_log.log</file>
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>
    <appender name="async" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="accessLog" />
    </appender>

    <logger name="reactor.netty.http.server.AccessLog" level="INFO" additivity="false">
        <appender-ref ref="async"/>
    </logger>
```

## [14. CORS配置](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#cors-configuration)

您可以配置网关以控制CORS行为。“全局” CORS配置是URL模式到[Spring Framework`CorsConfiguration`](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/cors/CorsConfiguration.html)的映射。以下示例配置了CORS：

例子73. application.yml

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

在前面的示例中，允许从`docs.spring.io`所有GET请求路径的源请求发出CORS请求。

要为某些网关路由谓词未处理的请求提供相同的CORS配置，请将`spring.cloud.gateway.globalcors.add-to-simple-url-handler-mapping`属性设置为`true`。当您尝试支持CORS预检请求并且您的路由谓词`true`由于HTTP方法为而不适用时，此功能很有用`options`。

## [15.执行器API](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#actuator-api)

该`/gateway`驱动器的端点，您可以监视和Spring的云网关应用程序进行交互。为了可远程访问，必须在应用程序属性中[通过HTTP或JMX](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints)[启用](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-endpoints-enabling-endpoints)和[公开](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints)端点。以下清单显示了如何执行此操作：

例子74. application.properties

```properties
management.endpoint.gateway.enabled=true # default value
management.endpoints.web.exposure.include=gateway
```

### [15.1。详细执行器格式](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#verbose-actuator-format)

一种新的，更详细的格式已添加到Spring Cloud Gateway。它为每个路由添加了更多详细信息，使您可以查看与每个路由关联的谓词和过滤器以及任何可用的配置。以下示例进行配置`/actuator/gateway/routes`：

```json
[
  {
    "predicate": "(Hosts: [**.addrequestheader.org] && Paths: [/headers], match trailing slash: true)",
    "route_id": "add_request_header_test",
    "filters": [
      "[[AddResponseHeader X-Response-Default-Foo = 'Default-Bar'], order = 1]",
      "[[AddRequestHeader X-Request-Foo = 'Bar'], order = 1]",
      "[[PrefixPath prefix = '/httpbin'], order = 2]"
    ],
    "uri": "lb://testservice",
    "order": 0
  }
]
```

默认情况下启用此功能。要禁用它，请设置以下属性：

例子75. application.properties

```properties
spring.cloud.gateway.actuator.verbose.enabled=false
```

`true`在将来的版本中，它将默认为。

### [15.2。检索路由过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#retrieving-route-filters)

本节详细介绍如何检索路由过滤器，包括：

- [全局过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-global-filters)
- [[网关路由过滤器\]](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-route-filters)

#### [15.2.1。全局过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-global-filters)

要检索应用于所有路由的[全局过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#global-filters)，`GET`请向发出请求`/actuator/gateway/globalfilters`。产生的响应类似于以下内容：

```
{ 
  “或g.springframework.cloud.gateway.filter.LoadBalancerClientFilter @ 77856cc5”：10100，
  “ o rg.springframework.cloud.gateway.filter.RouteToRequestUrlFilter @ 4f6fd101”：10000，
  “或g.springframework.cloud.cloud.gateway.filter .NettyWriteResponseFilter @ 32d22650“：-1，
  ” org.springframework.cloud.gateway.filter.ForwardRoutingFilter@10 6459d9“：2147483647，
  ” org.springframework.cloud.gateway.filter.NettyRoutingFilter@1fbd 5e0“：2147483647，
  ”组织。 springframework.cloud.gateway.filter.ForwardPathFilter@33a71 d23“：0，
  ”组织的pringframework.cloud.gateway.filter。AdaptCachedBodyGlobalFilter @135064ea“：2147483637，
  ” org.springframework.cloud.gateway.filter.WebsocketRoutingFilter @ 23c05889“：2147483646 
}
```

该响应包含已到位的全局筛选器的详细信息。对于每个全局过滤器，都有一个过滤器对象的字符串表示形式（例如`org.springframework.cloud.gateway.filter.LoadBalancerClientFilter@77856cc5`）和过滤器链中的相应[顺序](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-combined-global-filter-and-gatewayfilter-ordering)。}

#### [15.2.2。路线过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-route-filters)

要检索应用于路线的[`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gatewayfilter-factories)，`GET`请向发出请求`/actuator/gateway/routefilters`。产生的响应类似于以下内容：

```
{ 
  “ [ AddRequestHeaderGatewayFilterFactory @ 570ed9c configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]”：null，
  “ [[ SecureHeadersGatewayFilterFactory @ fceab5d configClass = Object]”：null，
  “ [[ SaveSessionGatewayFilterFactory @ 4449b273 configClass = Object]”：null 
}
```

该响应包含`GatewayFilter`应用于任何特定路线的工厂的详细信息。对于每个工厂，都有一个对应对象的字符串表示形式（例如`[SecureHeadersGatewayFilterFactory@fceab5d configClass = Object]`）。请注意，该`null`值是由于端点控制器的实现不完整所致，因为该值试图设置对象在过滤器链中的顺序，而该顺序不适用于`GatewayFilter`工厂对象。

### [15.3。刷新路由缓存](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#refreshing-the-route-cache)

要清除路由缓存，`POST`请向发送请求`/actuator/gateway/refresh`。该请求返回200，但没有响应正文。

### [15.4。检索网关中定义的路由](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#retrieving-the-routes-defined-in-the-gateway)

要检索网关中定义的路由，`GET`请向发出请求`/actuator/gateway/routes`。产生的响应类似于以下内容：

```
[{ 
  “ route_id”：“ first_route”，
  “ route_object”：{ 
    “ predicate”：“ org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory $$ Lambda $ 432/1736826640 @ 1e9d7e7d ”，
    “ filters”：[[ 
      OrderedGatewayFilter {delegate = org.springframework.cloud.gateway.filter.factory.PreserveHostHeaderGatewayFilterFactory $$ Lambda $ 436/674480275 @ 6631ef72 ，order = 0}“ 
    ] 
  }，
  ” order“：0 
}，
{ 
  ” route_id“：” second_route“，
  ” route_object“：{ 
    ” predicate“：” org.springframework.cloud.gateway.handler.predicate。PathRoutePredicateFactory $$ Lambda $ 432/1736826640 @ cd8d298 “，
    “过滤器”：[] 
  }，
  “订单”：0 
}]
```

该响应包含网关中定义的所有路由的详细信息。下表描述了响应的每个元素的结构（每个元素都是一条路线）：

| 路径                     | 类型 | 描述                                                         |
| :----------------------- | :--- | :----------------------------------------------------------- |
| `route_id`               | 串   | 路由ID。                                                     |
| `route_object.predicate` | 目的 | 路由谓词。                                                   |
| `route_object.filters`   | 数组 | 该[`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gatewayfilter-factories)使用的路由。 |
| `order`                  | 数   | 路线顺序。                                                   |

### [15.5。检索有关特定路线的信息](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-retrieving-information-about-a-particular-route)

要检索有关一条路线的信息，`GET`请向发出请求`/actuator/gateway/routes/{id}`（例如`/actuator/gateway/routes/first_route`）。产生的响应类似于以下内容：

```
{ 
  “ id”：“ first_route”，
  “谓词”：[{ 
    “ name”：“ Path”，
    “ args”：{“ _genkey_0”：“ / first”} 
  }}，
  “ filters”：[]，
  “ uri” ：“ https://www.uri-destination.org”，
  “订单”：0 
}]
```

下表描述了响应的结构：

| 路径         | 类型 | 描述                                                   |
| :----------- | :--- | :----------------------------------------------------- |
| `id`         | 串   | 路由ID。                                               |
| `predicates` | 数组 | 路由谓词的集合。每个项目都定义给定谓词的名称和自变量。 |
| `filters`    | 数组 | 应用于路线的过滤器集合。                               |
| `uri`        | 串   | 路由的目标URI。                                        |
| `order`      | 数   | 路线顺序。                                             |

### [15.6。创建和删除特定路线](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#creating-and-deleting-a-particular-route)

要创建路由，`POST`请`/gateway/routes/{id_route_to_create}`使用JSON主体发出请求，以指定路由的字段（请参阅[检索有关特定路由的信息](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#gateway-retrieving-information-about-a-particular-route)）。

要删除路线，`DELETE`请向发出请求`/gateway/routes/{id_route_to_delete}`。

### [15.7。回顾：所有端点的列表](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#recap-the-list-of-all-endpoints)

下表列出了Spring Cloud Gateway执行器端点（请注意，每个端点都`/actuator/gateway`作为基本路径）：

| ID              | HTTP方法 | 描述                                          |
| :-------------- | :------- | :-------------------------------------------- |
| `globalfilters` | 得到     | 显示应用于路由的全局过滤器列表。              |
| `routefilters`  | 得到     | 显示`GatewayFilter`应用于特定路线的工厂列表。 |
| `refresh`       | 开机自检 | 清除路由缓存。                                |
| `routes`        | 得到     | 显示网关中定义的路由列表。                    |
| `routes/{id}`   | 得到     | 显示有关特定路线的信息。                      |
| `routes/{id}`   | 开机自检 | 将新路由添加到网关。                          |
| `routes/{id}`   | 删除     | 从网关删除现有路由。                          |

## [16.故障排除](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#troubleshooting)

本节介绍了使用Spring Cloud Gateway时可能出现的常见问题。

### [16.1。日志级别](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#log-levels)

以下记录器可能在`DEBUG`和`TRACE`级别包含有价值的故障排除信息：

- `org.springframework.cloud.gateway`
- `org.springframework.http.server.reactive`
- `org.springframework.web.reactive`
- `org.springframework.boot.autoconfigure.web`
- `reactor.netty`
- `redisratelimiter`

### [16.2。窃听](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#wiretap)

反应堆的Netty`HttpClient`和`HttpServer`可具有窃听功能。与将`reactor.netty`日志级别设置为`DEBUG`或结合使用时`TRACE`，它将启用信息记录，例如通过电线发送和接收的标头和正文。要启用监听，请分别为和设置`spring.cloud.gateway.httpserver.wiretap=true`或。`spring.cloud.gateway.httpclient.wiretap=true``HttpServer``HttpClient`

## [17.开发人员指南](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#developer-guide)

这些是编写网关的某些自定义组件的基本指南。

### [17.1。编写自定义路线谓词工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#writing-custom-route-predicate-factories)

为了编写Route Predicate，您需要实现`RoutePredicateFactory`。有一个`AbstractRoutePredicateFactory`可以扩展的抽象类。

MyRoutePredicateFactory.java

```java
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<HeaderRoutePredicateFactory.Config> {

    public MyRoutePredicateFactory() {
        super(Config.class);
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        // grab configuration from Config object
        return exchange -> {
            //grab the request
            ServerHttpRequest request = exchange.getRequest();
            //take information from the request to see if it
            //matches configuration.
            return matches(config, request);
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

### [17.2。编写自定义GatewayFilter工厂](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#writing-custom-gatewayfilter-factories)

要编写一个`GatewayFilter`，您必须实现`GatewayFilterFactory`。您可以扩展名为的抽象类`AbstractGatewayFilterFactory`。以下示例显示了如何执行此操作：

例子76. PreGatewayFilterFactory.java

```java
public class PreGatewayFilterFactory extends AbstractGatewayFilterFactory<PreGatewayFilterFactory.Config> {

    public PreGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            //If you want to build a "pre" filter you need to manipulate the
            //request before calling chain.filter
            ServerHttpRequest.Builder builder = exchange.getRequest().mutate();
            //use builder to manipulate the request
            return chain.filter(exchange.mutate().request(builder.build()).build());
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

PostGatewayFilterFactory.java

```java
public class PostGatewayFilterFactory extends AbstractGatewayFilterFactory<PostGatewayFilterFactory.Config> {

    public PostGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                ServerHttpResponse response = exchange.getResponse();
                //Manipulate the response in some way
            }));
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

#### [17.2.1。在配置中命名自定义过滤器和引用](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#naming-custom-filters-and-references-in-configuration)

自定义过滤器类名称应以结尾`GatewayFilterFactory`。

例如，要引用`Something`配置文件中命名的过滤器，该过滤器必须位于名为的类中`SomethingGatewayFilterFactory`。

|      | 可以创建一个不带`GatewayFilterFactory`后缀的网关过滤器 ，例如`class AnotherThing`。可以`AnotherThing`在配置文件中引用此过滤器。这**不是**受支持的命名约定，并且在将来的版本中可能会删除此语法。请更新过滤器名称以使其符合要求。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [17.3。编写自定义全局过滤器](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#writing-custom-global-filters)

要编写自定义全局过滤器，必须实现`GlobalFilter`接口。这会将过滤器应用于所有请求。

以下示例说明如何分别设置全局前置和后置过滤器：

```java
@Bean
public GlobalFilter customGlobalFilter() {
    return (exchange, chain) -> exchange.getPrincipal()
        .map(Principal::getName)
        .defaultIfEmpty("Default User")
        .map(userName -> {
          //adds header to proxied request
          exchange.getRequest().mutate().header("CUSTOM-REQUEST-HEADER", userName).build();
          return exchange;
        })
        .flatMap(chain::filter);
}

@Bean
public GlobalFilter customGlobalPostFilter() {
    return (exchange, chain) -> chain.filter(exchange)
        .then(Mono.just(exchange))
        .map(serverWebExchange -> {
          //adds header to response
          serverWebExchange.getResponse().getHeaders().set("CUSTOM-RESPONSE-HEADER",
              HttpStatus.OK.equals(serverWebExchange.getResponse().getStatusCode()) ? "It worked": "It did not work");
          return serverWebExchange;
        })
        .then();
}
```

## [18.使用Spring MVC或Webflux构建一个简单的网关](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#building-a-simple-gateway-by-using-spring-mvc-or-webflux)

|      | 以下描述了替代样式的网关。先前的文档均不适用于以下内容。 |
| ---- | -------------------------------------------------------- |
|      |                                                          |

Spring Cloud Gateway提供了一个名为的实用程序对象`ProxyExchange`。您可以在常规的Spring Web处理程序中使用它作为方法参数。它通过镜像HTTP动词的方法支持基本的下游HTTP交换。使用MVC，它还支持通过该`forward()`方法转发到本地处理程序。要使用`ProxyExchange`，请在类路径中包含正确的模块（`spring-cloud-gateway-mvc`或`spring-cloud-gateway-webflux`）。

下面的MVC示例将请求代理`/test`到远程服务器的下游：

```java
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

    @Value("${remote.home}")
    private URI home;

    @GetMapping("/test")
    public ResponseEntity<?> proxy(ProxyExchange<byte[]> proxy) throws Exception {
        return proxy.uri(home.toString() + "/image/png").get();
    }

}
```

以下示例对Webflux执行相同的操作：

```java
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

    @Value("${remote.home}")
    private URI home;

    @GetMapping("/test")
    public Mono<ResponseEntity<?>> proxy(ProxyExchange<byte[]> proxy) throws Exception {
        return proxy.uri(home.toString() + "/image/png").get();
    }

}
```

`ProxyExchange`启用处理程序方法上的便捷方法可以发现和增强传入请求的URI路径。例如，您可能想要提取路径的尾随元素以将它们传递到下游：

```java
@GetMapping("/proxy/path/**")
public ResponseEntity<?> proxyPath(ProxyExchange<byte[]> proxy) throws Exception {
  String path = proxy.path("/proxy/path/");
  return proxy.uri(home.toString() + "/foos/" + path).get();
}
```

网关处理程序方法可以使用Spring MVC和Webflux的所有功能。结果，例如，您可以注入请求标头和查询参数，并且可以使用映射批注中的声明来约束传入的请求。有关`@RequestMapping`这些功能的更多详细信息，请参见Spring MVC中的文档。

您可以使用上的`header()`方法将标头添加到下游响应中`ProxyExchange`。

您还可以通过向`get()`方法（和其他方法）添加映射器来操纵响应头（以及响应中您喜欢的任何其他内容）。映射器是一个`Function`接受传入的对象`ResponseEntity`，并将其转换为传出的对象。

对于不传递到下游的“敏感”标头（默认为`cookie`和`authorization`）和“ proxy”（`x-forwarded-*`）标头提供了一流的支持。

## [19.配置属性](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/#configuration-properties)

要查看所有与Spring Cloud Gateway相关的配置属性的列表，请参阅[附录](https://docs.spring.io/spring-cloud-gateway/docs/2.2.5.RELEASE/reference/html/appendix.html)。