Spring Cloud Gateway는 Spring Cloud 그룹에 포함되는 프로젝트입니다.

이름과 같이 통신구간 내 Gateway의 역할을 수행하여, API를 효과적으로 라우팅하고 Cross-cutting Concern을 제공합니다.

## 대표 용어

Spring Cloud Gateway를 사용하는데에는 아래의 3가지를 기억해야합니다.

🎁 **Route** : Gateway의 라우팅 대상이 되는 기본 몸체의 역할을 담당합니다. ID, 대상 URI, Predicate 및 Filter의 모음으로 정의합니다.

🎁 **Predicate** : Input Type은 Spring Framework의 ServerWebExhange입니다. Predicate를 통해, 헤더나 파라미터 등 HTTP 요청으로부터 전달된 항목이 작성한 Predicate와 일치하는지 요청의 유효성을 검증합니다. Spring Cloud Gateway 공식 문서를 통해, Built-in Route Predicate Factories를 확인하고, 원하는 형태로 작성하면 됩니다.

🎁 **Filter** : Gateway로 요청이 전달되면, 들어온 요청을 다운스트림에게 전달하기 이전 또는 후에 처리할 과정을 작성할 수 있습니다. Spring Cloud Gateway 공식 문서를 통해, Built-in WebFilter Factories를 확인하고, 원하는 형태로 작성하면 됩니다.

## 동작 원리

![SpringCloudGateway_동작원리](./img/springCloudGateway_process.png)

Spring Cloud Gateway는 사용자로부터 전달된 요청이 사전 정의된 API 경로와 일치하다고 판단하면, Web Handler로 전달됩니다. Web Handler는 작성된 Filter를 통해 요청을 실행합니다. Filter 동작 이후, Proxied Service(Microservice)로 사용자의 요청이 전달됩니다.

## 사용 방법

predicate와 filter를 작성하는 방법은 **shortcut** 방식과 **fully expanded arguments** 방식이 있습니다. **shortcut** 방식의 경우, 한문장 안에 predicate 또는 filter를 모두 명시하는 방법이고, **fully expanded arguments** 방식은 요구하는 인자를 나열하여 작성하는 방식입니다.

predicate를 예시로 아래와 같이 shortcut 방식과 fully expanded arguments 방식을 표현할 수 있습니다.

```yaml
predicates:
  - Cookie=mycookie,mycookievalue

또는

predicates:
  - name: Cookie
    args:
      name: mycookie
      regexp: mycookievalue
```

## Predicate

```yaml
**spring:
	cloud:
		gateway:**
			routes:
			- id: a_route
				uri: http://localhost/a
				predicates:
				- After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

위와같이 predicate에 선언한 After Route Predicate Factory 의 경우 21년 3월 16일 한국시간 9시 이후에 전달된 요청에 한해, a_route ID를 가진 [http://localhost/a](http://localhost/a) URI로 요청이 전달됩니다.

위처럼 사전 정의된 Predicate Factory를 사용하면 Predicate를 쉽게 정의할 수 있습니다. 아래 링크에서는 Route Predicate Factory들을 나열하고 간단한 예시를 정리해두었습니다.

[🏭Route Predicate Factories](./detail/분석_RoutePredicateFactories.md)

## Filter

다음으로는 Filter입니다. Filter는 Global Filters와 GatewayFilter Factories, HttpHeadersFilters가 제공됩니다. Global Filters는 모든 요청에 대해 Global하게 적용되는 FIlter입니다. GateayFilter는 위의 Predicate와 같이, 특정 라우트에 명시하여 사용되어지는 필터이니다. HttpHeadersFilter는 HTTP 요청의 헤더를 필터링하는 Filter입니다.
[🏭Gateway Filter Factories](./detail/분석_GatewayFilterFactories.md)
[🏭Global Filters](./detail/분석_GlobalFilters.md)
[🏭Http Headers Filters](./detail/분석_HttpHeadersFilters.md)

## 🔒 암호 통신

### TLS and SSL

Spring Cloud Gateway로 들어오는 HTTPS요청을 에 대해 수신할 수 있도록 설정할 수 있습니다.

```yaml
server:
  ssl:
enabled: true
    key-alias: scg
    key-store-password: scg1234
    key-store: classpath:scg-keystore.p12
    key-store-type: PKCS12   
```

(개인적으로, 암호통신을 위한 개인키를 설정 프로터피에 명시하는 것은 보안상 문제가 된다고 생각합니다만...)

사설 인증서 등 신뢰할 수 없는 기관으로부터의 인증서 셋을 허용할 수도 있습니다. 프로덕션 환경에서는 사용하지 않는 것이 좋겠죠.

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          useInsecureTrustManager: true
```

신뢰된 인증기관으로부터의 인증서는 아래와 같이 명시합니다.

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

Spring Cloud Gateway가 신뢰할 수 있는 인증서로 공급되지 않았을 경우에는 DefaultTrustStore를 기본적으로 사용합니다. 해당 설정은 javax.net.ssl.trustStore를 설정하여 재정의할 수 있습니다.

### TLS Handshake

Gateway는 Backend로 라우팅하는데 사용하는 Client Pool을 유지합니다. HTTPS를 통해 통신할 경우, TLS Handshake를 통해, 보안상 통신을 가능케 하는데, 아래의 설정 값을 통해 TLS Handshake를 다룰 수 있습니다.

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

## Spring Cloud Gateway 설정 원리

Spring Cloud Gateway는 RouteDefinitionLocator 인스턴스 컬렉션을 통해 구동됩니다.

```java
//RouteDEfinitionLocator Interface
public interface RouteDefinitionLocator {
	Flux<RouteDefinition> getRouteDefinitions();
}
```

PropertiesRouteDefinitionLocator는 @ConfigurationProperties의 매커니즘을 통해 프로퍼티를 로드합니다.

## 라우트 메타데이터 설정하기

라우트에 메타데이터를 설정하여,

```yaml
routes:
- id: route_with_metadata
  uri: https://example.org
  **metadata:
    optionName: "OptionValue"
    compositeObject:
      name: "value"
    iAmNumber: 1**
```

exchange로부터, 메타데이터 프로퍼티를 활용할 수 있습니다.

```java
Route route = exchange.getAttribute(GATEWAY_ROUTE_ATTR);
// get all metadata properties
route.getMetadata();
// get a single metadata property
route.getMetadata(someKey);
```

## 타임아웃 설정하기

HTTP 요청 및 응답에 대해 Global Timeout을 설정하려면 아래와 같이 프로퍼티로 처리할 수 있습니다.

connect-timeout은 ms(밀리세컨드) 단위로 표기해야하며, response-timeout은 java.time.Duration으로 지정해야 합니다.

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

### 😀 Per-route timeouts

라우터 단위로 타임아웃을 설정할 수 있습니다.

connect-timeout과 response-timeout 프로퍼티는 밀리세컨드 단위로 표기해야합니다.

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

JAVA DSL을 사용하여, 라우터 단위 타임아웃을 제어할 수도 있습니다.

```java
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.CONNECT_TIMEOUT_ATTR;
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.RESPONSE_TIMEOUT_ATTR;

      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder routeBuilder){
         return routeBuilder.routes()
               .route("test1", r -> {
                  return r.host("*.somehost.org").and().path("/somepath")
                        .filters(f -> f.addRequestHeader("header1", "header-value-1"))
                        **.metadata(RESPONSE_TIMEOUT_ATTR, 200)
                        .metadata(CONNECT_TIMEOUT_ATTR, 200)**
                        .uri("http://someuri");
               })
               .build();
      }
```

### 😀 Fluent Java Routes API

Fluent Java를 사용하여 Route를 작성할 수도 있습니다. Fluent Java로 라우트를 작성하면, 기존 방식보다 더 다양하게 정의할 수 있습니다.

```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
    return builder.routes()
            .route(r -> r.host("**.abc.org")**.and()**.path("/image/png")
                .filters(f ->
                        f.addResponseHeader("X-TestHeader", "foobar"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.path("/image/webp")
                .filters(f ->
                        f.addResponseHeader("X-AnotherHeader", "baz"))
                .metadata("key", "value")
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.order(-1)
                .host("**.throttle.org").and().path("/get")
                .filters(f -> f.filter(throttle.apply(1, 1, 10, TimeUnit.SECONDS)))
                .metadata("key", "value")
                .uri("http://httpbin.org:80")
            )
            .build();
}
```

조건 연산에 해당하는 메서드를 호출하여, host와 path predicate를 작성하는 등의 작업이 가능합니다.

### 😀 The DiscoveryClient Route Definition Locator

DiscoverytClient Route Definition Locator를 설정하면, DiscoveryClient 서비스 레지스트리에 등록되어있는 서비스를 기반으로 Route를 생성하도록 설정할 수 있습니다.

application.yaml에 spring.cloudgateway.discovery.locator.enabled=true를 설정하고, DiscoveryClient 구현체(Netflix Eureka, Consul, Zookeeper ...)를 작성해야합니다.

### 😀 Configuring Predicates and Filters For DscoveryClient Routes

DiscoveryClient Route는 기본적으로 Predicate와 Filter를 하나씩 정의합니다.

Default Predicate는 /serviceId/**과 같은 패턴으로 정의한 Path Predicate이며, 여기서 serviceId는 DiscoveryClient에서 등록한 Service의 ID입니다.

Default Filter는 /serviceId/?(?<remaining>.*) 과 같은 정규식과 /${remaining} 과 같은 replacement를 가진  Rewrite Path Filter 입니다. 위와같이 작성하면, 다운스트림으로 전송하기 전의 서비스 ID는 Path에서 제거됩니다.

추가로, DiscoveryClient route에서 사용할 predicate나 Filter는 spring.cloud.gateway.discovery.cloator.predicates[x], pring.cloud.gateway.discovery.locator.filters[y]와 같이 커스터마이즈가 필요한 대상을 프로퍼티 명으로써 값을 수정하여 정의할 수 있습니다.

```yaml
spring.cloud.gateway.discovery.locator.predicates[0].name: Path
spring.cloud.gateway.discovery.locator.predicates[0].args[pattern]: "'/'+serviceId+'/**'"
spring.cloud.gateway.discovery.locator.predicates[1].name: Host
spring.cloud.gateway.discovery.locator.predicates[1].args[pattern]: "'**.foo.com'"
spring.cloud.gateway.discovery.locator.filters[0].name: CircuitBreaker
spring.cloud.gateway.discovery.locator.filters[0].args[name]: serviceId
spring.cloud.gateway.discovery.locator.filters[1].name: RewritePath
spring.cloud.gateway.discovery.locator.filters[1].args[regexp]: "'/' + serviceId + '/?(?<remaining>.*)'"
spring.cloud.gateway.discovery.locator.filters[1].args[replacement]: "'/${remaining}'"
```

## CORS 동작 제어하기

(작성 예정)

## Actuator API 활용하기

(작성 예정)