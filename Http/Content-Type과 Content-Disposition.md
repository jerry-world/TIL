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
  `multipart/form-data` Content-Type 에서는 Content-Type에 작성된 구분자를 기준으로 Content-Disposition을 통해, 대상에 대한 정보를 식별할수 있도록 명시하기도 한다.
  
  기본값은 `inline` 이며, 브라우저에 `inline`되어야하는 웹페이지 자체 또는 웹페이지의 일부 데이터를 의미한다.
  
  `attachment` 인 경우, Content-Type 형태의 대상을 첨부파일로써 다운로드되어야 함을 명시하고, 해당 파일명을 명시한다.


  ```
    Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
    Contnet-Disposition: attechment; filename=example.xlsx
  ```
  예시에서는 openxml 포멧의 스프레드 시트 형태의 오피스 문서에 해당하는 컨텐츠 타입을 갖고, 첨부파일로서 example.xlsx 이름으로 다운로드 됨을 명시하고있다.

  `multipart/form-data`의 경우는 대용량의 데이터를 대상으로한다. 여기서 대용량의 데이터는 여러 타입의 데이터를 구분하여 가지고 있음을 의미하여, 여러 파일이 될 수도 있고, 여러 값이 될 수도 있다.
  Content-Type에 명시된 boundary(구분자)를 기준으로 Content-Disposition에 각 대상의 정보를 명시한다.

  또한, 더이상 컨텐츠가 없음을 명시하기 위해서, 마지막 컨텐츠 최 하단에 구분자+ "--" 를 명시한다.

  ```
    Content-Type: multipart/form-data; boundary="WebKitFormBoundary8d7f6sd875bf7Jf"

    ------WebKitFormBoundary8d7f6sd875bf7Jf
    Content-Disposition: form-data; name="name"

    박선용
    ------WebKitFormBoundary8d7f6sd875bf7Jf
    Content-Disposition: form-data; name="age"

    32
    ------WebKitFormBoundary8d7f6sd875bf7Jf
    Content-Disposition: form-data; name="address"

    수원시 장안구 이목로
    ------WebKitFormBoundary8d7f6sd875bf7Jf
    Content-Disposition: form-data; name="file"; filename="주민등록등본.pdf"
    Content-Type: application/pdf


    ------WebKitFormBoundary8d7f6sd875bf7Jf--
  ```
