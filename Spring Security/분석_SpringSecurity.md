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
ext['spring-security.version']='5.3.2.RELEASE'
```

## 😀 Features

Spring Security는 인증, 권한부여 및 주요 취약점 공격에 대한 방어를 종합적으로 지원합니다. 다른 라이브러리와의 통합을 제공하여 보다 쉽게 사용할 수 있도록 합니다.

### Authentication

Spring Security는 Authentication(이하, 인증)에 대한 포괄적인 지원을 제공합니다. 인증은 특정 리소스에 액세스하려는 사람의 신원을 확인하는 방법입니다. 일반적으로는 사용자의 이름 및 비밀번호를 요구합니다. 인증이 수행되면 사용자의 신원을 파악하고 승인 또는 반려 처리를 통해, 사용자의 리소스 액세스에 대한 접근을 관리합니다.

Spring Security는 사용자 인증을 위한 내장 지원을 제공합니다. 본 절에서는 Servlet 및 WebFlux 환경 모두에 적용되는 일반 인증 지원에 대해 소개합니다.

Spring Seucrity의 `PasswordEncoder` 인터페이스는 비밀번호가 안전하게 저장될 수 있도록 암호의 단방향 변환ㅇ르 수행하는 데 사용합니다. 주어진 `PasswordEncoder` 는 비밀번호를 단방향으로 변환해주며, 변환된 데이터에 대해 복호화할 수 없습니다. 보통 `PasswordEncoder`를 사용하여 저장되는 비밀번호의 암호화된 값은 사용자가 로그인을 통해 인증을 시도할 때, 사용자의 입력된 비밀번호 정보와 암호화된 정보를 비교하여 인증을 처리할 때 사용합니다.

아래의 링크를 통해, Password Storage에 대한 내용을 학습하세요.

[📔 Password Storage](./detail/SpringSecurity_PasswordStorage.md)