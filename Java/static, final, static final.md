# static, final, static final

# Static

Static은 고정된 이라는 의미를 가지고있다. Static 키워드는 변수와 메서드를 만들 수 있는데, 다른말로 정적필드, 정적 메서드라고 표현한다. 이를 정적 멤버(클래스 멤버)라고 한다. 정적 필드와 정적 메서드는 객체에 소속된 멤버가 아니라 클래스에 고정된 멤버이다. 그렇기 때문에 클래스로더가 클래스를 로딩 후, Method Area에 클래스 별로 관리한다. 따라서, 클래스 로딩이 끝나는 즉시 바로 사용할 수 있다. 물론 정적 클래스도 존재한다.

Static은 앞서 말한 바와 같이 Heap영역이 아닌 Static 영역에 할당되는데, GC가 이를 관여하지 않는다. 따라서 프로그램의 종료시까지 메모리에 할당된 채로 존재하게 된다. 그렇기 때문에 Static을 너무 남발하게되면 시스템 성능에 악영향을 끼치게 된다.

필드나 메서드를 인스턴스로 생성할 것인지 정적으로 생성할 것인지의 여부는 필드나 메서드가 공용으로 사용되어져야 하는가를 기준으로 판단하면 된다. 공용으로 사용해야하는 경우에는 static 키워드를 작성하여 정적으로 만들면 된다.

**static 필드**

static 필드 또는 static 변수는 **메모리 공간에 하나만 존재**한다. 또한 **접근 제어자에 따라 어디서든지 접근이 가능**하다.

static 필드는 인스턴스가 생성되기 이전에 별도의 메모리 공간에 할당되어 초기화까지 완료한다. 초기화되는 시점은 JVM에 의해서 클래스가 메모리 공간에 올라갈 때이다.

위에서 언급한 바와 같이 해당 필드를 공유해서 사용해야한다면 static 필드로 만들면 된다.

```java
static int num = 0;
public static void staticMethod(){...}

```

**static 메서드**

마찬가지로, **인스턴스를 만들지 않아도 호출할 수 있는 메서드**이다.

객체를 생성할 필요가 없는 메서드일 경우 static 메서드를 활용한다. 보통 유틸성 메서드가 필요할 때, 이런식으로 많이 활용한다. 많이들 사용하는 Math 메서드가 이런식이다. Math.sum(x,y)를 호출할 때 Math 메서드의 인스턴스를 별도로 만들지 않고 그냥 정적 메서드를 호출하는 형태를 보면 더 쉽게 이해할 수 있다.

```java
class Name{
  static void print1(){
		System.out.println("안녕하세요");
  }
	void print2(){
		System.out.println("안녕하세요");
	}
}
```

앞서 말한바와 같이 static 키워드가 붙은 static 메서드나 static 필드는 Method Area의 Class Variable에 저장된다. 때문에 인스턴스를 생성하지 않아도 호출할 수 있다.

```java
Name.print1();
new Name().print2();
```

> 💡 Static이 나오면 Method Area, Class Variable, GC 키워드를 떠올리자.


**static 클래스**

static 클래스의 경우에는 분리한다는 의미에서 위와 비슷하다.

static 클래스의 경우에는 상위 클래스와 분리해준다.

여기서 상위 클래스라는 개념이 다소 읭? 할 수 있는 부분이긴한데, 아래 예시를 보자.

```java
public class Human{
	public int age = 31;

	public class eye{
	}
}
```

위와 같이 Class 내부에 또다른 class를 **내부 클래스(Inner Class)**라고 지칭한다.

예시 같은경우는 Human 클래스 안에 eye 클래스가 존재한다. 이런 경우에는 eye에서 age 즉, Human이라는 **상위클래스의 클래스 변수에 접근**할 수 있다.

만약 eye 클래스를 static으로 하면?

```java
public class Human{
	public int age = 31;

	public static class eye{
	}
}
```

더이상 eye 정적 클래스는 age에 접근할 수 없다. 완전히 Human과 분리된 클래스이다.

그러면 완전히 다른 클래스파일로 분리해서 사용하면 되는 것이 아니냐 라고 생각할 수 있는데, 이는 grouping의 이점을 가져가기 위함이다. 어떤 클래스와 연관된 클래스를 하위에 선언하여 관련있는 클래스들을 모아둘 수 있다.

근데 왜 그래야하나 싶긴하다… 굳이(?)

# final

final은 마지막이라는 뜻을 가지고 있고, 이 키워드는 변수, 클래스, 메서드에 적용될 수 있다.

**final 변수**

변수를 **상수화 시킨다.** 즉 한번 값이 결정된 변수 변경이 불가능하다.

```java
final String a = "변경이 불가능함";
```

**final 클래스**

해당 클래스의 **상속을 허용하지 않는다.**

대표적으로 String 클래스가 final 클래스이다.

![Untitled](static,%20final,%20static%20final/Untitled.png)

만약 final 키워드가 붙은 클래스를 상속하려고 하면,

![Untitled](static,%20final,%20static%20final/Untitled%201.png)

final 이기때문에 상속이 불가하다는 컴파일 에러가 발생한다.

**final 메서드**

final 키워드가 붙은 메서드를 **오버라이딩할 수 없다.**

이도 마찬가지로 final 메서드에 강제로 오버라이딩을 시도하면

![Untitled](static,%20final,%20static%20final/Untitled%202.png)

해당 메서드는 final 메서드이기때문에 오버라이딩할 수 없다고 컴파일 에러가 발생한다.