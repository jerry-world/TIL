이 섹션에서는 Spring Security의 모듈에 대한 참조와 실행중인 애플리케이션에서 작동하기 위해 필요한 추가 종속성을 제공합니다. Spring Security 자체를 빌드하거나 테스트할 때만 사용되는 종속성은 포함되어있지 않습니다. 또한 외부 종속성이 필요한 이행 종속성을 포함하지 않습니다.

필요한 Spring 버전은 프로젝트 웹사이트에 나열되어 있으므로 아래 Spring 종속성에 대한 특정 버전은 생략됩니다. 아래에 “Optional”로 나열된 종속성 중 일부는 Spring 애플리케이션의 다른 비보안 기능에 여전히 필요할 수 있습니다. 또한 “Optional”로 나열된 종속성은 대부분의 응용프로그램에서 사용되는 경우 프로젝트의 Maven POM 파일에 실제로 그렇게 표시되지 않을 수 있습니다. 지정된 기능을 사용하지 않는 한 필요하지 않다는 의미에서 “Optional”을 이해하셔야 됩니다.

## 😀 Core — spring-security-core.jar

이 모듈은 인증 및 ACL 클래스 그리고 인터페이스, 원격 지원, 기본 프로비저닝 API를 포함하고 있습니다. Core는 Spring Security를 사용하는 그 모든 애플리케이션에서 필요한 모듈입니다. Core는 standalone 애플리케이션, 원격 클라이언트, 메서드(service 레이어) 보안, JDBC 사용자 프로비저닝 을 지원합니다. Core는 아래의 최상위 패키지를 가지고 있습니다.

- org.springframework.security.core
- org.springframework.security.access
- org.springframework.security.authentication
- org.springframework.security.provisioning

**Core  Dependencies**

| Dependency | Version | Description |
| --- | --- | --- |
| ehcache | 1.6.2 | Ehcache 기반 사용자 캐시 구현이 사용되는 경우 필요함(Optional) |
| spring-aop |  | Spring AOP 기반 메서드 보안 |
| spring-beans |  | Spring 구성에 필요 |
| spring-expression |  | expression 기반 메서드 보안에 필수(Optional) |
| spring-jdbc |  | 데이터베이스를 사용하여 사용자 데이터를 저장하는 경우 필요함(Optional) |
| spring-tx |  | 데이터베이스를 사용하여 사용자 데이터를 저장하는 경우 필요함(Optional) |
| aspectjrt | 1.6.10 | AspectJ Support를 사용하는 경우 필수(Optional) |
| jsr250-api | 1.0 | JSR-250 메서드 보안 주석(Optional)을 사용하는 경우 필요함 |

## 😀 Remote — spring-security-remoting.jar

이 모듈은 Spring Remoting과의 통합을 제공합니다. Spring Remoting을 사용하는 원격 클라이언트를 작성하지 않을 경우 이 모듈은 필요하지 않습니다. 주요 패키지는 `org.springframework.security.remoting` 입니다.

**Remote Dependencies**

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| spring-web |  | HTTP 원격 지원을 사용하는 클라이언트가 있을 경우 필요함 |

## 😀 Web — spring-security-web.jar

이 모듈은 필터들과 web-scurity 인프라스트럭쳐 코드와 관련된 것들을 포함하고 있습니다. Web은 servlet API 의존성에 관련된 모든 것을 담고 있습니다.  Spring Security Web 인증 서비스와 URL 기반 access-control이 필요할 경우 이 모듈이 필요합니다. 기본 패키지는 `org.springframework.security.web` 입니다.

Web Dependencies

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| spring-web |  | Spring web 지원 클래스는 광범위하게 사용됨 |
| spring-jdbc |  | JDBC 기반 영구 remember-me 토큰 저장소에 필요(Optional) |
| spring-tx |  | remember-me 영구 토큰 저장소 구현체에 필요(Optional) |

## 😀 Config — spring-security-config.jar

이 모듈은 namespace 파싱 코드, Java configuration 코드가 포함되어 있습니다. 만약 configuration을 위한 Spring Security XML 네임스페이스 또는 Spring Security의 Java Configuration 지원이 필요한 경우 필요합니다. 기본 패키지는 `org.springframework.security.config` 입니다.

Config Dependencies

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| spring-security-web |  | 그 어떤 Web관련 namespace configuration을 사용할 경우 필요함(Optional) |
| spring-security-ldap |  | LDAP namespace 옵션을 사용할 경우 필요함(Optional) |
| spring-security-openid |  | OpenID 인증을 사용할 경우 필요함(Optional) |
| aspectjweaver | 1.6.10 | protect-pointcut namespace syntax를 사용하는 경우 필요함(Optional) |

## 😀 LDAP — spring-security-ldap.jar

이 모듈을 LDAP 인증 및 프로비저닝 코드를 제공합니다. 만약 LDAP 인증 또는 LDAP 사용자를 관리해야할 경우 이 모듈이 필요합니다. 최상위 패키지는 `org.springframework.security.ldap`입니다.

**LDAP Dependencies**

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| spring-ldap-core | 1.3.0 | LDAP support는 Spring LDAP에 기반함. |
| spring-tx |  | 데이터 예외 클라스에 필요함. |
| apache-ds | 1.5.5 | embedded LDAP server를 사용할 경우 필요함(Optional) |
| shared-ldap | 0.9.15 | embedded LDAP server를 사용할 경우 필요함(Optional) |
| ldapsdk | 4.1 | Mozilla LdapSDK. 예를들어, OpenLDAP에서 암호 정책 기능을 사용하는 경우 LDAP 암호 정책 제어를 디코딩하는데 사용. |

## 😀 OAuth 2.0 Core — spring-security-oauth2-core.jar

`spring-security-oauth2-core.jar`에는 OAuth 2.0 Authorization Framework 및 OpenID Connect Core 1.0을 지원하는 핵심 클래스와 인터페이스가 포함되어 있습니다. 클라이언스, 리소스 서버, 권한 서버를 사용하는 OAuth 2.0 이나 Open ID Connect Core 1.0에서 필요합니다. 최상위 패키지는 `org.springframework.security.oauth2.core` 입니다.

## 😀 OAuth 2.0 Client — spring-security-oauth2-client.jar

`spring-security-oauth2-client.jar`에는 OAuth 2.0 Authorization Framework 및 OpenID Connect Core 1.0에 대한 Spring Security의 Client Support가 포함되어 있습니다. OAuth 2.0 로그인 또는 OAuth Client Support를 사용하는 애플리케이션에 필요합니다. 최상위 패키지는 `org.springframework.security.oauth2.client` 입니다.

## 😀 OAuth 2.0 JOSE - spring-security-oauth2-jose.jar

`spring-security-oauth2-jose.jar`는 JOSE(Javascript Object Signing and Encryption) 프레임워크를 위한 Spring Security 지원이 포함되어 있습니다. JOSE framework는 당사자 간에 청구를 안전하게 이전하는 방법을 제공하기 위한 것입니다. 다음 사양의 컬렉션으로 구성되어 있습니다:

- JSON Web Token (JWT)
- JSON Web Signature (JWS)
- JSON Web Encryption (JWE)
- JSON Web Key (JWK)

OAuth 2.0 JOSE의 최상위 패키지는 아래와 같습니다:

- `org.springframework.security.oauth2.jwt`
- `org.springframework.security.oauth2.jose`

## 😀 **OAuth 2.0 Resource Server — spring-security-oauth2-resource-server.jar**

`spring-security-oauth2-resource-server.jar` 은 OAuth 2.0 리소스 서버에 대한 Spring Security의 지원이 포함되어 있습니다. OAuth 2.0 Bearer Token을 통해 API를 보호하는 데에 사용됩니다. 최상위 패키지는 `org.springframework.security.oauth2.server.resource` 입니다.

## 😀 **ACL — spring-security-acl.jar**

이 모듈은 특수 도메인 객체 ACL 구현체를 포함하고 있습니다. 이 모듈은 애플리케이션 내의 특정 도메인 객체 인스턴스에 보안을 적용하는 데 사용됩니다. 최상위 패키지는 `org.springframework.security.acls`입니다.

**ACL Dependencies**

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| ehcache | 1.6.2 | Ehcache 기반 ACL 캐시 구현이 사용되는 경우 필요함(자체 구현을 사용하는 경우 선택 사항). |
| spring-jdbc |  | 기본 JDBC 기반 AclService를 사용하는 경우 필요함(사용자 고유의 AclService를 구현하는 경우 선택 사항). |
| spring-tx |  | 기본 JDBC 기반 AclService를 사용하는 경우 필요함(사용자 고유의 AclService를 구현하는 경우 선택 사항). |

## 😀 **CAS — spring-security-cas.jar**

이 모듈은 Spring Security의 CAS 클라이언트 통합을 포함하고 있습니다. CAS single sign-on 서버와 함께 Spring web 인증을 사용하길 원한다면 이 모듈을 사용해야합니다. 최상위 패키지는 `org.springframework.security.cas` 입니다.

**CAS Dependencies**

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| spring-security-web |  |  |
| cas-client-core | 3.1.12 | JA-SIG CAS 클라이언트. 이는 Spring Security 통합의 기초 |
| ehcache | 1.6.2 | Ehcache 기반 티켓 캐시를 사용하는 경우 필수입니다(선택 사항). |

## 😀 **OpenID — spring-security-openid.jar**

>💡 OpenID 1.0 및 2.0 프로토콜은 더 이상 사용되지 않으며, 아직 이를 사용하는 사용자는 spring-security-oauth2에서 지원하는 OpenID Connect로 마이그레이션하는 것이 좋습니다.

이 모듈은 OpenID 웹 인증 지원이 포함되어 있습니다. 외부 OpenID 서버에 대해 사용자를 인증하는 데 사용합니다. 최상위 패키지는 `org.springframework.security.openid` 입니다. 이 모듈은 OpenID4Java 가 필요합니다.

**OpenID Dependencies**

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| spring-security-web |  |  |
| openid4java-nodeps | 0.9.6 | Spring Security OpenID 통합은 OpenID 4 Java를 사용함. |
| httpclient | 4.1.1 | openid4java-nodeps는 HttpClient 4에 의존함. |
| guice | 2.0 | openid4java-nodeps는 Guice 2에 의존함. |

## 😀 **Test — spring-security-test.jar**

이 모듈은 Spring Security로 테스트하기 위한 지원이 포함되어 있습니다.

## 😀 **Taglibs — spring-secuity-taglibs.jar**

Spring Security의 JSP 태그 구현을 제공합니다.

**Taglib Dependencies**

| Dependency | Version | Description |
| --- | --- | --- |
| spring-security-core |  |  |
| spring-security-web |  |  |
| spring-security-acl |  | ACL(선택 사항)과 함께 accesscontrollist 태그 또는 hasPermission() 표현식을 사용하는 경우 필요함 |
| spring-expression |  | 태그 접근 제약 조건에서 SPEL 표현식을 사용하는 경우 필요함 |