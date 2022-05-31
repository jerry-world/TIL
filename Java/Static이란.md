# Static이란

Static은 고정된 이라는 의미를 가지고있다. Static 키워드는 Static 변수와 Static 메서드를 만들 수 있는데, 다른말로 정적필드 또는 정적 메서드라고 표현한다. 이를 정적 멤버(클래스 멤버)라고 한다. 정적 필드와 정적 메서드는 객체에 소속된 멤버가 아니라 클래스에 고정된 멤버이다. 그렇기 때문에 클래스로더가 클래스를 로딩 후, Method Area에 클래스 별로 관리한다. 따라서, 클래스 로딩이 끝나는 즉시 바로 사용할 수 있다.

Static은 앞서 말한 바와 같이 Heap영역이 아닌 Static 영역에 할당되는데, GC가 이를 관여하지 않는다. 따라서 프로그램의 종료시까지 메모리에 할당된 채로 존재하게 된다. 그렇기 때문에 Static을 너무 남발하게되면 시스템 성능에 악영향을 끼치게 된다.

필드나 메서드를 인스턴스로 생성할 것인지 정적으로 생성할 것인지의 여부는 필드나 메서드가 공용으로 사용되어져야 하는가를 기준으로 판단하면 된다. 공용으로 사용해야하는 경우에는 static 키워드를 작성하여 정적으로 만들면 된다.

```java
static int num = 0;
public static void staticMethod(){...}

```

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

Static이 나오면 Method Area, Class Variable, GC 키워드를 떠올리자.