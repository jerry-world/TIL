# 싱글톤 패턴(Singleton Pattern)

싱글톤 패턴은 한 클래스의 인스턴스를 단 하나만 만드는 패턴이다.

자바 언어에서 클래스의 인스턴스를 만들기 위해서는 인스턴스를 만드고자하는 클라이언트 코드에서 new 연산자를 생성자에 접근하여 생성할 수 있다. `AClass aClass = new AClass();`

그리고 이 new 연산자를 통한 생성자를 접근할 때마다 클래스의 인스턴스가 계속적으로 생성된다.

그러면 이 싱글톤 패턴은 어떻게 한 클래스에 대해 하나의 인스턴스만을 만들 수 있도록 보장할까?

클래스 내부에서 인스턴스를 만들어서 만든 인스턴스만 사용하도록 만드는 것이다.

### 1. 생성자 접근 차단

위에서 언급했던 것 처럼 인스턴스를 만들기 위해서는 new 연산자와 함께 생성자를 호출하는 것인데, 생성자를 호출하면 결국 클래스의 인스턴스를 만들 수 있기 때문에, 생성자를 호출하지 못하도록 접근제어자를 통해 제어한다.

```java
public class AClass{
	//private 접근 제어자를 통해 생성자 외부 접근 차단
	private AClass(){
	}
}
```

### 2. 내부에서 인스턴스 생성하기

이번엔 외부에서 해당 클래스에게 인스턴스를 요청하면 인스턴스를 제공할 수 있도록 메서드를 구현해보자.

```java
//Eager Initialization

public class AClass{
	private static AClass instance = new AClass();

	//private 접근 제어자를 통해 생성자 외부 접근 차단
	private AClass(){
	}

	public static AClass getInstance(){
		return instance;
	}
}
```

내부에 AClass의 하나뿐인 인스턴스를 저장하는 정적변수 

그리고 외부에서 이를 접근하여 갖을 수 있도록 `getInstance`메서드를 제공하였다.

이젠 외부에서 생성자로 인스턴스를 만들지 않고, AClass를 통해 생성된 인스턴스를 가져다가 사용하는 구조로 만들었다.

기억하자. 싱글톤 패턴이 적용된 이 싱글톤 객체는 `public 으로 지정된 생성자가 없다.`

오로지 `싱글톤 클래스로부터 객체를 제공받는 getInstance()` 정적 메서드만 존재한다.

그런데 이 AClass에는 문제점이 있어보인다. 만약 그 어느 누구도 애플리케이션에서 AClass를 사용하지 않는다 할지라도, 무조건적으로 인스턴슽를 만들어 static 메모리 영역을 차지한다는 점이다.

그러면 누군가 단한명이라도 사용한다고 할때 인스턴스를 만들고 이를 제공해주는 형태로 바뀌는편이 좋을 것 같다.

### 3. 사용 시점에 인스턴스를 만들고 이를 제공하기

`getInstance`를 호출할 때, 한번도 호출되지 않았으면 인스턴스를 만들고, 누군가 호출한 적이있으면 호출했을 때 생성한 인스턴스를 반환해주면 되겠다.

```java
//Lazy Initialization

public class AClass{
	private static AClass instance;

	//private 접근 제어자를 통해 생성자 외부 접근 차단
	private AClass(){
	}

	public static AClass getInstance(){
		if(Objects.isNull(instance){
			instance = new AClass();
		}
		return instance;
	}
}
```

Null 체크를 통해, 호출한 적이 있으면 instance를 바로 반환해주고, 그렇지 않으면 인스턴스를 만들어 반환하게끔 변경했다. 이젠 쫌 형태가 잡힌 것 같다.

앗, 그런데 한가지 더 문제가 있다. 만약에 멀티 스레드 환경에서 아주 극적으로 동시에 인스턴스가 없는 상태인데 getInstance를 두 스레드가 접근하게 되었다면?

둘다 instance가 없다고 인지해버려서 객체를 두개를 만들어 버렸다면?

싱글톤 패턴의 특징인 하나의 인스턴스만 만들어서 제공한다는 그 규칙이 깨져버린다…

가장 간단하게 생각할 수 있는 방법이 `synchronized`키워드를 사용해서 동시에 접근할 수 없도록 막아버리면 어떨까?

### Syncronized를 통해 Thread Safe하게 만들기

```java
//Thread-safe (1)

public class AClass{
	private static AClass instance;

	//private 접근 제어자를 통해 생성자 외부 접근 차단
	private AClass(){
	}

	public static synchronized AClass getInstance(){
		if(Objects.isNull(instance){
			instance = new AClass();
		}
		return instance;
	}
}
```

음.. 된건가?

그래도 이 synchronized 키워드때문에 동시에 생성되는 것만 막으려고 했는데, 매번 스레드에서 요청할 때마다 이 임계구역때문에 성능이 저하될 것 같다.. 그러면 생성되는 부분만 감싸보자.

```java
//Double Check! Thread-safe (2)
//DCL(Double-Checked Locking)

public class AClass{
	private volatile static AClass instance;

	//private 접근 제어자를 통해 생성자 외부 접근 차단
	private AClass(){
	}

	public static AClass getInstance(){
		if(Objects.isNull(instance){
			synchronized{
				//앞서 임계구역에 접근했던 대상이 있으면 인스턴스를 이미 만들었을 거니깐 한번 더 체크
				if (Objects.isNull(instance)) {
					instance = new AClass();
				}
			}
		}
		return instance;
	}
}
```

뭔가.. 처음보다 굉장히 복잡해지고 어질어질해졌다.

사실 여기까지만 해도 원하는 형태는 이미 만들어진 것 같다.

### Bill Pugh

사실 구조도 복잡하고 이를 좀더 쉽게 만들 수 없을까 고민하던 윌리엄 퓨라는 개발자가 아래의 모델을 제안했다. 방법은 `동기화 작업을 JVM에게 위임`하는 것이다.

```java
public class Singleton {
  private Singleton() {}

  public static Singleton getInstance() {
  	return LazyHolder.singleton;
  }

  private static class LazyHolder{
  	private static final Singleton singleton = new Singleton();
  }

}
```

구조를 살펴보면, 싱글톤 클래스 내에 이너 클래스(정적 클래스)가 하나 더있다. JVM이 클래스를 로딩할 때 이너 클래스를 바로 생성하지 않는다. getInstance 메서드가 호출되어 LazyHolder를 호출할 때 비로소 객체가 생성된다. LazyIntialization 방식과 다소 비슷해보이지만, 극명한 차이는 동기화 작업을 JVM에게 위임하는데 있다. 누군가 getInstance를 호출하면 static final로 작성된 이 싱글톤 객체가 이미 JVM 메모리에 올라와있기 때문에 싱글톤 객체임을 보장할수 있다.

기존에 작성한 DCL 과 volatile을 결합한 패턴보다 훨씬더 형태가 간단해졌다.