# 01. IoC 컨테이너

IoC 컨테이너는 Spring Framework에서 객체를 관리하는 창고로 정의할 수 있다.

객체를 관리하기는 하는데, IoC 원리에 의해 객체를 관리한다.

여기서 말하는 **IoC** 는 **Inversion of Control**의 약자인데, 한글로 쉽게 풀이하면 제어의 역전이다.

제어의 역전이라는 표현자체가 굉장히 어색한데(알고 있어도 어색해.. ) 아래의 코드를 통해, 제어의 역전을 이해해보자.


Java에서 Non-static한 객체를 정의하려면 new 키워드를 사용하여 인스턴스를 생성해야한다.

아래는 Service를 사용하기 위해, Controller에 Service 인스턴스를 생성하여 Controller 인스턴스를 생성하는 전통적인 방식이다.

생성된 Service 인스턴스를 Controller 인스턴스 생성 시점에 매개로 전달(주입)하여, Controller에서 service 인스턴스에 접근할 수 있도록 했다.

```java
Service s = new ServiceImpl();
Controller c = new ControllerImpl(s);
```

하지만, Spring Framework에서는 필요한 객체(Bean)를 직접 만들고, 필요한 의존성을 자체적으로 주입(`DI;Dependency Injection`)한다.

위의 방식처럼 직접 대상 인스턴스를 생성하고 주입하는 과정이 생략(물론 내부적으로 Spring이 일련의 주입작업을 해주지만)할 수 있다.

```java
@Configuration
public class AppConfig {
    @Bean
    public Service service() {
        return new ServiceImpl();
    }

    @Bean
    public Controller controller() {
        return new Controller(service());
    }
}
```

다시 말해, `실제로 인스턴스를 만들고 필요한 인스턴스를 매개로 전달하거나 관리하는 등의 행동`(제어)들을 직접 수행하지 않고, Spring Framework가 직접 이 역할을 수행하는 것이다.

그래서 이러한 제어를 수행하는 주체가 역전되었다고하여 제어의 역전이라고 부른다.

---

## IoC 컨테이너의 주인공
IoC 컨테이너는 위에서 살펴본 바와 같이, IoC 매커니즘을 통해 객체를 생성하고 이를 관리하며, 객체 상호간의 의존 관계를 관리한다.

`DI를 수행하고, Bean을 정의하고 생명주기를 관리하는 컨테이너` 라고 이해하면 된다.

이 역할의 주인공은 `BeanFactory` 인터페이스 이다.

스프링 빈 관리의 책임 주체는 BeanFactory 인터페이스이고, ApplicationContext의 상위 타입이다.

ApplicationContext는 BeanFactory 외에도, 스프링 애플리케이션의 주요 컴포넌트 인터페이스를 상속받고 있다.

![img.png](img.png)

간단히 짚고 넘어가자면, ApplicationContext는 여러 상위 인터페이스(BeanFactory를 비롯한, MessageSource, ResourceLoader 등)를 확장한 통합 인터페이스이며,

이 인터페이스의 구체 구현체로 ServletWebServerApplicationContext, ReactiveWebServerApplicationContext 등이 제공된다.

### BeanFactory 생성과정

여하튼, ApplicationContext의 상위클래스이자, IoC 컨테이너의 주역인 BeanFactory가 어떻게 생성되는지 알아보자.

BeanFactory는 ApplicationContext가 만들어지는 과정 사이에서 자연스럽게 생성된다.

SpringApplication 클래스가 동작되면서 ApplicationContext를 만들고, 이 ApplicationContext의 `refresh`메서드가 호출되는 시점에 생성된다.

아래는 AbstractApplicationContext(ApplicationContext의 추상클래스)의 refresh 메서드이다.

> **[여기서 잠깐!]**
> 
> 그런데 왜 Spring Application을 처음 시작하는데, initialize나 create, start 등의 초기 접근하는 듯한 메서드가 아니라 refresh일까.
> 이걸 역으로 생각해보면, 동작중에 refresh를 수행하는 경우가 있는 것일까? 로 접근할 수 있다.
> 이미 애플리케이션이 런타임 상황에서 ApplicationContext가 새로고침될 수 있는 경우, 가장 많이 떠오르는 부분이 spring-boot dev-tools 일 것이다.
> 맞다. ApplicationContext는 SpringApplication이 시작될 때 새롭게 만들어지지만, 런타임 상황에서 dev-tools와 같이 JVM 프로세스는 그대로인채
> 클래스를 reload하면서, ApplicationContext의 주요 컴포넌트들을 모두 초기화하는 작업을 수행한다. 

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    this.startupShutdownLock.lock();
    try {
        this.startupShutdownThread = Thread.currentThread();

        StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);
            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);
            beanPostProcess.end();

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (RuntimeException | Error ex ) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            contextRefresh.end();
        }
    }
    finally {
        this.startupShutdownThread = null;
        this.startupShutdownLock.unlock();
    }
}
```

#### prepareRefresh()
이 과정에서는 
1. 초기화 상태임을 나타내는 Flag를 세우고,
2. 표준 서블릿 환경변수에 해당하는 서블릿 컨텍스트, 서블릿설정정보와 같은 환경 프로퍼티를 등록하고
3. `Environment`에 등록된 PropertySources를 검사해서 필수값이 모두 있는지, 문제가 없는지 유효성 검증을 수행하며,
4. 이전 refresh에서 등록되어있던 리스너를 백업하고 다시 초기상태로 되돌린다.(리로드를 수행한 경우)

다시 말해, ApplicationContext가 Refresh할 수 있도록 사전 준비 작업을 수행하는 것이다.(깊은 내용도 중요하지만 따로 다루도록..)

```java
protected void prepareRefresh() {
    // Switch to active.
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);

    if (logger.isDebugEnabled()) {
        if (logger.isTraceEnabled()) {
            logger.trace("Refreshing " + this);
        }
        else {
            logger.debug("Refreshing " + getDisplayName());
        }
    }

    // Initialize any placeholder property sources in the context environment.
    initPropertySources();

    // Validate that all properties marked as required are resolvable:
    // see ConfigurablePropertyResolver#setRequiredProperties
    getEnvironment().validateRequiredProperties();

    // Store pre-refresh ApplicationListeners...
    if (this.earlyApplicationListeners == null) {
        this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
        // Reset local application listeners to pre-refresh state.
        this.applicationListeners.clear();
        this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // Allow for the collection of early ApplicationEvents,
    // to be published once the multicaster is available...
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### obtainFreshBeanFactory()

이 단계에서는 메서드명처럼 신선한 빈 팩토리를 얻는 작업을 수행한다. 여기서 말하는 신선한 빈 팩토리를 얻는다라 함은,
앞서 언급된 이미 존재하는 빈 팩토리를 소멸시키고 새로운 빈 팩토리를 만들어 내는 과정을 말한다.

```java
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
```

obtainFreshBeanFactory 메서드는 빈 팩토리를 초기화하고 빈 팩토리를 반환하는 메서드이다.
```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		return getBeanFactory();
	}
```

`AbstractApplicationContext를` 상속받는 구체구현체인 `AbstractRefreshableWebApplicationContext`가 빈 팩토리가 이미 존재하는지를 검증하고,

`Bean을 소멸`한 뒤 기존에 존재하던 `Bean Factory를 close` 하는 작업을 수행한다.

그 다음 작업으로 DefaultListableBeanFactory 인스턴스를 생성하고 초기 설정 작업 이 멤버필드로 생성한 이 Bean Factory를 주입한다.

이때 초기설정이라는 것은 beanFactory에 SerailizationId를 부여하고, Application startup Metrics을 작성하는 등의 일련 필수 작업들이다.(순환 참조 허용 여부도 여기서 정의함)

그리고 getBeanFactory()를 호출하여, beanFactory를 반환하여 신선한 빈 팩토리를 최종 반환하는 구조로 되어있다.

여기까지 진행되면 BeanFactory(간단히 초기설정된) 인스턴스가 Context에 주입되는 상태가 된다.

#### prepareBeanFactory(beanFactory)
이 단계에서는 빈팩토리를 사용할 수 있는 상태로 만들기 위한 준비단계이다.

1. ApplicationContext 가 가지고 있는 환경 정보는 BeanFactory에게 전달

- 이 세부 단계에서는 ApplicationContext가 갖고있는 ClassLoader를 BeanFactory에 주입해주고,
- SpEL 표현식을 해석할 수 있는 StandardBeanExpressionResolver를 주입해주고,
- `"classpath:"`와 같은 문자열을 Resource 객체로 변환하게 하는 PropertyEditor를 등록
 
하는 작업을 수행한다.
```java
beanFactory.setBeanClassLoader(getClassLoader());
beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
```

2. Application 레벨의 콜백 구조(Aware 계열)들을 BeanFactory에 연결하고, 일반 의존성 주입 대상으로는 제외

BeanPostProcessor로 `ApplicationContextAwareProcessor`를 추가한다.

ApplicationContextAwareProcessor는 `BeanPostProcessor`로, 빈 생성 이후 "Aware"계열 인터페이스를 구현한 객체를 감지해서 적절한 setter를 호출할 수 있도록 해준다.

만약 EnvironmentAware를 구현한 주체가 있다면, BeanPostProcessor에 의해 별도의 @Autowired 없이도 setApplication(XXXAware)가 호출되도록 해준다.

```java
// Configure the bean factory with context callbacks.
beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
beanFactory.ignoreDependencyInterface(ApplicationStartupAware.class);
```

3. 컨테이너 자기 자신을 주입하려하는 대상에게 자동으로 주입할 수 있도록 등록 작업 수행
```java
beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```
특정 Bean에서 BeanFactory, ResourceLoader, ApplicationEventPublisher, ApplicationContext를 의존주입하려할때, Bean으로 검색하지 않고, Context가 가지고있는
대상을 직접 반환해준다.

4. AOP/바이트코드 조작을 쓸 준비
```java
// Detect a LoadTimeWeaver and prepare for weaving, if found.
if (!NativeDetector.inNativeImage() && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
    beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
    // Set a temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
}
```
Load-time Weaving(LTW) 는 클래스가 로딩되는 시점에 바이트 코드를 조작하는 행위를 수행한다.
- AOP, AspectJ LTW
- JPA 엔티티 Instrumentation
- 기타 바이트코드 변환기

이를 초기화하는데, 만약 GraalVM native-image 환경이면 런타임 클래스로딩이나 바이트 코드 변환이 거의 안되거나 제한적이어서 사용이 불가능하다.

때문에 native-image환경이 아니라면, LoadTimeWeaverAwareProcessor를 등록해서, LoadTimeWeaverAware 인터페이스를 구현한 빈에

LoadTimeWeaver를 주입해주는 BeanPostProcessor를 정의한다.

templateClassLoader 는 클래스를 로딩할 때 바이트코드 변경이 될 수 있으니깐, 타입 매칭 할때는 안전하게 'LTW-aware'한 클래스로더를 쓰도록 정의하는 것이다.

AspectJ LTW나 기타 바이트코드 조작으로 인해 클래스 구조가 달라질 수 있으니, BeanFactory가 타입 매칭을 할 때도 이런 변형을 반영할 수 있도록 ContextTypeMatchClassLoader를 tempClassLoader로 설정한다.

5. 기본환경 Bean 등록
```java
// Register default environment beans.
if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
    beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
}
if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
    beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
}
if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
    beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
}
if (!beanFactory.containsLocalBean(APPLICATION_STARTUP_BEAN_NAME)) {
    beanFactory.registerSingleton(APPLICATION_STARTUP_BEAN_NAME, getApplicationStartup());
}
```
스프링이 기본적으로 제공하는 환경 관련 객체(Environment, SystemProperties, SystemEnvironment)를 BeanFactory에 Bean으로 등록한다.

여기서 containsLocalBean 조건이 있냐는 것은, 이미 사용자가 같은 이름으로 Bean을 등록했을 수도 있기 때문에, 그렇지 않은 경우에 대해서만 싱글턴으로 등록한다.

만약 있음에도 불구하고 Bean을 등록한다면, BeanDefinition 과정에 충돌이 발생한다.(Bean 충돌)

#### postProcessBeanFactory(beanFactory)

postProcessBeanFactory는 구체 클래스인 서브클래스가 오버라이드해서 할일을 수행하는 템플릿 메서드이다. 실행 실행 주체는
- AbstractRefreshableWebApplicationContext
- AnnotationConfigServletWebServerApplicationContext
- GenericWebApplicationContext
등이다.

각 서브클래스에서 하는 역할이 다르긴하다. 이 시점에 컨텍스트 별 주체성을 갖는 확장이 이루어지는 거라고 볼 수 있다.

가령, AbstractRefreshableWebApplicationContext에서는 웹용 BeanPostProcessor가 추가되거나, WebApplication 스코프, 서블릿 컨텍스트의 환경 설정 빈들이 추가도니다.

각 구체 컨텍스트들 마다 공통적으로 BeanFactory의 추가설정을 수행한다고 보면된다.

#### invokeBeanFactoryPostProcessors(beanFactory)

이 단계에서는 BeanFactory에 등록된 BeanFactoryPostProcessor 타입의 Bean들을 찾아서 호출(실행)하는 단계이다.
```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

좀더 깊게 들어가면 어떻게 @Configuration, @ComponentScan, @PropertySource 같은 애들이 실제 BeanDefinition으로 등록되는거지? 라는 의문도 해결되는 구간이다.

`refresh()` 흐름 중 일부를 다시 확인해보면
```java
prepareBeanFactory(beanFactory);
postProcessBeanFactory(beanFactory);

// 이제부터 BeanFactoryPostProcessor 실행
invokeBeanFactoryPostProcessors(beanFactory);
registerBeanPostProcessors(beanFactory);
```
이런 과정들을 수행하는데,

1) prepareBeanFactory -> 기본설정 단계이고
2) postProcessBeanFactory -> 서브클래스 별 확장 단계이고
3) invokeBeanFactoryPostProcessors -> BeanDefinition 레벨 후처리를 시작하는 단계이다.

즉, 이시점이 BeanDefinition(Bean 정의)을 조작하는 단계이다.

> BeanPostProcessor랑 이름이 비슷해서그렇지, BeanFactoryPostProcessor는 서로 다른 주체이다.
> BeanFactoryPostProcessor → Bean 인스턴스가 만들어지기 이전, BeanDefinition(설계도)을 조작하는 후처리기
> BeanPostProcessor → Bean 인스턴스가 만들어진 이후, 실제 객체를 가공하거나 프록시를 생성하는 후처리기

BeanFactoryPostProcessor는 Bean 인스턴스가 생성되기 전에, BeanDefinition을 수정할 수 있는 후처리기라고 볼 수 있다.(빈이 인스턴스로 만들어지기 전에, BeanDefinition 단계에서 속성을 조작할 수도 있음)

그래서, InvokeBeanFactoryPostProcessors가 실제로 하는 작업은

- BeanFactory에 등록된 모든 BeanFactoryPostProcessor를 찾는다.(ApplicaitonContext에 의해 등록된 Processor, BeanDefinition으로 정의된 Processor를 모두 포함)
- 우선순위에 따라 실행 순서를 정렬한다(Order, ProirityOrdered, Ordered 애너테이션을 참조함)
- Processor의 postProcessBeanFactory(beanFactory)를 호출하여 BeanDefinition 조작
- ConfigurationClassPostProcessor가 @Configuration, @ComponentScan, @Import 등 메타정보를 파싱, 새로운 BeanDefinition을 등록

조금더 쉽게 설명하자면, Configuration, Bean 등의 스프링 리소스를 모두 스캔해서 BeanDefinition을 등록하는 단계이다(아직 인스턴스가 생성된 단계는 아님)