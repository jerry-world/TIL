# CORS

## CORS는 무엇인가?
CORS는 Cross-Origin-Resource-Sharing 의 약어로 교차 출처 리소스 공유라고 부른다.
단어가 주는 의미 자체가 난해하긴 하지만, 굉장히 직관적인 단어이다. 교차되는 출처 간 리소스를 공유한다. 라고 받아들이면 되는데,
여기서 말하는 교차되는 출처라함은, 서로 다른 도메인을 가진 이라는 의미이고, 리소스 공유라 함은 상호간 HTTP 요청 및 응답 정도로 받아들이면 된다.

Mozilla.org 에서는 CORS에 대해 아래와같이 설명하고 있다. 

>CORS는 HTTP 헤더 기반 메커니즘으로, 브라우저가 리소스 로드를 허용해야 하는 자체 출처(내부 서비스) 이외의
모든 출처(도메인, 스키마 또는 포트)를 서버가 나타낼 수 있도록 한다. CORS는 또한 서버가 실제 요청을 허용하는지 확인하기 위해,
브라우저가 출처 간 리소스를 호스팅하는 서버에 "preflighted" 요청을 하는 메커니즘에 의존한다. 해당 "preflighted"에서
브라우저는 HTTP 메서드를 나타내는 헤더와 실제 요청에 사용될 헤더를 전송한다.

## 왜 CORS를 사용하는가?
과거의 웹 애플리케이션에서는 FE 및 BE의 분리 형태보단 단일 애플리케이션인 경우가 대부분이었다. 하지만, 웹의 발전에 따라 현대에 들어서면서,
기존 브라우저가 가지고 있던 보안 정책만으로는 이를 해소할 수 없게 되었다.

익명의 FE 서비스에서 악의적으로 나의 BE로 요청을 시도했을 때, 이를 근본적으로 차단할 수 있어야하고, 때에 따라서는 그 누군가의 대상을 허용해야한다.

단순히 정적인 웹 페이지로 구성된 웹 애플리케이션에서 이러한 서비스 구분과 다양한 형태의 구성은
상호간 출처에 따른 요청을 허용할지 말지에 대한 요구사항이 필요하게 되었고,

이에 따라, 웹은 동일 출처 정책(Same-Origin Policy)라는 보안 매커니즘이 나오게되었다.
이 동일 출처 정책은 웹 브라우저의 중요한 요소중 하나로, 동일 출처라 함은 프로토콜, 도메인, 포트가 모두 동일한 경우를 
동일한 출처라고 간주한다.

그런데 동일 출처 정책에는 아주 큰 한계가 있다. 실제로 서드 파티 애플리케이션이나 다른 출처를 가진 애플리케이션과의
상호작용이 필요할때가 그렇다.

예를 들어, A 회사의 애플리케이션을 나의 애플리케이션에 적용해서 A회사의 요청을 받아들여야할 필요가 있을 때,
이 동일 출처 정책은 서로 다른 출처를 갖고있기 때문에 요청을 허용하지 않는다.

떄문에, 이의 한계를 극복하기 위해서 CORS(Cross-Origin Resource Sharing)이라는 웹 보안 매커니즘이 나오게 되었다.
이 CORS 는 2004년에 처음 제안되었으며, 2014년에 표준 사양으로 채택되었다.

## 어떻게 동작하는가?
CORS의 동작방식은 꽤 간단하지만 많이들 어려워하는 부분이기도 하다.

CORS는 웹 브라우저와 서버 간의 통신에서 시작된다. 웹 애플리케이션이 다른 리소스에 접근할 때, CORS는 웹 브라우저가 서버에 요청을 보내기 전에
추가적인 보안 검사를 수행한다. 이 보안 검사는 HTTP 헤더를 통해 수행된다.

우리는 이 보안 검사를 **Preflight Request** 사전 요청이라고 부른다.

HTTP Method 중에 OPTIONS라는 Method을 본적이 있을 것이다. OPTIONS 방식은 주로 사전 요청을 수행할 때 사용된다.

여하튼 우리는 사전 요청을 통해, CORS가 허용되어있는지 확인을 하고 웹 브라우저가 이를 통해 검사가 완료되었다면 실제 요청의 전송 여부를 결정한다.

## Prefilght Request는 어떻게 생겼는가
사전 요청의 형태는 아래와 같다.

```
OPTIONS /resource HTTP/1.1
Host: www.smalldogg.com
Origin: http://www.your-web-app.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type, authorization
```

사전 요청은 항상 OPTIONS 방식으로 수행되며, HTTP 헤더에 Host 정보와 Origin, Access Control Request Method, Access Control Request Header를 통해,
사전 검증 요청을 수행한다.

결국엔 HTTP 헤더 정보를 통해서 사전 검증을 한다고 보면된다.

- Host 헤더 : 요청이 보내지는 서버의 호스트를 지칭한다.
- Origin 헤더 : 웹 애플리케이션이 요청을 보내는 출처를 지정한다.
- Access-Control-Request-Method 헤더 : 웹 애플리케이션이 실제로 보내고자하는 요청의 HTTP 메서드를 지정한다.
- Access-Control-Request-Header 헤더 : 웹 애플리케이션이 실제로 보내고자 하는 요청에 포함할 특정 헤더들을 지정한다.

## Spring Security 6를 사용하여 CORS를 적용해보자
> 이전 버전의 Spring Security와는 상이할 수 있음

```java
@Configuration
public class SomeClass{

    @Bean
    CorsConfigurationSource corsConfiguration() {
        CorsConfiguration configuration = new CorsConfiguration();
        
        //출처의 형태를 작성하여, 형태에 일치하는 대상들을 허용할 수 있다.
        configuration.setAllowedOriginPatterns(Arrays.asList("*"));
        //CORS 요청에 대해 허용할 HTTP 메서드를 정의할 수 있다.
        configuration.setAllowedMethods(Arrays.asList("HEAD","POST","GET","DELETE","PUT"));
        //허용할 헤더를 제한할 수 있다.
        configuration.setAllowedHeaders(Arrays.asList("*"));
        //자격증명과 함께 요청을 해야하는지에 대한 여부를 작성한다. 
        configuration.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        //접근 가능한 리소스를 URL 기반으로 처리할 수 있다.
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```