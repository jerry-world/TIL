# Lamda나 Stream에서 변수 참조 시, final을 써야하는 이유

면접에서 익명 객체에서는 외부 변수를 사용하려면 final을 사용해야하는데 왜 그런지 물어봤었다. 

그런데 제대로 대답할 수가 없었다.. 정리해보자.

결론은 `동시성` 때문이다. 동시성을 생각하면서 아래의 내용을 읽어보자.

먼저 람다에서 외부 지역변수를 사용할때는 아래와 같은 특징이 있다.

- 람다에서 외부 지역변수를 사용할 때, 이 외부 지역변수의 `복사본`을 사용한다.
- `final` 혹은 `effectively final`인 외부 지역변수만 사용할 수 있다.
- 복사된 지역 변수 값은 `변경할 수 없다`.

그러면 이 특징들을 하나씩 이해해보자.

![Untitled](Lamda나%20Stream에서%20변수%20참조%20시,%20final을%20써야하는%20이유/Untitled.png)

 f2 람다 구문 내에서 지역변수를 증가시켜보았다. 그랬더니, 람다 표현식에서 사용되는 변수는 `final` 이거나 `effectively final` 이어야 한다고 컴파일 에러가 나왔다.

effectively final은 Java8에서 추가된 기능으로, non-final로 선언 즉, final 키워드를 붙히지 않았지만 초기화 이후 한번도 변경되지 않은 지역 변수를 대상으로 컴파일러가 Lambda나 익명 클래스에서 final 변수로 취급하는 것이다.

<aside>
💡 **Effectively final**
A non-final local variable or method parameter whose value is never changed after initialization is known as effectively final.

초기화 이후 한번도 변경되지 않은 ‘final이 아닌 로컬 변수’ 또는 ‘메서드 파라미터’ 는 effectively final 이라고 할 수 있다.

</aside>

Java 7에서는 익명 클래스가 외부 지역변수에 접근하기 위해서는 final인 경우에만 접근이 가능했다. 따라서, 항상 final 키워드를 추가해줘야했다. 하지만 Java 8부터는 이 effectively final이 추가되어 effectively final 조건을 충족하면 final 키워드를 붙히지 않고도 익면 클래스 내에서 외부 지역변수를 사용할 수 있게 되었다.

그 이유는 무엇일까? 

## 람다에서 외부 지역변수를 사용할 때, 이 외부 지역변수의 복사본을 사용한다.

바로 `람다 캡처링` 때문이다. 아까 서두에서 람다는 외부 지역 변수의 `복사본` ****을 사용한다고 말했다.

<aside>
💡 Lambda Variable Capturing
기존 메서드의 스택 변수들에 대해 참조가 가능하도록 람다의 스택에 복사하는 과정

</aside>

람다를 사용할때 JVM 메모리 구조를 이해해야 내용을 이해할 수 있다.

```java
public static void main(String[] args) {
    int listCount = 0;
    List<Integer> numList = Arrays.asList(1, 2, 3);
}
```

위와같은 코드가 JVM 메모리로 표현하면

![Untitled](Lamda나%20Stream에서%20변수%20참조%20시,%20final을%20써야하는%20이유/Untitled%201.png)

이렇게 표현될 수 있다.

이구조에서 람다가 실행되면

![Untitled](Lamda나%20Stream에서%20변수%20참조%20시,%20final을%20써야하는%20이유/Untitled%202.png)

이렇게 람다만의 새로운 스택이 생기게 된다.

즉, 람다가 실행되면 람다는 새로운 스택을 생성해서 사용한다. 이게 복사본이다.

지역 변수는 JVM 영역에서 위 그림처럼 스택에 생성이 되는데, 따라서 지역 변수가 선언된 block이 끝나면 그 해당 스택은 제거된다.

그러면 메서드 내 지역 변수를 참조하는 람다식을 리턴하는 메서드가 있을 때, 만약 이 람다식이 리턴하기전에 메서드 블럭이 끝나버리면 람다식은 지역 변수를 리턴할 수 없게 된다.

또, 지역 변수를 관리하는 스레드와 람다표현식이 실행되는 스레드가 다를 수 있다.

스택은 각 스레드의 고유 공간이다. 그리고 스레드끼리는 이 고유 공간을 공유하지 않기 떄문에 마찬가지로 람다식이 수행될 때 값을 참조할 수가 없다.

때문에, 람다식에서 외부 지역변수의 복사본을 쓰는 것이다.

그래서 final이나 effectively final을 써야하는 것이다.

## `final` 혹은 `effectively final`인 외부 지역변수만 사용할 수 있다.

아래의 예시를 보자.

![Untitled](Lamda나%20Stream에서%20변수%20참조%20시,%20final을%20써야하는%20이유/Untitled%203.png)

![Untitled](Lamda나%20Stream에서%20변수%20참조%20시,%20final을%20써야하는%20이유/Untitled%204.png)

좌측 예시에서는 마지막 라인에 boolean인 isReal 값을 false로 변경하였고, 우측은 변경하지 않았다.

그랬더니, Runnable인터페이스의 run을 정의하는 과정에서 isReal이 에러가 나지 않았다. 서로 스레드가 다르기 때문에, 익명 클래스 객체가 외부 지역 변수의 복사본을 사용하는데, 이 복사된 값이 익명 클래스 바깥쪽에서 초기화 이후 변경되지 않음을 보장하였기 때문에 에러가 나지 않았다.

이때의 isReal 변수가 바로 `effectively final` 인 것이다.

마찬가지로 final로 변경하면

![Untitled](Lamda나%20Stream에서%20변수%20참조%20시,%20final을%20써야하는%20이유/Untitled%205.png)

final 이어서 위에서 이미 초기화되엇기 때문에 값을 변경할 수가 없다. 때문에, 값이 변경되지 않음을 보장할 수 있었다. 그래서!! final 또는 effectively final을 사용해야 하는 것이다.