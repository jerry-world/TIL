# Service Discovery: Eureka Server

## 😀 How to include Eureka server

프로젝트에 Eureka Server를 추가하기 위해, `org.springframework.cloud` Group ID와
spring-cloud-`starter-netflix-eureka-server` artifact ID를 추가하세요.

**application.yaml**

```yaml
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    prefer-file-system-access: false
```

## 😀 How to Run a Eureka Server

아래는 Eureka server를 동작시키기 위한 최소한의 방법을 보여줍니다:

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {
	new SpringApplicationBuilder(Application.class).web(true).run(args);
}
```

Eureka Server는 기본적으로 Eureka 기능을 위한 UI 및 HTTP API endpoint를 제공하는 `/eureka/*` 홈페이지를 제공합니다.

>💡 Gradle의 종속성 해결규칙 및 부모 bom 기능의 부재로 인해, `spring-cloud-starter-netflix-eureka-server`가 application에서 실행되지 않을 수 있습니다. 따라서, 이러한 문제를 해결하기 위해, Spring cloud starter 부모 bom을 import하고, Spring Boot Gradle plugin을 추가해야 합니다.


```groovy
buildscript {
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:{spring-boot-docs-version}")
    }
}

apply plugin: "spring-boot"

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:{spring-cloud-version}"
    }
}
```

## 😀 High Availability, Zones and Regions

Eureka server는 backend 저장소를 가지고 있지 않습니다. 하지만, 레지스트리 상의 모든 service instance는 그들의 현재 상태를 계속적으로 확인할 수
있도록 하트비트를 전송해야 합니다(그래서, 메모리에서 수행할 수 있음). Client는 또한 Eureka 등록의 인-메모리 캐시를 가지고 있습니다(따라서 서비스에 대한 모든 요청에
대해 레지스트리로 이동할 필요가 없음).

기본적으로, 모든 Eureka Server는 Eureka Client 이기도하며, 동시에 피어를 찾기 위한 Service URL이 필요합니다. 만약 Service URL을 제공하지
않을 경우, 서비스는 동작하겠지만, 피어를 등록할 수 없다는 내용의 로그가 계속적으로 증적될 것입니다.

## 😀 Standalone Mode

2개의 캐시(Server 및 Client)와 하트비트의 조합은 모니터링이 유지되거나 Elastic runtime(Cloud Foundry 같은..)이 유지되는한 standalone
Eureka Server를 탄력적으로 만듭니다. standalone 모드에서는 클라이언트 측 동작을 해제하여 피어에 계속적으로 연결하는 행위를 하지 않도록 할 수 있습니다. 다음
예시는 클라이언트 측 동작(client-side behavior)를 끄는 방법을 보여줍니다.

**application.yaml**

```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

## 😀 Peer Awareness

Eureka는 동작중인 다중 인스턴스를 실행하고, 서로 각각에게 등록하도록 요청하여 보다 탄력적으로 운용할 수 있습니다. 아래의 예제와 같이 피어를 대상으로
유효한 `serviceUrl`을 추가하기만 하면, 구성이 완료됩니다.

**application.yaml (Two Peer Aware Eureka Servers)**

```yaml
---
spring:
  profiles: peer1
eureka:
  instance:
    hostanme: peer1
  client:
    serviceUrl:
      defaultZone: https://peer2/eureka/

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/
```

위의 예시에서는 서로 다른 두 Peer가 서로를 인식하여, 동일한 Server 환경을 만들 수 있는 예시였습니다. `eureka.instance.hostname`
은 `/etc/hosts` 파일로 부터 자기 자신의 호스트 명을 인지할 수 있는 OS 환경에서는 사용하지 않아도 됩니다.

시스템에 여러 피어를 추가할 수 있으며, 모두 최소한 하나의 edge로 서로 연결되어있으면 서로간 등록을 통해 동기화할 수 있습니다. 피어가 물리적으로 분리되어있는 경우
split-brain 이 발생하는 경우를 대비할 수 있습니다. 여러 피어를 시스템에 추가할 수 있으며, 모두 서로에게 직접 연결되어있으면 서로 등록을 동기화합니다.

**application.yaml (Three Peer Aware Eureka Servers)**

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/,http://peer2/eureka/,http://peer3/eureka/

---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2

---
spring:
  profiles: peer3
eureka:
  instance:
    hostname: peer3
```

## 😀 When to Prefer IP Address

필요에 따라 Eureka 등록을 hostname이 아닌 IP로 사용해야할 때도 있습니다. `eureka.instance.preferIpAddress`를 `true`로 설정하면,
hostname이 아닌 IP를 사용합니다.

## 😀 Securing The Eureka Server

`spring-boot-starter-security`를 서버의 ClassPath에 포함하고만 있으면됩니다. 기본적으로 Spring Seucrity가 ClassPath에
포함하고있을 경우, CSRF token을 보내도록 되어있습니다. Eureka Client는 일반적으로 유효한 CSRF(Cross-Site Request Forgery) 토큰을
소유하고 있지 않기 때문에, Spring Security 설정을 통해, /eureka/** 엔드포인트에 대해 이 요구사항을 비활성화 해야 합니다.

```java

@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

  @Override
  protected void confiure(HttpSecurity http) throws Exception {
    http.csrf().ignoringAntMatchers("/eureka/**");
    super.configure(http);
  }
```

## 😀 JDK 11 Support

Eureka 서버가 의존하는 JAXB 모듈은 JDK 11에서 제거되었습니다. Eureka 서버를 실행할 때 JDK 11을 사용하려면 POM 또는 Gradle 파일에 이러한 종속성을
포함해야 합니다.