#Spring Security

본 내용은 Spring Security 5.6.2에 준하여 작성된 글입니다.

Spring Security는 인증 및 인가를 지원하며, 주요 공격으로부터 애플리케이션을 보호해주는 framework입니다. 명령형과 리액티브 애플리케이션을 모두 보안을 위해 지원합니다.

## 😀 Getting Spring Security

Spring Boot는 Spring Security에 관련한 의존성을 제공하는 `spring-boot-starter-security`스타터를 제공합니다.

**build.gradle**

```groovy
dependencies {
	compile "org.springframework.boot:spring-boot-starter-security"
}
```

Spring Boot는 Maven BOM으로 의존성 버전을 관리하기 때문에, 버전을 생략해도 됩니다. Spring Security 버전을 오버라이드하고 싶다면 아래와 같이 Gradle 속성에 버전을 명시하세요.

**build.gradle**
```groovy
ext['spring-security.version']='5.6.2.RELEASE'
```

## 😀 Features

Spring Security는 인증, 권한부여 및 주요 취약점 공격에 대한 방어를 종합적으로 지원합니다. 다른 라이브러리와의 통합을 제공하여 보다 쉽게 사용할 수 있도록 합니다.
[📁 Features](./features/features.md)

##