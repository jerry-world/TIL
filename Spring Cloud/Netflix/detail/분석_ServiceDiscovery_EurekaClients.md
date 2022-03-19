# Service Discovery:Eureka Clients

---

## 😀 How to Include Eureka Client

Eureka Client 등록은 아주 간단합니다. org.springframework.cloud Group ID와
spring-cloud-starter-netflix-eureka-client Artifact ID를 Build System에 추가하세요.

| Group ID | org.springframework.cloud |
| --- | --- |
| Artifact ID | spring-cloud-starter-netflix-eureka-client |

## 😀 Registering with Eureka

Client가 Eureka를 등록하면, host, port, health indicator URL, home page 그리고 세부 사항 등에 해당하는 Client의 메타 데이터를
제공합니다. Eureka는 서비스에 해당하는 각각의 인스턴스로부터 heartbeat메시지를 받습니다. 만약, heartbeat가 configurable timetable을 통해
장애 조치되었다고 판단되면, 인스턴스는 일반적으로 레지스트리에서 제거됩니다.

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

앞의 예시는 일반적인 SpringBoot Application과 형태가 동일합니다. 클래스 경로에 spring-cloud-starter-netflix-eureka-client가
존재하면, 애플리케이션은 자동으로 Eureka Server에 Client로 등록이 되는겁니다. 그러면, 대상 Client가 Eureka Server를 찾을 수 있도록 도와줘야
합니다.

**Application.yaml**

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

위 예시에서, defaultZone은 모든 Client에게 service URL을 제공하는 값입니다. preference에 해당하는 Zone을 작성하지 않은 경우,
defaultZone으로 Eureka Server가 대체됩니다.

위에서 언급한 바와 같이, ClassPath에 spring-cloud-starter-netflix-eureka-client가 있을 경우 자동으로 Eureka Instance 및
Client로 등록됩니다. Instance의 동작은 eureka.instance.*에 해당하는 구성 키로 구동되며, spring.application.name 값이 존재하면, 굳이
작성하지 않아도 됩니다.

[**Instance](https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)
및 [Client](https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java)
구성 옵션**

## 😀 Authentication with the Eureka Server

eureka.client.serviceUrl.defaultZone URL이 자격증명에 포함되어있을 경우, Eureka Client들은 자동으로 HTTP 기본인증이 추가됩니다. 만약
보다 복잡한 인증을 요구해야하는 경우 DiscovertClientOptionalArgs 유형의 Bean을 작성하고 ClientFilter 인스턴스를 주입할 수 있습니다. 이
인스턴스는 모두 클라이언트에서 서버로의 호출에 적용됩니다.

만약, Eureka 서버에 인증을 위해 Client 측 인증서가 필요한 경우는 아래와 같이 프로퍼티를 작성하여 Client 측 인증서 및 trust store를 구성할 수도
있습니다.

**application.yaml**

```yaml
eureka:
  client:
    tls:
      enabled: true
      key-store: <path-of-key-store>
      key-store-type: PKCS12
      key-store-password: <key-store-password>
      key-password: <key-password>
      trust-store: <path-of-trust-store>
      trust-store-type: PKCS12
      trust-store-password: <trust-store-password>
```

## 😀 Status Page and Health Indicator

Spring Boot Actuator 앱에서 Status 페이지와 health indicator는 각각 /info 와 /health로 기본 설정됩니다.

만약 이 기본 페이지를 변경하고 싶다면, 아래와 같이 설정하면 됩니다.

```yaml
eureka:
  instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health
```

## 😀 Registering a Secure Application

만약 HTTPS를 연결하길 원한다면, 아래의 두 프로퍼티를 설정하세요.

```java
//EnrekaInstanceConfigBean
eureka.instance.nonSecurePortEnabled=false
    eureka.instance.securePortEnabled=true
```

위와 같이 설정하면, Eureka가 인스턴스 정보 상에 위의 인스턴스는 암호 통신을 필요로 한다고 명시적으로 기재합니다. Spring Cloud는 위와같이 암호 통신이 필요로 하는
DicoveryClient에 대해 Endpoint URL을 항상 출력해줍니다.

위와 같이 설정을 했으면, statusPageUrl, healthCheckUrl, homePageUrl에 대한 프로퍼티 값도 정상적으로 https 통신을 수행할 수 있도록 URL을
작성해주어야합니다.

```yaml
eureka:
  instance:
    statusPageUrl: https://${eureka.hostname}/info
  healthCheckUrl: https://${eureka.hostname}/health
  homePageUrl: https://${eureka.hostname}/
```

## 😀 Eureka’s Health Checks

기본적으로, Eureka는 heartbeat를 통해 Client가 살아있는지의 여부를 확인합니다. 특별히 설정하지 않으면 Discvoery Client는 Spring Boot
Actuator에 따라 애플리케이션의 현재 상태를 전달하지 않습니다. 설정한 Client에 대해 Eureka는 Application의 상태가 ‘UP’인지의 여부를 체크합니다.

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

com.netflix.appinfo.HeatlCheckHandler의 구현체를 참고하면 더 많은 설정 정보를 확인할 수 있습니다.

## 😀 Eureka Metadata Instances and Clients

### Using Eureka on Cloud Foundry

[https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#using-eureka-on-cloud-foundry](https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#using-eureka-on-cloud-foundry)

### Using Eureka on AWS

[https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#using-eureka-on-aws](https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#using-eureka-on-aws)

## 😀 Changing the Eureka Instance ID

바닐라 Netflix Eureka instance는 host name을 ID로 등록합니다. Spring Cloud Eureka는 아래와 같은 합리적인 기본값을 제공합니다.

${spring.cloud.client.hostname}:${spring.application.name}:${sporing.application.instance_id:
${server.port}}

아래와 같이 프로퍼티를 작성하여 재정의할 수 있습니다.

```yaml
eureka:
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

## 😀 Using the EurekaCleint

하나 이상의 Discovery Client인 애플리케이션이 존재하면, Eureka Server로부터 서비스 인스턴스를 검색할 수 있습니다. 다음과 같이
com.netflix.discovery.EurekaClient를 사용하면 됩니다.

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl(){
    InstanceInfo instance-discoveryClient.getNextServerFromEureka("STORES",false);
    return instance.getHomePageUrl();
    }
```

### EurekaClient with Jersey

[https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#eurekaclient-with-jersey](https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#eurekaclient-with-jersey)

## 😀 Alternatives to the Native Netflix EurekaClient

[https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#alternatives-to-the-native-netflix-eurekaclient](https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#alternatives-to-the-native-netflix-eurekaclient)

## 😀 Why Is It so Slow to Register a Service?

기본적으로, 인스턴스는 serviceURL을 통해 30초 주기로 하트비트 통신하여 서비스를 등록합니다. 메타데이터가 일치해질때까지는 어느정도 시간이 걸리기때문에, Server가
Client의 정보를 수신하는데까지 어느정도 시간이 걸리 수 밖에 없습니다. 물론, 설정값을 통해 30초 주기보다 더 빠르게 앞당길 수는 있지만, 프로덕션 환경에서는 성능적 측면을
고려하여 가급적이면 기본 설정을 따르는 것을 권고합니다.

```java
eureka.instance.leaseRenewalIntervalInSeconds=30
```

## 😀 Zones

만약 Eureka Client를 Multiple zone에 배포할 경우, 특정 Zone에 우선권을 부여할 수 있습니다. Eureka Client마다 이를 설정해주면 됩니다.

먼저, [zones and regions](https://docs.spring.io/spring-cloud-netflix/docs/3.1.1/reference/html/#spring-cloud-eureka-server-zones-and-regions) 문서를 통해, 각각의 Zone과 Peer를 구성해주어야 합니다. 위의 링크를 참고하세요.

다음으로, 서비스가 속해있어야 할 Zone을 지정해줘야 합니다. metadataMap 프로퍼티를 사용하여 설정하면 되는데, 만약 service1이 zone1과 zone2에 모두 배포되어 있다면, 각 zone에 속해 있다고 설정해주어야 합니다.

아래의 예시를 확인하세요.

Service 1 in Zone 1

```java
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

Service 1 in Zone 2
```java
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

## 😀 Refreshing Eureka Clients

기본적으로, EurekaClient 빈은 새로고침이 가능합니다. 즉, Eureka Client의 Properties를 수정하고 새로고침할 수 있다는 뜻입니다. 새로고침이 발생한 Client는 Eureka Server로부터 등록이 해제되고, 주어진 서비스의 모든 인스턴를 일정 시간 사용할 수 없게 됩니다. 이를 방지하기 위해서, refresh 설정을 비활성화할 수도 있습니다.

```java
eureka.client.refresh.enable=false
```

## 😀 ****Using Eureka with Spring Cloud LoadBalancer****

Spring Cloud LoadBalancer `ZonePreferenceServiceInstanceListSupplier` 를 사용할 수 있습니다. Eureka instance metadata(`eureka.instance.metadataMap.zone`)의 `zone`은 Instance 별로 Service Instance를 필터링하는데 사용되는 spring-cloud-loadbalancer-zone 속성 값을 설정하는데 사용됩니다.

위의 내용을 설정하지 않고 `spring.cloud.loadbalancer.eureka.approximateZoneFromHostname`의 값을 `true`로 설정할 경우, 서버 호스트 이름의 도메인 이름을 zone에 대한 proxy로 사용할 수 있습니다.

Zone 데이터의 다른 소스가 없으면 Client 구성(Instance가 아닌)을 기반으로 추측되어집니다. region 이름에서 zone 목록까지의 맵인 `eureka.client.availabilityZones`를 가져 와서 Instance의 자체 region의 첫 번째 영역(즉, `eureka.client.region`을 기본 Netflix와의 호환성을 위해 "us-east-1"로 기본 설정됨)을 가져 옵니다.