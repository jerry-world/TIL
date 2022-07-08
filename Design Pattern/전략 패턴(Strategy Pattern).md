# 전략 패턴(Strategy Pattern)

왜 전략이라는 단어를 사용했을까?  전략의 사전적 정의를 살펴보면 `전쟁을 전반적으로 이끌어 가는 방법이나 책략` 이라는 뜻을 가진 단어이다.

그러면 이 단어를 중심으로 이해해보면, 이 전략 패턴 안에는 방법이나 책략 따위가 있고 그것을 사용하거나 활용하는 대상이 있지 않을까? 유추해볼 수 있을 것 같다.

전략 패턴을 한 문장으로 요약해보자면, `전략을 캡슐화하여 이 전략을 사용하는 대상이 동적으로 또다른 전략을 바꿀 수 있게 해주는 패턴`이다. 여기서의 전략은 어떠한 목적을 달성하기 위한 일을 수행하는 방식을 담당하는 알고리즘일 것이다.

## UML 살펴보기

전략 패턴의 일반적인 UML을 한번 살펴보도록하자.

![Untitled](./전략 패턴(Strategy Pattern)/Untitled.png)

Context, Strategy 인터페이스, Concrete 클래스에 집중해보자.

Strategy 인터페이스는 method라는 추상 메서드를 가지고 있고, 이를 실체화한 Concrete 클래스 3개가 있다.

그리고 실제로 이 Strategy를 사용하는 Context가 있다.

구조는 상당히 단순하다. 전략을 사용하는 Context가 Strategy 인터페이스에 의존하여 전략을 선택해 사용할 수 있다. 또한 동적으로 또다른 전략을 사용할 수 있겠다.

## 전략 패턴 구현하기

그러면 한번 전략 패턴을 구현해보자.

전략 패턴의 예시로는 여러 무기로 공격을 수행하는 공격 기능을 제공해보자.

공격 무기로는 맨주먹과 나이프가 준비되어 있다. 게임이 클로즈 베타 단계여서 그렇게 무기를 많이 제공하지 못하였다.

Weapon 인터페이스를 상속받아 다양한 Concrete 클래스인 맨주먹 클래스와 나이프 클래스를 구현하였다. 그리고 각 클래스 들은 공격기능을 동일하게 제공한다. 플레이어는 무기를 직접 습득하고 이를 사용할 수 있다.

```java
//캐릭터를 생성하고 게임을 하는 플레이어
public class Client{
	public static void main(String[] args){
		//사용자가 캐릭터를 지정하였다.
		Character bob = new Character();

		//무기가 없는 상태에서 좌클릭을 수행하였다.	
		bob.leftClick();
		
		//나이프를 획득하였다.
		bob.getWeapon(new Knife());
		//나이프를 든 채로 좌클릭을 수행하였다.
		bob.leftClick();
	}
}
```

```java
//캐릭터 클래스는 좌클릭 기능과 무기 습득 기능을 제공한다.
public class Character(){
	private Weapon weapon;

  //좌클릭을 수행하면 가지고 있는 무기 또는 맨주먹으로 공격을 수행한다.
	public void leftClick(){
		weapon.attack();
	}
	
	//무기를 습득하면 자동으로 무기를 장착한다.
	public void setWeapon(Weapon weapon){
		this.weapon = weapon;
	}
}
```

```java
//공격 기능을 추상화
public interface Weapon{
  void attack();
}
```

```java
public class Fist implements Weapon{
	@override
	void attack(){
		System.out.println("맨손 공격 얍얍!");
	}
}
```

```java
public class Knife implements Weapon{
	@override
	void attack(){
		System.out.println("슈슈슉슉");
	}
}
```

만약 추가적으로 더 많은 무기 제공을 위해 업데이트를 해야한다면, 이 Weapon 인터페이스를 추가 구현하여 기존 소스코드를 건드리지 않고도 무기를 확장할 수 있다. `SOLID 원칙에서 OCP 원칙을 준수할 수 있다.`

```java
public class M4A1 implements Weapon{
	@override
	void attack(){
		System.out.println("투두둑!!");
	}
}
```

이처럼 전략 패턴은 클라이언트 코드에서 전략을 런타임에 동적으로 변경할 수 있도록 하고, 이에 따라 적절한 알고리즘을 변경하며 활용할 수 있다.