# Optional

T 타입 객체의 래퍼클래스 - Optional<T>

```java
public final class Optional<T>{
	private final **T valu**e; //T타입의 참조변수
  ...
}
```

- 필요한 이유

null을 직접 다루는 것은 위험하다.(NullpointException 발생) → 간접적으로 null을 다룰 수 있음

null체크를 위한 if문을 반드시 넣어야하는 코드의 지저분한 문제를 해결

## Optional<T> 객체 생성하기

- Optional<T> 객체를 생성하는 다양한 방법

```java
String str = "abc";

Optional<String> optVal = Optional.of(str);
Optional<String> optVal = Optional.of("abc");
Optional<String> optVal = Optional.of(null); //NullPointerException발생
Optional<String> optVal = Optional.ofNullable(null); //Okay
```

<aside>
💡 null대신 빈 Optional<T> 객체를 사용하자!!

</aside>

```java
Optional<String> optVal = null; // (x)
Optional<String> optVal = Optional.<String>empty(); //빈 객체로 초기화
```

## Optional<T> 객체의 값을 가져오기

- Optional객체의 값을 가져오는 방법 - get(), orElse(), orElseGet(), orElseThrow()

```java
Optional<String> optVal = Optional.of("abc");

String str1 = optVal.get(); //잘 사용하지 않음
String str1 = optVal.orElse(""); //optVal에 저장된 값을 반환. null이면 예외발생
String str1 = optVal.orElseGet(String::new); //null이면 new String()으로 초기화해서 반환
String str1 = optVal.orElseThrow(NullPointerException::new)//null이면 NullPointerException을 던짐
```

- isPresent() - Optional객체의 값이 null이면 false, 아니면 true를 반환
- ifPresent(람다) - Null이 아닐때만 내용을 수행

```java
if(Optional.ofNullable(str).**isPresent**()) {
	System.out.println(str);
}

Optional.ofNullable(str).**ifPresent**(System.out::println);
```

## OptinonalInt, OptionalLong, OptionalDouble

- 기본형 값을 감싸는 래퍼클래스
- 성능을 위해 씀

```java
public final class OptionalInt {
  ...
  private final boolean isPresent; //값이 저장되어 있으면 true
	private final int value; //int타입의 변수
```

- OptionalInt의 값 가져오기 - int getAsInt()

| Optional클래스 | 값을 반환하는 메서드 |
| --- | --- |
| Optional<T> | T get() |
| OptionalInt | int getAsInt() |
| OptionalLong | long getAsLong() |
| OptionalDouble | double getAsDouble() |

- 빈 Optional객체와의 비교

```java
OptionalInt opt = OptionalInt.of(0); //OptionalInt에 0을 저장
OptionalInt opt2 = OptionalInt.empty(); // OptionalInt에 0을 저장

System.out.println(opt.isPresent()); //true
System.out.println(opt2.isPresent()); //false
System.out.println(opt.equals(opt2)); //false 
```