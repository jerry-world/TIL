# String  Constant Pool(String 상수 풀)

```java
String a = "a";
String b = "a";
String c = a;
String d = new String("a");
```

위에 a, b, c, d 각각의 변수에 `“a”` 라는 값을 담았다. 이 변수들은 모두 같은 주소 값을 가지고 있는가?

결과를 살펴보자.

![Untitled](String%20Constant%20Pool(String%20상수%20풀)/Untitled.png)

a와 b, c는 모두 동일하다. 하지만 d는 유일하게 혼자 다르다고 나온다. 이 이유가 무엇일까?

이게 바로 상수풀 때문이다. new 키워드를 사용하여 String 객체를 만들어 초기화한 d를 제외한 나머지는 상수풀에 “a”라는 값을 두고 각각의 a, b, c가 이를 가르키고 있는 것이고, d 만 유일하게 Heap 영역에 String 객체를 만들어 가르기고 있기 때문에 이는 서로 다르다고 판단하는 것이다.

물론, String의 equals 재정의로 모든 값들을 equals 메서드로 비교하면 같다고 나온다. 하지만 이 대상들은 엄연히 서로 다른 주소를 갖은 값이다.

그러면 JVM 구조로 이를 조금더 이해해보자.

과거 JAVA 1.7 버전 이하에서는 PermGen 이라는 공간이 존재하였었다. 이 공간은 클래스 메타 데이터가 들어가는 공간이였는데, 이 영역에서 String 상수 풀이 관리되었었다. 그리고 PermGen 영역은 JAVA 1.8 버전 이후로 Metaspace로 영역이 바뀌고 Native Memory영역에 해당하는 영역으로 변경되었다. 그래서 OS레벨에서 이를 관리하기 때문에 개발자가 이 영역의 할당 크기에 대한 부분을 크게 의식할 필요가 없어졌다.

기존 PermGen 영역에서 관리된 Static Object는 Native Memory 영역에 해당하면 안되니깐 이를 Heap 영역으로 옮겼다. 그리고 이들은 GC의 관리 대상이 되었다.

그 대상 중 하나가 바로 String 이다. 그러니깐 다시말하면 String 상수 풀은 기존에 Permgen 영역에 별도로 존재했었지만, Java 1.8 이후 부터 Heap 영역에 함께 공존한다. 그리고 GC에 의해 관리 될 수 있는 대상이다.

위의 예제에서 a, b, c는 힙영역의 String 상수 풀에 저장된 값을 쓰고 있는 것이고, d는 힙영역에 쓰여진 값을 쓰고 있는 것이다. 서로 바라보고 있는 대상의 위치가 다르다.