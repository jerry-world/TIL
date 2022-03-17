# HttpHeadersFilters

- 목차

Http Headers Filter는 NettyRoutingFilter와 같이, 다운스트림에 전송하기 이전에 적용되는 Filter입니다.

## 😀 Forwarded Headers Filter

Fowareded Headers Filter는 다운스트림 서비슬 보낼 Forwarded 헤더를 생성합니다. 기존 Forwarded 헤더에 현재 요청의 Host 헤더, Scheme, Port를 추가합니다.

## 😀 RemoveHopByHop Headers Filter

RemoveHopByHop Headers Filter는 Forwarded 요청에서 헤더를 제거합니다. 제거할 기본 헤더 목록은 IETF에 기반합니다.

[draft-ietf-httpbis-p1-messaging-14](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-p1-messaging-14#section-7.1.3)

제거되는 헤더의 목록은 아래와 같습니다.

- Connection
- Keep-Alive
- Proxy-Authentication
- Proxy-Authorization
- TE
- Trailer
- Transfer-Encoding
- Upgrade

Application.yaml에 spring.cloud.gateway.filter.remove-hop-by-hop.headers 프로퍼티에 제거할 헤더명의 리스트를 설정하여 제거 대상 헤더를 커스터마이즈할 수 있습니다.

## 😃 XFowarded Headers Filter

XForarded Headers Filter는 다운스트림 서비스로 보낼 다양한 X-Fowarded-* 헤더를 생성합니다. 요청의 Host Header, Scheme, Port, Path를 사용해서 다양한 헤더를 만들 수 있습니다.

각 헤더에 대한 생성 여부는 Application.yaml에서 프로퍼티로 작성하여 변경할 수 있습니다.(default: true)

- spring.cloud.gateway.x-forwarded.for-enabled
- spring.cloud.gateway.x-forwarded.host-enabled
- spring.cloud.gateway.x-forwarded.port-enabled
- spring.cloud.gateway.x-forwarded.proto-enabled
- spring.cloud.gateway.x-forwarded.prefix-enabled

각 헤더에 여러개를 작성해야할 경우, append 가능여부를 Application.yaml에서 프로퍼티로 작성하여 변경할 수 있습니다.(default : true)

- spring.cloud.gateway.x-forwarded.for-append
- spring.cloud.gateway.x-forwarded.host-append
- spring.cloud.gateway.x-forwarded.port-append
- spring.cloud.gateway.x-forwarded.proto-append
- spring.cloud.gateway.x-forwarded.prefix-append