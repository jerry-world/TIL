# Spring Boot에서 AWS Parameter Store로 환경설정정보 사용하기

## 왜 소스 내 패스워드를 하드코딩하면 안될까?
기본적으로 Spring Framework에서 제공하는 JDBC Datasource를 작성할 때는, application.yaml(application.properties) 내 정해진 key값에 따라 작성하고 이를 주입하여 DB Connection 객체를 생성한다.
Datasource 빈을 직접 작성하여 주입할 수도 있다. 다들 한번쯤 설정해봤을 것이다.

그뿐만 아니라, 다양한 인증 정보들을 application.yaml 내에 기재하여 환경 설정 정보를 사용한다.

```yaml
spring:
    datasource:
        url: jdbc:mysql://localhost/~/db
        username: username
        password: P@ssw0rd!
        driver-class-name: org.h2.Driver
```

그런데 사실 우리는 이처럼 application.yml에 민감한 패스워드 정보나 노출되어서는 안되는 secret 값들을 소스 내에서 관리한다는 것 자체가 보안상 얼마나 위협이 되는지를 인지해야 한다.

KISA 소프트웨어 개발보안 가이드(2021)를 보면 6.하드코드된 중요정보 에서는 이렇게 이야기 한다.
> - 프로그램 코드 내부에 하드코드된 패스워드 또는 암호화키를 포함하여 내부 인증에 사용하거나 암호화를 수행하면 중요정보가 유출도리 수 있는 보안 약점이다.
> - 패스워드는 암호화 하여 별도의 파일에 저장하여 사용한다. 또한 중요정보를 암호화하면, 상수가 아닌 암호화 키를 사용하도록 하며 소스코드 내부에 상수형태의 암호화키를 저장해서 사용하지 않도록 한다.

결론은 소스코드 내 비밀번호 또는 중요 정보를 담고있으면 소스코드 유출 시, 중요 정보도 유출될 수 있으니 소스 코드내 저장하지 말라는 의미이다.

기존에는 비밀번호를 담은 파일을 별도로 숨김 파일로 관리하는 등의 방식을 사용했었다. 실제 외부에 있는 저장 매체 내 존재하는 파일을 불러와 암호화된 패스워드를 복호화하여 사용하는 방식이었다.

요즘은 더 현대적인 방법을 사용한다. 대표적으로 Spring Cloud 에서 제공하는 config server를 사용하는 방법이 있다. 이러한 보안 이슈를 고려하는 업체에서
가장 대중적으로 사용하는 방법이라고 생각한다.

또 다른 방법으로는 이번 포스트에서 소개되는 AWS의 Parameter Store라는 것을 사용할 수 있겠다.
AWS Parameter Store는 외부에서 키를 관리할 수 있도록 config server와 유사한 방식으로 기능을 제공한다.

그러면 이 Parameter Store가 무엇인지 알아보자.

## AWS Parameter Store란?
![img.png](img.png)
Parameter Store는 AWS Systems Manager 서비스에 포함되는 서비스이다. 작업 관리, 애플리케이션 관리, 변경 관리, 노드 관리 중 애플리케이션 관리 서비스에 속한다.

이 파라미터 스토어는 구성 데이터 관리 및 secret 관리를 위한 안전한 계층적 스토리지를 제공한다고 소개한다. 계층적 스토리지라는 표현은 파라미터 스토어를 다루다보면 이해하겠지만,

이 파라미터 스토어의 이름이 계층 구조 형태를 띈다. 이를테면, 박선용 사용자의 a라는 서비스 내 사과, 바나나, 파인애플 이라는 파라미터 이름으로 특정 값을 저장한다면,

- /박선용/a/사과
- /박선용/a/바나나
- /박선용/a/파인애플

과 같은 형태로 파라미터 이름을 작성한다. 이러한 계층형 구조로 파라미터를 저장하는 스토리지 구조를 가지고 있다.

또한 특정 파라미터에 저장할 값을 일반 텍스트 또는 암호화된 데이터로 저장할 수 있다.
사과라는 파라미터에 저장되는 데이터가 "는 맛있다" 라는 일반 텍스트 형태로도 저장할 수 있지만, 암호화할 만큼 중요한 데이터라면 이를 AWS KMS를 사용하여, 암호화하고 저장할 수 있다.

부가적으로, 특정 파라미터의 값이 변경되면 변경을 감지하여 이를 알림할 수 있도록 AWS EventBridge와도 연동할 수 있다. 다룰 수 있는 범위가 상당히 포괄적이다.

이번 포스팅에서 다룰 핵심 주제는 SpringBoot내 application.yml에 기재한 하드코딩된 패스워드를 AWS의 Parameter Store로 변경하는 것이다.

## AWS Parameter Store를 사용하여, Spring Boot의 환경설정 정보 다루기

### AWS Parameter Store에서 파라미터 생성
제일 먼저할 일은 어떠한 데이터를 외부에서 다룰 것인가를 결정해야한다.
일반적인 URL이나, 단순 설정과 같은 정보들보다 노출될 경우 민감한 정보가 될 수 있는 것들을 주로 다룬다.

보편적으로 패스워드나 secret 값, 인증정보 등이 이에 해당한다.

대표적으로 많이 사용하는 `spring.datasource.password`가 대표적이지 않을까 싶다.
이 포스팅에서는 `spring.datasource.password`의 값을 Parameter Store로 관리하고 이를 애플리케이션 구동시점에 정상적으로 읽어, DB커넥션이 정상적으로 이루어지는지를 확인하겠다.

먼저 AWS의 Parameter Store 서비스로 이동하자.

우측 상단에서 파라미터 생성버튼을 클릭하여 이동하자.

![img_1.png](img_1.png)

파라미터 세부 정보를 입력하면 아주 간단히 파라미터를 생성할 수 있다.

이름은 계층형태 즉, 경로 형태로 입력한다. 사실 이 계층형을 사용하면, 프로젝트 정책에 따라 효율적으로 관리할 수 있다.
이를테면, 내 프로젝트에서는 local,stage, live 프로파일별로 환경설정을 구성하고, 각각 서로 다른 데이터베이스를 쓴다고 가정해보겠다.
그리고 여러 데이터베이스를 써야해서, datasource를 여러개를 작성해야한다고 상황을 가정해보겠다.

나는 local, stage, live 마다 서로 다른 A와 B 데이터베이스를 관리해야 한다.
- local - A DB
- local - B DB
- stage - A DB
- stage - B DB
- live - A DB
- live - B DB

나는 이 형태를 계층적으로 관리하기 위해서 이런 형태를 만들 예정이다.
- /local/A/datasource_password
- /local/B/datasource_password
- /stage/A/datasource_password
- /stage/B/datasource_password
- /live/A/datasource_password
- /live/B/datasource_password

필요에 따라서는 `보안 문자열`을 선택하여 KSM 기반으로 값을 암호화하여 관리할 수도 있다.

![img_3.png](img_3.png)

파라미터 생성이 완료되었다.

![img_4.png](img_4.png)

다음 나머지 5개도 이와같은 방식으로 만들 것이다.

어라? 만들려고 보니 계층에 대해 자동완성 되고 있다.
![img_5.png](img_5.png)

아주 좋은 기능이다.

다음은 Spring Boot에서 이렇게 만든 파라미터를 읽어와 사용할 수 있도록 처리해보자.

### Spring Boot 설정하기

(https://spring.io/projects/spring-cloud-aws 에서 컨트롤 가능하지만, 이 포스팅에서는 com.coveo를 사용하여 테스트한다.)

먼저, 해당 깃헙(https://github.com/coveooss/spring-boot-parameter-store-integration)을 참고하여 의존성을 주입한다.
필자는 포스팅 작성일 기준 최신버전인 1.5.0 버전을 사용하였다. 스프링 부트 버전에 따라 라이브러리 동작 여부가 확실하지 않으니 이를 주의해야한다.

```groovy
//...build.gradle

dependencies {
  implementation 'com.coveo:spring-boot-parameter-store-integration:1.5.0'
}
```

파라미터 스토어를 사용하기 위해서는, 환경변수 내 parameter store 사용 여부를 정의해야한다.
```yaml
#...application.yml

awsParameterStorePropertySource:
  enabled: true
```

해당 Github 내 README를 읽어보면 이 라이브러리는 AWS Java SDK를 사용하고 있다.
AWS Java SDK에서는 외부에서 AWS 자격 증명을 획득하기 위해, 몇가지의 자격 증명 취득 방법에 대해서 제시한다.

이말인 즉슨, 내 계정의 parameter store에 접근하기 위해, SDK를 사용하여 Credential 을 작성하고 이를 통해 나의 Parameter Store 에 접근하는 것으로 이해하면 된다.

AWS SDK는 자격 증명 정보를 작성하는 방법 몇가지를 제안한다.
- 시스템 속성 내 특정 속성에 기재
- 환경 변수 내 특정 변수에 기재
- Amazon ECS 컨테이너 자격 증명
- 인스턴스 프로파일 자격 증명
- 파일로 관리

이 중 `파일로 관리`하는 방법으로 포스팅을 진행한다.

기본적으로 AWS SDK는 홈 디렉터리 내 .aws 디렉터리가 존재하는지 확인하고, 사전 지정된 변수에 값이 존재할 경우 이를 자격 증명 정보로 활용한다.

Windows 및 Linux 무관하다. Windows 는 C:\user\사용자명\ 디렉터리를 사용하면 되고, Linux는 /home/사용자명/ 디렉터리를 사용하면 된다.

```shell
cd ~
mkdir -p .aws
touch .aws/credentials
touch .aws/config
```

`credentials` 파일엔 자격증명 정보를 저장하고, `config` 파일엔 region 정보 등 설정 정보를 저장한다.

```shell
# .aws/credentials
[default]
aws_access_key={ACCESS KEY}
aws_secret_access_key={aws_secret_access_key}
```
```shell
[default]
region=northeast-2
output=json
```

이러면 자격증명 부분도 끝이납니다.

이제는 윗단원에서 수행한 파라미터를 실제 application.yml에서 불러올 수 있도록 작성만 하면 된다.
방법은 상당히 심플하다.
단순히 파라미터 스토어의 이름만 넣으면 된다.

```yaml
#application.yml

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/a
    username: user
    password: ${/local/A/datasource-password}
```

이제 애플리케이션을 구동하면 parameter store 적용 끝이다!