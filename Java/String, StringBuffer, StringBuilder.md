# String, StringBuffer, StringBuilder

String과 StringBuffer/StringBuilder의 가장 큰 차이점은 String은 불변성을 지니지만 StringBuffer와 StringBuilder는 가변성을 지닌다는 것이다.

String 객체는 한번 값이 할당되면 그 공간은 변하지 않는다. 만약 새로운 값을 할당하면 해당 값에 해당하는 메모리 영역을 가리키도록 변경된다. 그러면 기존 값은 Garbage로 남아있다가 GC에 의해 사라지게 된다.

결국 String 클래스는 불변하기 때문에 문자열을 수정하게되면 새로운 String 인스턴스가 생성되는 것이다.

![Untitled](String,%20StringBuffer,%20StringBuilder%2076e604f54fe84ce3bf42e9228c16898d/Untitled.png)

String은 불변성을 가지기 때문에 문자열을 자주 읽는 작업을 수행하는 곳에서는 좋은 성능을 기대할 수 있다. 하지만 문자열 추가, 수정, 삭제 등의 연산작업이 빈번히 발생하는 경우 힙 메모리에 많은 Garbage가 생성되어 힙메모리 부족 이슈가 발생할 수도 있다.

이러한 연산 작업이 빈번한 경우에는 가변성을 가지는 StringBuffer나 StringBuilder를 사용해야한다.

StringBuffer와 StringBuilder는 Mutable 성질을 가지고 있기 때문에, 연산 작업을 수행하더라도, 동일 객체의 문자열을 변경한다.

![Untitled](String,%20StringBuffer,%20StringBuilder%2076e604f54fe84ce3bf42e9228c16898d/Untitled%201.png)

그러면 StringBuffer와 StringBuilder는 어떤 차이를 가지고 있을까?

이는 동기화의 유무에 있따. StringBuffer는 동기화 키워들르 지원하여 멀시트레드 환경에서 안전하다(thread-safe). String 또한 마찬가지로 불변성을 가지기 때문에 멀테스레드환경에서 안전하다.

반대로 StringBuilder는 동기화를 지원하지 않기 때문에 멀티스레드 환경에서 사용하는 것은 부적합하다. 하지만 이 3가지 중에 가상 성능이 뛰어나다는 장점을 가지고 있다. 동기화가 필요하지 않으면 StringBuilder를 선택하면 된다.

![Untitled](String,%20StringBuffer,%20StringBuilder%2076e604f54fe84ce3bf42e9228c16898d/Untitled%202.png)