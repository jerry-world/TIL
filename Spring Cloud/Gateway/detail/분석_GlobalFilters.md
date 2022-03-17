# Global Filters

Global Filter는 GatewayFilter와 역할을 동일하고, 모든 route에 조건부로 적용되는 필터입니다.

## 😃 Combined Global Filter and GatewayFilter Ordering

새로운 HTTP 요청이 Gateway로 들어오고, 대상 요청의 route가 매칭되면 Web Handler는 모든 GlobalFilter 인스턴스와 route에 작성되어있는 Filter를 Filter Chain에 추가합니다. org.springframework.core.Ordered 인터페이스의 getOrder()메서드를 구현하면 원하는 형태로 정렬을 수행할 수 있습니다.

Spring Cloud Gateway는 필터로직을 수행할 떄, ‘pre’ 단계와 ‘post’ 단계로 구분됩니다. 따라서 우선순위가 높은 기준으로 ‘pre’에서는 첫번째로 수행하고, ‘post’단계에서는 가장 마지막에 수행합니다.

```java
@Bean
public GlobalFilter customFilter(){
  return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered{

  @Override
  public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
    log.info("custom global filter");
    return chain.filter(exchange);
  }

  @Override
  public int getOrder(){
    return -1;
  }
```

## 😃 Forward Routing Filter

ForwardRoutingFilter 는 exchange 속성 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR  에서 URI를 탐색하여, URL에 Foward 스킴이 존재할 경우, 스프링 DispatcherHandler를 사용하여  요청 URL의 path를 Foward URL로 대체하고 이를 재 라우팅합니다. 원래의 URL의 경우 ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR 속성에 존재하는 리스트에 추가합니다.

## 😃 The LoadBalancerClient Filter

LoadBalancerClientFilter는 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR의 exchange 속성에서 URI를 찾고, 해당 URL에 LB 스킴이 존재할 경우, Spring Cloud LoadBalancerClient를 사용하여, URI를 대체합니다.

```yaml
routes:
- id: myRoute
  uri: lb://service
  predicates:
  - Path=/service/**
```

## 😃 The ReactiveLoadBalancerClientFilter

ReactiveLoadBalancerClientFilter는 LoadBalancerClient Filter와 유사하게 동작하지만, ReactorLoadBalancer를 사용하여, LB 스킴의 정보를 리졸브하고 URI를 대체합니다. 설정 방법은 LoadBalancerClient Filter 와 동일합니다.

```yaml
routes:
- id: myRoute
  uri: lb://service
  predicates:
  - Path=/service/**
```

## 😀 The Netty Routing Filter

Netty Routing Filter는 exchange의 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR 속성에서 URL에 http나 https 스킴이 존재할 경우 실행됩니다. 이 경우, Netty HttpClient를 사용하여 다운스트림으로 프록시 요청을 생성합니다.

## 😀 The Netty Write Response Filter

NettyWriteResponseFilter는 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR에 HttpClientREsponse가 있을 때 실행됩니다. 다른 필터를 모두 완료한 뒤에, HTTP 요청한 Client에게 응답을 되돌려줄 때, 프롲시 응답을 작성합니다.

## 😀 The RouteToRequestUrl Filter

RouteToRequestUrl Filter는 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR에 Route 객체가 있을 때 실행됩니다. 요청 URI를 기반으로 새 URI를 작성할 때, Route 객체의 URI 속성을 반영하여 작성됩니다.

## 😀 The Websocket Routing Filter

WebsocketRouting Filter는 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR에 ws 또는 wss 스킴이 있을 때 실행됩니다. 이때는 Spring WebScoket Infra를 통해, 다운스트림으로 웹소켓 요청을 전달합니다.

```yaml
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

## 😀 The Gateway Metrics Filter

Gateway Metric을 활성화하려면 spring-boot-starter-actuator 의존성을 추가해야합니다. 또한, application.yaml에 spring.cloud.gatway.metrics.enabled 프로퍼티를 false로 명시하면 안됩니다.(기본값이 true). The Gateway Metrics Filter는 아래의 태그로 gatway.requests라는 타이머 메트릭을 추가합니다.

- routeId: route ID
- routeUri: API를 라우팅한 URI
- outcome: HttpStatus.Series로 분류한 결과
- status: HTTP 상태
- httpStatusCode: HTTP 상태 코드
- httpMethod: HTTP 메서드

  ![spring-cloud-gateway-metrics.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cb33206c-12f0-471f-b8d5-8fc5660ad355/spring-cloud-gateway-metrics.png)

  ## 😀 Marking An Exchange As Routed

  Gateway에서 ServerWebExchange를 라우팅 한 후에 exchange 속성에 gatewayAlreadyRouted 속성을 추가하여 라우팅 되었음을 마킹할 수 있습니다. Exhange 속성에 라우팅 여부를 마킹했는지 확인할 수 있는 메서드도 제공됩니다.

  ServerWebExchangeUtils.isAreadyRouted