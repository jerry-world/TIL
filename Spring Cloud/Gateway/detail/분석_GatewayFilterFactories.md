# GatewayFilter Factories

---

### 😃 **The Add**RequestHeader GatewayFilter Factory

다운스트림으로 요청을 보내기 이전에, 새로운 헤더를 작성하여 보냅니다.

```yaml
routes:
- id: add_request_header_route
  uri: http://example.org
  filters:
  - AddRequestHeader=X-Request-red, blue
```

추가로, AddRequestHeader Filter는 URI 변수를 활용할 수도 있습니다.

```yaml
routes:
- id: add_request_header_route
  uri: http://example.org
  predicates:
  - Path=/red/**{segment}**
  filters:
  - AddRequestHeader=X-Request-red, blue-**{segment}**
```

### 😃 **The AddRequestParameter GatewayFilter Factory**

다운스트림으로 요청을 보내기 이전에, 새로운 파라미터를 작성하여 보냅니다. 작성할 때는 name과 value를 구분하여 파라미터를 작성합니다.

```yaml
routes:
- id: add_request_parameter_route
  uri: http://example.org
  filters:
  - AddRequestParameter=red, blue
```

마찬가지로, URI 변수를 활용할 수 있습니다.

```yaml
routes:
- id: add_request_parameter_route
  uri: http://example.org
  predicates:
  - Host={segment}.myhost.org
  filters:
  - AddRequestParameter=foo, bar-{segment}
```

### 😃 **The AddResponseHeader GatewayFilter Factory**

다운스트림의 응답 헤더에 새로운 헤더를 추가할 수 있습니다.

```yaml
routes:
- id: add_response_header_route
  uri: http://example.org
  filters:
  - AddResponseHeader=X-Request-red, blue
```

마찬가지로 URI 변수를 활용할 수 있습니다.

```yaml
routes:
- id: add_response_header_route
  uri: http://example.org
  predicates:
  - Path=/red/**{segment}**
  filters:
  - AddResponseHeader=X-Request-red, blue-**{segment}**
```

### 😃 **The DedupeResponseHeader GatewayFilter Factory**

응답 헤더의 중복을 제거합니다.

```yaml
routes:
  - id: dedupe_response_header_route
    uri: https://example.org
    filters:
    - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

CORS때문에? 응답헤더 중복제거인건 알겠는데 CORS이야기는 왜 계속 나오는거지....

### 😃 **The Spring Cloud CircuitBreaker GatewayFilter Factory**

Spring Cloud CircuitBreaker API를 사용하여, CircuitBreaker를 Wrapping 하는 Filter입니다. Resilience4J를 사용합니다.(CirtcuitBreaker를 학습하면 좀더 보충...)

```yaml
routes:
  - id: circuitbreaker_route
    uri: https://example.org
    filters:
      - CircuitBreaker=myCircuitBreaker
```

### 😃 **The FallbackHeaders GatewayFilter Factory**

### 😃 **The MapRequestHeader GatewayFilter Factory**

### 😃 **The PrefixPath GatewayFilter Factory**

### 😃 **The PreserveHostHeader GatewayFilter Factory**

### 😃 **The RequestRateLimiter GatewayFilter Factory**

### 😃 **The RedirectTo GatewayFilter Factory**

### 😃 **The RemoveRequestHeader GatewayFilter Factory**

### 😃 **The RemoveResponseHeader GatewayFilter Factory**

### 😃 **The RemoveResponseParameter GatewayFilter Factory**

### 😃 **The RewritePath GatewayFilter Factory**

### 😃 **The RewriteLocationResponseHeader GatewayFilter Factory**

### 😃 **The RewriteResponseHeader GatewayFilter Factory**

### 😃 **The SetPath GatewayFilter Factory**

### 😃 **The SetResponseHeader GatewayFilter Factory**

### 😃 **The SetRequestHostHeader GatewayFilter Factory**

### 😃 **The SaveSession GatewayFilter Factory**

### 😃 **The SecureHeaders GatewayFilter Factory**

### 😃 **The StripPrefix GatewayFilter Factory**

### 😃 **The Retry GatewayFilter Factory**

### 😃 **The RequestSize GatewayFilter Factory**

### 😃 Modify a Request Body **GatewayFilter Factory**

### 😃 Modify a Response Body **GatewayFilter Factory**

### 😃 Token Relay **GatewayFilter Factory**

### 😃 Default Filters