# 데코레이터 패턴(Decorator Pattern)

데코레이터는 장식가라는 뜻을 가지고 있는 단어이다. 그리고 장식은 액세서리 따위로 치장함. 또는 그 꾸밈새. 라는 사전적 의미를 가지고 있다. 즉, 꾸미기 패턴인 셈이다.

디자인 패턴에서 이 데코레이터 패턴은 특정 객체를 꾸미기 위한 패턴이다. 이 데코레이터 패턴을 이렇게 정의할 수 있다. `특정 객체에 추가 요소(꾸밈 요소)를 동적으로 더한다.`

커피숍의 메뉴를 생각해보자.

먼저, 음료를 선택한다. 아메리카노, 라떼, 에스프레소 등..

그 다음으로는 음료에 추가할 토핑을 선택한다. 시럽을 추가한다던지, 모카, 두유 등등 다양하게 선택한 뒤 음료 + 토핑 가격을 지불한다.

위의 내용을 프로그래밍 해보자.

먼저 음료는 추상화하고 각 음료의 설명과 가격을 제공할 수 있도록 기능을 만들자.

```java
public abstract class Beverage {
    String description = "이름 없음";
		int cost = 0;

    public String getDescription() {
        return description;
    }

    abstract long cost();
}
```

그리고 음료에 해당하는 Concrete Class를 만들자.

```java
public class Americano extends Beverage{

    public Americano() {
        description = "아메리카노";
    }

    @Override
    long cost() {
        cost += 2000;
    }
}
```

```java
public class Espresso extends Beverage{

    public Espresso() {
        description = "에스프레소";
    }

    @Override
    long cost() {
        cost += 1500;
    }
}
```

```java
public class Latte extends Beverage{

    public Latte() {
        description = "라떼";
    }

    @Override
    long cost() {
        cost += 2500;
    }
}
```

토핑 추가를 위해 각종 토핑을 추상화한 토핑클래스를 만들자.

```java
public class Topping extends Beverage{
	Beverage beverage;
	public abstract String getDescription();
}
```

어라? 이상하다. 왜 토핑인데 Beverage 클래스를 상속받고 인스턴스 변수로 Beverage를 두고있지?

이 데코레이터 패턴은 꾸밈패턴이라고 위에서 언급했는데, 꾸미기 위해서는 꾸밀 대상을 알고있어야 꾸밀 수 있다. 그래서 꾸밀 대상을 가지고 있는 것이다.

모카 토핑을 추가해보자.

```java
public class Mocha extends Topping{
	//꾸밀 객체를 대상으로 초기화 수행
	public Mocha(Beverage beverage){
		this.beverage = beverage;
	}

	//꾸밀 대상 객체의 음료 설명 뒤에 모카를 추가
	@Override
	public String getDescription(){
		return beverage.getDescription() + ", 모카");
	}

	//기존 음료가격에 가격 추가
	@Override
	public long cost(){
		return beverage.cost() + 700;
	}	
}
```

이번엔 우유 토핑을 추가해보자.

```java
public class Milk extends Topping{
	//꾸밀 객체를 대상으로 초기화 수행
	public Milk(Beverage beverage){
		this.beverage = beverage;
	}

	//꾸밀 대상 객체의 음료 설명 뒤에 모카를 추가
	@Override
	public String getDescription(){
		return beverage.getDescription() + ", 우유");
	}

	//기존 음료가격에 가격 추가
	@Override
	public long cost(){
		return beverage.cost() + 300;
	}	
}
```

오케이. 얼추 토핑을 추가할 수 있는 음료를 모두 준비한 것 같다.

음료를 한번 주문해보자.

“라떼 주문할껀데요. 우유 한번더 넣어주시고 모카 토핑 2번 추가해주세요.” 이사람.. 커피를 마시고싶은 것인지 모카 우유를 마시고싶은 것인지 모르겠지만, 일단은 가능하니 한번 주문대로 만들어줘보자.

```java
public class Caffe{
	public static void main(String[] args){
		//라떼 준비하고
		Baverage beverage = new Latte();
		//우유 추가하고
		beverage = new Milk(beverage);
		//모카 두번!
		beverage = new Mocha(new Mocha(beverage));

		System.out.println("주문하신 음료 나왔습니다!");
		System.out.println(beverage.getDescription()+" "+beverage.cost()+"원");
	}
}
```

앞서 토핑 추상 클래스에서 음료 추상 클래스를 상속받았고, 인스턴스 변수로 음료를 가지고 있었다. 그리고 토핑 콘크리트 클래스에서 생성자 매개변수로 음료를 전달받아 이를 꾸밀 수 있도록 하였다.

위 Caffe 클래스에서 `new Milk`, `new Mocha`와 `beverage.getDescription()`, `beverage.cost()`를 조금만 더 이해해보자.

1. `new Milk(beverage)` 라떼 음료 객체를 우유 객체로 감쌌다.
2. `new Mocha(berverage)` 우유가 추가된 라떼 음료 객체를 모카 객체로 감쌌다.

여기서 이제 `berverage.getDescription()`을 요청하면 `Mocha Class`의 `getDescription()`이 먼저 호출된다. 그리고 순차적으로 감싸진 객체의 `getDescription`이 호출될 것이다.

```java
//모카클래스
return beverage.getDescription() + ", 모카";
//현재 결과 : beverage.getDescription() + ", 모카"

//한번더 모카클래스
return beverage.getDescription() + ", 모카";
//현재 결과 : beverage.getDescription() + ", 모카, 모카"

//이번엔 우유클래스
return beverage.getDescription() + ", 우유";
//현재 결과 : beverage.getDescription() + ", 우유, 모카, 모카"

//마지막으로 라떼 음료
return description;
//최종 결과 : "라떼, 우유, 모카, 모카"
```

최종적으로는 “라떼, 우유, 모카, 모카” 라는 Description이 나온다.

이게 가능했던 이유는 데코레이터 패턴을 사용하여 구성과 위임을 통해 새로운 행동을 계속적으로 추가할 수 있었기 때문이다. 이 데코레이터 패턴을 통해 구상 구성요소(예시에서 라떼)를 계속적으로 감싸는(토핑 추가) 행위를 수행하였고, 이를 통해 `확장`할 수 있었다.

만약 위의 행위에 데코레이터 패턴을 적용하지 않았더라면, 각 음료마다 토핑을 추가할 수 있는 기능을 제공하고, 금액도 최종적으로 계산하려면 추상 클래스에 토핑이 있는지의 여부와 그에따른 금액 추가기능을 넣었어야할 것이다. 즉, 토핑이 추가될 때마다 `변경`이 이루어졌었을 것이라는 의미이다.

끝으로 데코레이터 패턴을 통해,  계속적으로 확장될 수 있는 추가 요소들을 대상으로 구성과 위임을 통해, 유연하게 확장시킬 수 있는 방법을 알아보았다.


> 💡 데코레이터 패턴(Decorator Pattern)으로 객체에 추가 요소를 동적으로 더할 수 있다.
> 데코레이터 패턴을 사용하면 서브클래스를 만들 때 보다 훨씬 유연하게 기능을 확장할 수 있다. 데코레이터는 데코레이팅 되는 대상에 대해 인스턴스 변수로 대상을 들고 있으며, 행위를 위임하는 일 말고도 추가할 수도 있다.