# Spring Cloud Netflix

Spring Cloud Netflix 프로젝트는 autoconfiguration이나 바인딩, **Spring Programming model idioms**를 통해, Spring Boot 기반의 앱을 위한 Netflix OSS 통합을 제공합니다. Spring Cloud Netflix에서 제공하는 간단한 Annotation들을 사용하여, 애플리케이션 내부의 공통 패턴을 빠르게 활성화하거나 설정할 수 있으며, 대형 분산 시스템을 빠르게 구축할 수 있습니다. 제공되는 패턴에는 Service Discovery(Eureka)가 포함됩니다.

## 😀 Service Discovery: Eureka Clients

Service Discovery는 마이크로서비스 기반의 아키텍처(MSA)의 핵심 요소중 하나입니다. 마이크로서비스에 해당하는 각 클라이언트를 개별적으로 정한 규칙에 따라 수동으로 구성하는 것은 불편하면서 동시에 휴먼 에러가 발생되기 쉽습니다. Netflix의 Service Discovery Eureka는 Server 와 Client를 제공합니다. 이 Eureka Server들은 각자 등록된 서비스들에 대한 상태를 복제하여 저장함으로써, 고가용성(High Availablity:HA)을 제공합니다.

## 😀 How to Include Eureka Client

Eureka Client 등록은 아주 간단합니다. org.springframework.cloud Group ID와 spring-cloud-starter-netflix-eureka-client Artifact ID를 Build System에 추가하세요.

| Group ID | org.springframework.cloud |
| --- | --- |
| Artifact ID | spring-cloud-starter-netflix-eureka-client |

## 😀 Registering with Eureka

Client가 Eureka를 등록하면, host, port, health indicator URL, home page 그리고 세부 사항 등에 해당하는 Client의 메타 데이터를 제공합니다. Eureka는 서비스에 해당하는 각각의 인스턴스로부터 heartbeat메시지를 받습니다. 만약, heartbeat가 configurable timetable을 통해 장애 조치되었다고 판단되면, 인스턴스는 일반적으로 레지스트리에서 제거됩니다.

아래는 Eureka Client Application을 구성하는 가장 간단한 예시입니다

```java
@SpringBootApplication
@RestController
public class Application {

  @RequestMapping("/")
  public String home() {
    return "Hello world";
  }

  public static void main(String[] args) {
    new SpringApplicationBuilder(Application.class).web(true).run(args);
  }
}
```

앞의 예시는 일반적인 SpringBoot Application과 형태가 동일합니다. 클래스 경로에 spring-cloud-starter-netflix-eureka-client가 존재하면, 애플리케이션은 자동으로 Eureka Server에 Client로 등록이 되는겁니다. 그러면, 대상 Client가 Eureka Server를 찾을 수 있도록 도와줘야 합니다.

**Application.yaml**

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

위 예시에서, defaultZone은 모든 Client에게 service URL을 제공하는 값입니다. preference에 해당하는 Zone을 작성하지 않은 경우, defaultZone으로 Eureka Server가 대체됩니다.

위에서 언급한 바와 같이, ClassPath에 spring-cloud-starter-netflix-eureka-client가 있을 경우 자동으로 Eureka Instance 및 Client로 등록됩니다. Instance의 동작은 eureka.instance.*에 해당하는 구성 키로 구동되며, spring.application.name 값이 존재하면, 굳이 작성하지 않아도 됩니다.

[Instance](https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java) 및 [Client](https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java) 구성 옵션

## 😀 Authentication with the Eureka Server

eureka.client.serviceUrl.defaultZone URL이 자격증명에 포함되어있을 경우, Eureka Client들은 자동으로 HTTP 기본인증이 추가됩니다. 만약 보다 복잡한 인증을 요구해야하는 경우 DiscovertClientOptionalArgs 유형의 Bean을 작성하고 ClientFilter 인스턴스를 주입할 수 있습니다. 이 인스턴스는 모두 클라이언트에서 서버로의 호출에 적용됩니다.

만약, Eureka 서버에 인증을 위해 Client 측 인증서가 필요한 경우는 아래와 같이 프로퍼티를 작성하여 Client 측 인증서 및 trust store를 구성할 수도 있습니다.

**application.yaml**