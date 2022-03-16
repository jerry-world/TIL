# Route Predicate Factories

### 😃 **The After Route Predicate Factory**

작성된 datetime 이후에 발생한 요청만을 매칭시켜주는 Predicate입니다.  사용되는 파라미터는 datetime 입니다. 지정된 datetime보다 이후에 발생한 요청만을 매칭시켜주는 Predicate입니다.

```yaml
#생략
predicates:
- After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

### 😃 **The Before Route Predicate Factory**

After 와 반대로 작성된 datetime 이전에 발생한 요청만을 매칭시켜주는 Predicate입니다.

```yaml
predicates:
- Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

### 😃 **The Between Route Predicate Factory**

이름처럼 지정된 datetime 사이에 발생한 요청만을 매칭시켜주는 Predicate입니다. 사이의 값이 대상이기 때문에 필요한 파라미터도 2개입니다. After와 Before. 반드시 값을 입력할때 첫번째 인자는 After에 해당하는 값이어야하고, 두번째 인자는 Before에 해당하는 값이어야합니다.

```yaml
predicates:
- Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

### 😃 **The Cookie Route Predicate Factory**

특정 Cookie의 값이 value로 작성하는 정규식과 일치할 때 요청을 매칭시켜주는 Predicate입니다. 대상 Cookie와 그 Cookie의 값이 대상이기 때문에 필요한 파라미터도 2개입니다. Cookie의 이름과 자바 정규 표현식. 첫번째 파라미터는 이름이어야하고, 두번쨰 인자는 자바 정규 표현식이어야합니다.

```yaml
predicates:
- Cookie=chocolate, ch.p
```

### 😃 **The Header Route Predicate Factory**

HTTP 요청의 특정 Header와 그 Header의 값이 일치할 때 요청을 매칭시켜주는 Predicate입니다. 마찬가지로 Header Predicate도 파라미터가 2개입니다. Header의 이름과 자바 정규표현식. 첫번째 파라미터는 이름이어야하고, 두번쨰 인자는 자바 정규 표현식이어야합니다.

```yaml
predicates:
- Header=X-Request-Id, \d+
```

### 😃 **The Host Route Predicate Factory**

HTTP 요청한 대상의 Host의 값이 일치할 때 요청을 매칭시켜주는 Predicate입니다. 파라미터로는 Host의 URI를 나열하는데,  Ant-style 패턴으로  **,**   로 Host 정보를 나열합니다.

```yaml
predicates:
- Host=**.a.co.kr,**b.co.kr
```

추가로, Host 정보를 URI 템플릿으로 변수화하여, 값을 추출하고 사용할 수도 있습니다.

이를테면, {sub},a.co.kr 로 Host정보를 작성하였으면, **ServerWebExchange.getAttributes()**로부터 **URI_TEMPLATE_VARIABLES_ATTRIBUTE**에 정의된 키로 값을 추가하여 작성할 수 있습니다. 그러면 Filter에서도 변수 값을 활용할 수 있게 됩니다.

### 😃 **The Method Route Predicate Factory**

요청의 HTTP Method 대상이 일치할 때 요청을 매칭시켜주는 Predicate입니다. 파라미터로는 HTTP Method를 작성합니다.

```yaml
predicates:
- Method=GET,POST
```

### 😃 **The Path Route Predicate Factory**

Path Predicate는 요청의 Path가 일치할 때 매칭시켜주는 Predicate입니다. 스프링의 PathMatcher와 리스트 Patterns를 사용하며, matchTrailingSlash라는 플래그값을 제공하여,  /  로 끝나는 Path를 허용할지에 대한 여부를 의사결정할 수 있습니다.

```yaml
predicates:
- Path:/a/{segment},/b/{segment}
matchTrailingSlash=false
```

/a/apple, /b/banana 와 같은 Path는 모두 일치합니다. 다만, /a/apple/와 같은 Path는 matchTrailingSlash가 false이기 때문에 일치하지 않다고 판단하여, 요청이 매칭되지 않습니다.

### 😃 **The Query Route Predicate Factory**

HTTP 요청 상 Query에 작성된 키와 해당 키의 값이 표현식과 일치할 때 매칭시켜주는 Predicate입니다. 키값과 자바 정규표현식. 첫번째 파라미터는 키의 값이어야하고, 두번쨰 인자는 자바 정규 표현식이어야합니다.

```yaml
predicates:
- Query=red, app.
```

red라는 키의 값이 apple이거나 apples와 같이 app이 전방에 포함되어있으면 일치합니다.

### 😃 **The RemoteAddr Route Predicate Factory**

파라미터는 CIDR 표기로 작성합니다. 요청된 대상의 RemoteAddr 값에 따라 CIDR 내 포함되면 일치합니다.

```yaml
predicates:
- RemoteAddr=192.168.1.1/24
```

192.168.1.1의 24bit 내 대역에 존재하는 요청의 RemoteAddr에 대해 일치합니다.

### 😃 **The Weight Route Predicate Factory**

Weight Predicate는 그룹을 분리하고 가중치를 작성하여, 트래픽의 정도에 따라 대상 그룹으로 요청을 전달하는 Predicate입니다. group과 weight 파라미터를 사용합니다.

```yaml
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

80%의 요청은 weigh_high로 요청이 전달되고, 20%의 요청은 weigh_low로 전달됩니다.