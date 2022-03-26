# Concurrency Support

대부분 환경들에서, 보안은 Thread 별로 저장됩니다. 그 말인즉슨 새 Thread에서 작업이 끝나면 SecurityContext는 사라집니다. Spring Security는 사용자에게 보다 쉽게 이를 다룰 수 있도록 도움을 주는 몇몇의 인프라스트럭처를 제공합니다. Spring Security는 멀티 스레드 환경에서 Spring Security로 작업할 수 있도록 저수준의 추상화를 제공합니다. 실제로, 이는 Spring Security가 [AsyncContext.start(Runnable)](https://docs.spring.io/spring-security/reference/servlet/integrations/servlet-api.html#servletapi-start-runnable) 및 [Spring MVC Async](https://docs.spring.io/spring-security/reference/servlet/integrations/mvc.html#mvc-async) 통합을 기반으로 구축한 것입니다.

## 😀 DelegatingSecurityContextRunnable

Spring Security에서 동시성 지원하는 것들 중 하나는 `DelegatingSecurityContextRunnable` 입니다. 이는 대리 수행자을 위해 지정된 `SecurityContext` 로 `SecurityContextHolder`를 초기화 하기 위해 delegate `Runnable`을 Wrapping합니다. 그런 다음 delegate Runnable 을 호출하여 나중에 `SecurityContextHolder`를 지우도록 합니다.

`DelegatingSecurityContextRunnable`은 아래과 같습니다:

```java
public void run() {
	try{
		SecurityContextHolder.setContext(securityContext);
		delegate.run()
	} finally {
		SecurityContextHolder.clearContext();
	}
}
```

매우 간단해보이지만 이렇게 함으로써 한 Thread에서 또다른 Thread로 원활하게 SecurityContext를 넘길 수 있습니다. 이것은 대부분의 경우 SecurityContextHolder가 Thread 별로 동작하기 때문에 매우 중요합니다. 예를 들어, Spring Securirty의 [<global-method-security>](https://docs.spring.io/spring-security/reference/servlet/appendix/namespace/method-security.html#nsa-global-method-security) 지원을 사용하여 당신의 서비스 중 하나를 보호했을 수도 있습니다. 이제 보안 서비스를 호출하는 최근 `Thread`의 `SecurityContext`를 다른 `Thread`로 안전하게 넘길 수 있습니다.

다음 예시는 어떻게 위의 내용처럼 수행하는지를 나타냅니다:

```java
Runnable originalRunnable = new Runnable() {
	public void run() {
		//invoke secured service
	}
};

SecurityContext context = SeucrityContextHolder.getContext();
DelegatingSecurityContextRunnable wrappedRunnable
	= new DelegatingSeucrityContextRunnable(originalRunnable, context);

new Thread(wrappedRunnable).start();
```

위의 코드는 다음 단계를 수행합니다:

- 보안 서비스를 호출할 `Runnable` 을 생성합니다. Spring Security를 인식하지 못한다는 점을 유의해야 합니다.
- `SecurityContextHolder`로부터 우리가 사용하고자 하는 `SecurityContext`를  얻고 `DelegatingSecurityContextRunnable`을 초기화합니다.
- Thread를 생성하기 위해, `DelegatingSecurityContextRunnable`을 사용합니다.
- 우리가 생성한 Thread를 시작합니다.

위처럼 `SecurityContextHolder`에서 `SecurityContext`를 사용하여 `DelegatingSecurityContextRunnable`은 일반적이기 때문에 이를 쉽게 사용할 수 있도록 shortcut 생성자가 존재합니다. 다음 코드는 위의 코드와 동일합니다:

```java
Runnable originalRunnable = new Runnable() {
public void run() {
	// invoke secured service
}
};

DelegatingSecurityContextRunnable wrappedRunnable =
	new DelegatingSecurityContextRunnable(originalRunnable);

new Thread(wrappedRunnable).start();
```

위의 코드는 간단하지만 여전히 Spring Security를 사용하고 있다는 지식이 필요합니다. 다음 섹션에서는 `DelegatingSecurityContextExecutor`를 사용하여 Spring Security를 사용하고 있다는 사실을 숨기는 방법을 살펴보겠습니다.

## 😀 DelegatingSecurityContextExecutor

이전 섹션에서는 `DelegatingSecurityContextRunnable` 을 사용하는 쉬운 방법을 알아보았습니다. 하지만, 이를 사용하기 위해서는 Spring Security를 알아야한다는 전제가 있기 때문에 이상적이지 않습니다. 그럼 이번에는 어떻게 `DelegatingSecurityContextExecutor`로 Spring Security 사용에 대한 사전지식 없이 사용할 수 있는지 알아보겠습니다.

`DelegatingSecurityContextExecutor` 는 Runnable 대신 Executor를 사용할 뿐, 구조는 `DelegatingSecurityContextRunnable`과 상당히 유사합니다. 아래의 예시를 통해 사용방법을 확인해보겠습니다:

```jsx
SecurityContext context = SecurityContextHolder.createEmptyContext();
Authentication authentication = 
	new UsernamePasswordAuthenticationToken("user","doesnotmatter", AuthorityUtils.createAuthorityList("ROLE_USER"));
context.setAuthentication(authentication);

SimpleAsyncTaskExecutor delegateExecutor = new SimpleAsyncTaskExecutor();
DelegatingSecurityContextExecutor executor =
	new DelegatingSecurityContextExecutor(delegateExecutor, context);

Runnable originalRunnable = new Runnable() {
	public void run() {
		// invoke secured service
	}
};

executor.execute(originalRunnable);
```

코드는 다음 단계를 수행합니다.

- DelegatingSecurityContextExecutor에 사용될 SecurityContext를 생성합니다. 이 예시에서는 간단히 손으로 SecurityContext를 생성합니다. 그러나, 어디에서 어떻게 SecurityContext를 얻는지는 중요하지 않습니다.(즉, 원하는 경우에 SecurityContextwHolder를 얻을 수 있음)
- 제출된 `Runnable`s 실행을 담당하는 delegateExecutor를 생성합니다.
- 마지막으로 `DelegatingSecurityContextRunnable`을 사용하여 실행 메서드로 전달되는 모든 Runnable을 래핑하는 역할을 수행하는 `DelegatingSecurityContextExecutor`를 만듭니다. 그다음 래핑된 Runnable을 `delegateExecutor`에게 전달합니다. 이 경우 `DelegatingSecurityContextExecutor`에 제출된 모든 `Runnable`에 대해 동일한 `SecurityContext`가 사용됩니다. 이것은 상승된 권한을 가진 사용자가 실행해야 하는 백그라운드 작업을 실행할 때 유용합니다.
- 이 시점에서 스스로 “이것이 내 코드에서 Spring Security의 지식으로부터 어떻게 보호되는거지?” 라는 질문을 갖을 수 있습니다. 우리는 코드에서 SecurityContext 및 DelegatingSecurityContextExecutor를 만드는 대신 DelegatingSecurityContextExecutor의 이미 초기화 된 인스턴스를 주입할 수 있습니다.

```java
@Autowired
private Executor executor; // becomes an instance of our DelegatingSecurityContextExecutor

	public void submitRunnable() {
	Runnable originalRunnable = new Runnable() {
		public void run() {
		// invoke secured service
		}
	};
	executor.execute(originalRunnable);
}
```

이제 우리 코드는 `SecurityContext`가 `Thread`로 전파되고, `originalRunnable`이 실행되고 `SecurityContexetHolder`가 지워지는 것을 인지하지 못합니다. 위의 예에서, 동일 사용자가 각 스레드를 실행하는데 사용되어집니다. `originalRunnable`을 처리하기 위해 `executor.execute(Runnable)` (즉, 현재 로그인한 사용자)를 호출할 때 `SecurityContextHolder`의 사용자를 사용하려면 어떻게 해야할까요? 이것은 `DelegatingSecurityContextExecutor` 생성자에서 `SecurityContext` 인수를 제거하여 수행할 수 있습니다.

예를들어:

```java
SimpleAsyncTaskExecutor delegateExecutor = new SimpleAsyncTaskExecutor();
DelegatingSecurityContextExecutor executor =
	new DelegatingSecurityContextExecutor(delegateExecutor);
```

이제 `executor.execute(Runnable)`이 실행될 때마다 `SecurityContext`는 먼저 `SecurityContextHolder`에 의해 획득되어지고 그 `SecurityContext`는 `DelegatingSecurityContextRunnable`을 생성하는데에 사용됩니다. 그 말인즉슨 `executor.execute(Runnable)` 코드를 호출하는데 사용된 동일한 사용자로 `Runnable`을 실행하고 있음을 의미합니다.

## 😀 Spring Security Concurrency Classes

Java 동시성 API 및 Spring Task 추상화 모두와의 통합에 대해서는 Javadoc을 참조하세요. 이전 코드를 이해했다면 아래의 것을 이해하기 굉장히 쉬울 것입니다.

- `DelegatingSecurityContextCallable`
- `DelegatingSecurityContextExecutor`
- `DelegatingSecurityContextExecutorService`
- `DelegatingSecurityContextRunnable`
- `DelegatingSecurityContextScheduledExecutorService`
- `DelegatingSecurityContextSchedulingTaskExecutor`
- `DelegatingSecurityContextAsyncTaskExecutor`
- `DelegatingSecurityContextTaskExecutor`
- `DelegatingSecurityContextTaskScheduler`