# Localization

다른 Locale을 지원해야하는 경우 알아야할 모든 것이 이 섹션에 포함되어 있습니다.

인증 실패나 접근 거부(인가 실패)와 관련된 모든 예외 메시지는 현지화할 수 있습니다. 개발자나 시스템 개발자를 위한 예외나 로깅용 메시지(잘못된 속성, 인터페이스 제약 조건 위반, 올바르지 않은 생성자 사용, startup 시간 검증, 디버깅 수준의 로깅)는 현지화할 수 없습니다. 대신 Spring Security 코드에 영어로 하드코딩 되어있습니다.

`spring-security-core-xx.jar` 파일을 보면 `org.springframework.security` 패키지에 `messages.properties` 파일과 몇몇 공통 언어에 대한 현지화 파일이 순서대로 있는 것을  확인할 수 있습니다. Spring Security에는 스프링의 `MessageSourceAware` 인터페이스를 구현한 클래스가 존재하고, 애플리케이션을 기동할 때 Application Context에 Message Resolver를 주입하기 때문에, `ApplicationContext`에서 메시지를 참조할 수 있습니다.  일반적으로 메시지를 참조하기 위해서는 ApplicationConext 내에 빈을 등록하기만 하면 됩니다. 아래의 예시를 보시면:

```xml
<bean id="messageSource"
	class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
<property name="basename" value="classpath:org/springframework/security/messages"/>
</bean>
```

[message.properties](http://message.properties) 이름은 표준 리소스 번들에 따라 지정되며 Spring Security 메시지가 지원하는 기본 언어로 나타냅니다. 기본 파일은 영어로 되어있습니다.

만약 [messages.properties](http://messages.properties) 파일을 커스텀하거나 다른 언어를 지원하고 싶다면, 파일을 복사하고 적절하게 이름을 변경하고 위의 빈 정의 내용처럼 빈을 등록해야 합니다. 이파일에는 그렇게 많은 양의 메시지 키가 있는 것이 아니기 떄문에 현지화를 주요 이니셔티브로 간주해서는 안됩니다. 이파일의 현지화를 진행해야하는 경우 JIRA 작업을 기록하고 적절한 이름의 현지화 버전의 messages.properties를 첨부하여 커뮤니티와 작업을 공유하는 것을 고려해주세요.

Spring Security는 실제로 적절한 메시지를 조회하기 위해 Spring의 Localization Support에 의존합니다. 이것이 작동하려면 들어오는 요청의 로케일이 Spring의 `org.springframework.context.i18n.LocaleContextHolder`에 저장되어 있는지를 확인해야 합니다. Spring MVC의 `DispatcherServlet`이 이를 자동으로 저장해주지만 Spring Security 필터는 그전에 실행되기 때문에, 필터를 호출하기 전에 `LocaleContextHolder`에 정확한 `Locale`을 설정해야 합니다. 필터에서 직접 설정하거나(`web.xml`에서 Spring Security Filter보다 앞에 위치한 Filter로) 스프링의 `RequestContextFilter`를 사용할 수 있습니다. Spring의 localization을 사용하는 자세한 방법은 Spring Framework를 참고하세요.

“contacts” 샘플 애플리케이션은 현지화된 메시지가 설정되어있습니다.