### Authentication

Spring Security는 Authentication(이하, 인증)에 대한 포괄적인 지원을 제공합니다. 인증은 특정 리소스에 액세스하려는 사람의 신원을 확인하는 방법입니다. 일반적으로는 사용자의 이름 및 비밀번호를 요구합니다. 인증이 수행되면 사용자의 신원을 파악하고 승인 또는 반려 처리를 통해, 사용자의 리소스 액세스에 대한 접근을 관리합니다.

Spring Security는 사용자 인증을 위한 내장 지원을 제공합니다. 본 절에서는 Servlet 및 WebFlux 환경 모두에 적용되는 일반 인증 지원에 대해 소개합니다.

Spring Seucrity의 `PasswordEncoder` 인터페이스는 비밀번호가 안전하게 저장될 수 있도록 암호의 단방향 변환을 수행하는 데 사용합니다. 주어진 `PasswordEncoder` 는 비밀번호를 단방향으로 변환해주며, 변환된 데이터에 대해 복호화할 수 없습니다. 보통 `PasswordEncoder`를 사용하여 저장되는 비밀번호의 암호화된 값은 사용자가 로그인을 통해 인증을 시도할 때, 사용자의 입력된 비밀번호 정보와 암호화된 정보를 비교하여 인증을 처리할 때 사용합니다.

아래의 링크를 통해, Password Storage에 대한 내용을 학습하세요.

[📔 Password Storage](features/SpringSecurity_PasswordStorage.md)

### Protection Against Exploits

Spring Security는 주요 취약점 공격을 보호할 수 있는 기능을 제공합니다. 가능한 모든 곳에서 이 보호기능은 Default로 활성화되어 있습니다.

아래의 링크를 통해, Protection Against Exploits에 대한 내용을 계속 학습하세요.

[📔 Protection Against Exploits](features/SpringSecurity_ProtectionAgainstExploits.md)

### Integrations

Spring Security는 수많은 프레임워크와 API와의 통합을 지원합니다. 이번 섹션에서 우리는 Servlet 또는 Reactive 환경에 국한되지 않고 일반적인 통합에 대해 다룹니다. [Servlet](https://docs.spring.io/spring-security/reference/servlet/integrations/index.html) 또는 [Reactive](https://docs.spring.io/spring-security/reference/servlet/integrations/index.html) 에 관련한 지정된 통합에 관해서는 링크를 참고하세요.

아래의 링크를 통해, Integrations에 대한 내용을 계속 학습하세요.
