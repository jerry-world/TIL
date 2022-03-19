# Spring Cloud Netflix

Spring Cloud Netflix 프로젝트는 autoconfiguration이나 바인딩, **Spring Programming model idioms**를 통해, Spring
Boot 기반의 앱을 위한 Netflix OSS 통합을 제공합니다. Spring Cloud Netflix에서 제공하는 간단한 Annotation들을 사용하여, 애플리케이션 내부의
공통 패턴을 빠르게 활성화하거나 설정할 수 있으며, 대형 분산 시스템을 빠르게 구축할 수 있습니다. 제공되는 패턴에는 Service Discovery(Eureka)가 포함됩니다.

## 😀 Service Discovery: Eureka Clients

Service Discovery는 마이크로서비스 기반의 아키텍처(MSA)의 핵심 요소중 하나입니다. 마이크로서비스에 해당하는 각 클라이언트를 개별적으로 정한 규칙에 따라 수동으로
구성하는 것은 불편하면서 동시에 휴먼 에러가 발생되기 쉽습니다. Netflix의 Service Discovery Eureka는 Server 와 Client를 제공합니다. 이
Eureka Server들은 각자 등록된 서비스들에 대한 상태를 복제하여 저장함으로써, 고가용성(High Availablity:HA)을 제공합니다.

아래의 문서를 통해, Eureka Clients에 대해 학습할 수 있습니다.

[📔Service Discovery: Eureka](./detail/분석_ServiceDiscovery_EurekaClients.md)

