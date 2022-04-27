# 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화

## API 개발 고급 - 지연 로딩과 조회 성능 최적화

### Entity를 그대로 반환하지 말아야 한다.

xToOne 연관관계에서..
- 
- Entity를 노출하게 되면, 반환 Entity내에 존재하는 양방향 관계에 있는 대상들이 서로를 반환하게 되면서 무한 루프가 발생하게 된다.
- 서로 연관관계에 있는 대상들을 그러면 `@JsonIgnore` 애너테이션을 작성하여, Jackson이 이를 반환하지 않도록 처리하면 되지 않는가?
    - Hibernate는 연관관계에 있는 대상을 LAZY LOADING으로 지연하여 로딩하기 위해 Proxy 객체를 만들어두는데, 이 Proxy 객체를 반환할 수 있는 방법이 없어 에러가 발생하게 된다.
    - ⇒ 이 문제를 해결하기 위해, `Hibernate5Module`이 이를 반환할 수 있도록 도와주기는 한다.

    ```groovy
    implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
    ```

    - ⇒ 또한, Lazy loading으로 작성된 대상들을 강제로 불러올 수도 있다.

    ```java
    hibernate5Module.configure(Hibernate5Module.Feature.*FORCE_LAZY_LOADING*,true);
    ```

    - 추가로는, 강제로 불러와야하는 대상을 초기화할 수도 있다.

    ```java
    orderA.getMember().getName();
    ```

    - fetch 타입을 EAGER로 만들게 되도 마찬가지로 처리할 수는 있다.
- 하지만, 근본적인 문제는 이렇게 Entity를 반환한다 할지라도, 사용자의 요청과 무관한 데이터(연관관계에 얽힌 데이터들)을 무분별하게 노출하여 보안상의 위험이 될 수 있다.
- 또한, 사용하지 않는 무관한 데이터를 불러오기 위한 불필요한 DB 요청도 발생하게 된다.

