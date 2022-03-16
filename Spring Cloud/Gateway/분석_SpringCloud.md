# Spring Cloud Gateway

Spring Cloud Gateway는 Spring Cloud 그룹에 포함되는 프로젝트입니다.

| Group ID | org.springframework.cloud |
| --- | --- |
| Artifact ID | spring-cloud-starter-gateway |

이름과 같이 통신구간 내 Gateway의 역할을 수행하여, API를 효과적으로 라우팅하고 Cross-cutting Concern을 제공합니다.

Spring Cloud Gateway를 사용하는데에는 아래의 3가지를 기억해야합니다.

🎁 **Route** : Gateway의 기본 빌딩 블록입니다. ID, 대상 URI, Predicate 및 Filter의 모음으로 정의합니다.

🎁 **Predicate** : Input Type은 Spring Framework의 ServerWebExhange입니다. Predicate를 통해, 헤더나 파라미터 등 HTTP 요청으로부터 전달된 항목이 작성한 Predicate와 일치하는지 요청의 유효성을 검증합니다. Spring Cloud Gateway 공식 문서를 통해, Built-in Route Predicate Factories를 확인하고, 원하는 형태로 작성하면 됩니다.

🎁 **Filter** : Gateway로 요청이 전달되면, 들어온 요청을 마이크로서비스에게 전달하기 이전 또는 후에 처리할 과정을 작성할 수 있습니다. Spring Cloud Gateway 공식 문서를 통해, Built-in WebFilter Factories를 확인하고, 원하는 형태로 작성하면 됩니다.

![img_1.png](../img/img.png)

Spring Cloud Gateway는 사용자로부터 전달된 요청이 사전 정의된 API 경로와 일치하다고 판단하면, Web Handler로 전달됩니다. Web Handler는 작성된 Filter를 통해 요청을 실행합니다. Filter 동작 이후, Proxied Service(Microservice)로 사용자의 요청이 전달됩니다.

## Application.yaml 설정

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: a_route
      uri: http://localhost/a
      predicates:
      - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

위와같이 predicate에 선언한 After Route Predicate Factory 의 경우 21년 3월 16일 한국시간 9시 이후에 전달된 요청에 한해, a_route ID를 가진 http://localhost/a URI로 요청이 전달됩니다.

위처럼 사전 정의된 Predicate Factory를 사용하면 Predicate를 쉽게 정의할 수 있습니다. 아래 링크에서는 [Route Predicate Factories](./분석_RoutePredicateFactories.md)들을 나열하고 간단한 예시를 정리해두었습니다.

