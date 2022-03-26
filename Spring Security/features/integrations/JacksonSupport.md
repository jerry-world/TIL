# Jackson Support

Spring security는 Spring Security와 관련된 클래스를 영속화할 수 있도록 Jackson을 지원합니다. 때문에, 분산 세션(즉, 세션복제, 스프링 세션, 등)을 사용할 때 스프링 시큐리티 관련 클래스의 직렬화 성능을 끌어올릴 수 있습니다.

이를 사용하려면, `ObjectMapper`에 `SecurityJackson2Modules.getModules(ClassLoader)`(https://github.com/FasterXML/jackson-databind):

```java
ObjectMapper mapper = new ObjectMapper();
ClassLoader loader = getClass().getClassLoader();
List<Module> modules = SecurityJackson2Modules.getModules(loader);
mapper.registerModules(modules);

// ... use ObjectMapper as normally ...
SecurityContext context = new SecurityContextImpl();
// ...
String json = mapper.writeValueAsString(context);
```


> 💡 Jackson 지원을 제공하는 Spring Security 모듈은 아래와 같습니다:
> - spring-security-core (`CoreJackson2Module`)
> - spring-security-web ( `WebJackson2Module`, `WebServletJackson2Module`, `WebServerJackson2Module` )
> - [spring-security-oauth2-client](https://docs.spring.io/spring-security/reference/servlet/oauth2/client/index.html#oauth2client) (`OAuth2ClientJackson2Module`)
> - spring-security-cas (`CasJackson2Module`)