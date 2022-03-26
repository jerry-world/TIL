# Spring Data Integration

Spring Security는 쿼리 내에서 현재 사용자를 참조할 수 있는 Spring Data 통합을 제공합니다. 페이징 결된 결과를 이후 확장할 경우 사용자를 포함하지 않을 경우 확장이 불가하기 떄문에 이는 유용할 뿐더러 필요한 항목입니다.

## 😀 Spring Data & Spring Security Configuration

이 지원 항목을 사용하기 위해, `org.springframework.security:spring-security-data` 의존성을 추가하고 `SecurityEvaluationContextExtension` 타입의 bean을 주입합니다.

```java
@bean
public SecurityEvaluationContextExtension securityEvaluationContextExtension(){
	return new SecurityEvaluationContextExtension();
}
```

## 😀 Security Expressions within @Query

이제 아래의 예제를 통해 Query에서 Spring Security를 사용하는 것을 살펴봅시다:

```java
@Repository
public interface MessageRepository extends PagingAndSortingRepository<Message,Long> {
	@Query("select m from Message m where m.to.id = ?#{ principal?.id }")
	Page<Message> findInbox(Pageable pageable);
}
```

위의 쿼리는 `Authentication.getPrincipal().getId()`가 `메시지` 의 수신자와 같은지를 확인합니다. 이 예에서는 id 속성이 있는 개체가 되도록 보안 주체를 사용자 지정했다고 가정합니다. `SecurityEvaluationContextExtension` 빈을 노출함으로써 모든 [공통 보안 표현식](https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#common-expressions) 을 쿼리 내에서 사용할 수 있습니다.