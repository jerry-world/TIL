# Spring Boot에서 AWS Parameter Store로 환경설정정보 사용하기

Spring Boot를 사용하여 datasource(DB 커넥션 정보)를 작성해보신 분들은 알겠지만,
기본적으로 Spring Framework에서 제공하는 JDBC Datasource를 작성할 때는, application.yaml(application.properties) 내 정해진 key값에 따라 작성하고 이를 주입하여 DB Connection 객체를 생성합니다.
Datasource 빈을 직접 작성하여 주입할 수도 있겠죠.

그뿐만 아니라, 다양한 인증 정보들을 application.yaml 내에 기재하여 환경 설정 정보를 사용하기도 합니다.

```yaml
spring:
    datasource:
        url: jdbc:mysql://localhost/~/db
        username: username
        password: P@ssw0rd!
        driver-class-name: org.h2.Driver
```

그런데 사실 우리는 이처럼 application.yml에 민감한 패스워드 정보나 노출되어서는 안되는 secret 값들을 소스 내에서 관리한다는 것 자체가 보안상 얼마나 위협이 되는지를 인지해야 합니다.

KISA 소프트웨어 개발보안 가이드(2021)를 보면 6.하드코드된 중요정보 에서는 이렇게 이야기 한다.
> - 프로그램 코드 내부에 하드코드된 패스워드 또는 암호화키를 포함하여 내부 인증에 사용하거나 암호화를 수행하면 중요정보가 유출도리 수 있는 보안 약점이다.
> - 패스워드는 암호화 하여 별도의 파일에 저장하여 사용한다. 또한 중요정보를 암호화하면, 상수가 아닌 암호화 키를 사용하도록 하며 소스코드 내부에 상수형태의 암호화키를 저장해서 사용하지 않도록 한다.

결론은 소스코드 내 비밀번호 또는 중요 정보를 담고있으면 소스코드 유출 시, 중요 정보도 유출될 수 있으니 소스 코드내 저장하지 말라는 의미이다.

기존에는 비밀번호를 담은 파일을 별도로 숨김 파일로 관리하는 등의 방식을 사용했었다. 실제 외부에 있는 저장 매체 내 존재하는 파일을 불러와 암호화된 패스워드를 복호화하여 사용하는 방식이었다.

요즘은 더 현대적인 방법을 사용한다. 대표적으로 Spring Cloud 에서 제공하는 config server를 사용하는 방법이 있다. 이러한 보안 이슈를 고려하는 업체에서
가장 대중적으로 사용하는 방법이라고 생각한다.

또 다른 방법으로는 이번 포스트에서 소개되는 AWS의 Parameter Store라는 것을 사용할 수 있겠다.
AWS Parameter Store는 외부에서 키를 관리할 수 있도록 config server와 유사한 방식으로 기능을 제공한다.

그러면 이 Parameter Store가 무엇인지 AWS의 소개를 한번 들어보자.

