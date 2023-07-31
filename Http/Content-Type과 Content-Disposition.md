# Content-Type과 Content-Disposition

  브라우저는 컨텐츠를 처리하는 방식으로 Content-Type과 Content-Disposition 헤더값을 조합하여 해당 컨텐츠의 성격과 타입을 인지하고 이를 바탕으로 컨텐츠를 처리한다.

  ## Content-Type
  **Content-Type** 은 리소스의 Media Type을 나타내기 위해 사용된다.
  HTTP 응답 내에 존재하는 **Content-Type** 헤더는 클라이언트에게 반환된 컨텐츠의 유형이 무엇인지를 알려주는 역할을 수행하는데, 일부 브라우저에서는 MIME 스니핑으로 이 헤더의 값을 무시하는 경우가 있다.
    이를 해소하기 위해, **X-Content-Type-Option** 헤더를 nosniff로 설정하면 스니핑이 불가능하다.

  HTTP 응답 메시지 내 Content-Type의 예시는 아래와 같다.

  ```
      Content-Type: text/html; charset=utf-8
      Content-Type: multipart/form-data; boundary=something
  ```

  ## Content-Disposition
  **Content-Disposition** 은 HTTP 응답되는 컨텐츠가 브라우저에 `inline`되어야 하는 컨텐츠인지 또는 `attachment`로써 다운로드 되어야하는 컨텐츠인지를 식별할 수 있도록 작성되는 헤더이다.
  
