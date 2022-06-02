# Lamda

함수형 언어는 Java 8부터 지원한다.

# 람다식(Lambda Expression)

함수(메서드)를 **간단한 식으로 표현**하는 방법이다.

익명 함수(이름이 없는 함수, anonymous function)이라고도 부른다.

자바에서는 익명 함수가 아닌 **익명 객체** 이다. 클래스 없이 메서드만 존재할 수 없기 때문에.

```java
int max(int a, int b) {
	return a > b ? a : b;
}
```

```java
(int a, int b) -> {
return a > b ? a : b;
}
```

- 함수와 메서드의 차이

근본적으로는 동일하다. 메서드는 객체지향개념에서 사용하는 용어이다.

함수는 클래스로부터 독립적이고 메서드는 클래스에 종속적이다.

- 람다식 작성하기
1. 메서드의 이름과 반환타입을 제거하고 ‘→’를 블록{} 앞에 추가한다.

```java
(int a, int b) -> {
return a > b ? a : b;
}
```

1. 반환값이 있는 경우, 식이나 값만 적고 return문 생략할 수 있다.(끝에 ‘;’ 안 붙힘)

```java
(int a, int b) -> a > b ? a : b
```

1. 매개변수의 타입이 추론 가능하면 생략할 수 있다.(대부분 생략가능)

```java
(a, b) -> a > b ? a : b
```

- 람다식 작성시 주의사항
1. 매개변수가 하나인 경우, 괄호를 생략할 수 있다.(타입이 없을 때만)

```java
(a) -> a * a
(int a) -> a * a
```

```java
(a) -> a * a
int a -> a * a //에러 발생
```

1. 블록 안의 문장이 **하나뿐일 때**, 괄호{} 생략가능(끝에 ‘;’ 안 붙힘)

```java
(int i) -> {
	System.out.println(i);
} 
```

```java
(int i) -> System.out.println(i)
```

# 함수형 인터페이스

단 하나의 추상 메서드만 선언된 인터페이스를 함수형 인터페이스라 부른다.

```java
@FuntionalInterface
interface MyFunction {
	public abscract int max(int a, int b);
}
```

MyFuction의 익명클래스, 클래스 선언, 객체 생성

```java
MyFunction f1 = new MyFunction() {
            @Override
            public int max(int a, int b) {
                return a > b ? a : b;
            }
        };

MyFunction f2 = (a, b) -> a > b ? a : b;
int max = f2.max(3, 5);

System.out.println("f1.max(3,5) = " + f1.max(3, 5));
System.out.println("max = " + max);
```

위 f1과 f2는 동일한 결과를 반환한다.(너무 당연한 이야기다..)

Comparator 같은 경우가 함수형 인터페이스로 되어있다.

## 함수형 인터페이스 타입의 매개변수, 반환타입

### 함수형 인터페이스 타입의 매개변수

```java
void aMehtod(MyFunction f) {
	f.myMethod();
}
```

```java
MyFunction f = () -> System.out.println("myMethod()");
aMethod(f);

//inline
aMethod(()-> System.out.println("myMethod()");
```

### 함수형 인터페이스 타입의 반환타입

```java
MyFunction myMethod() {
	MyFunction f = () -> {};
	return f;
}

//inline
MyFunction myMethod(){
	reutrn () -> {};
}
```

예시

```java
@FunctionalInterface
interface MyFunction2{
    void run();
}

public class LamdaExample02 {
    static void execute(MyFunction2 f){
        f.run();
    }

    static MyFunction2 getMyFunction(){
        return () -> System.out.println("f3.run()");
    }

    public static void main(String[] args) {
        MyFunction2 f = () -> System.out.println("f1.run()");

        MyFunction2 f2 = new MyFunction2() {
            @Override
            public void run() {
                System.out.println("f2.run()");
            }
        };

        MyFunction2 f3 = getMyFunction();

        f.run();
        f2.run();
        f3.run();

        execute(f);
        execute(()-> System.out.println("run()"));
    }
}
```

# java.util.function 패키지

### 자주 사용하는 함수형 인터페이스

자주 사용되는 다양한 함수형 인터페이스를 제공하는 패키지이다.

| 함수형 인터페이스 | 메서드 | 설명 |
| --- | --- | --- |
| java.lang.Runnable | void run() | 매개변수도 없고, 반환값도 없음 |
| Supplier<T> | T get() | 매개변수는 없고, 반환값만 있음 |
| Consumer<T> | void accept(T t) | Supplier와 반대로 매개변수만 있고, 반환값이 없음 |
| Function<T,R> | R apply(T t) | 일반적인 함수, 하나의 매개변수를 받아서 결과를 반환 |
| Predicate<T> | boolean test(T t) |  표현하는데 사용됨.
매개변수는 하나, 반환 타입은 boolean |

```java
Supplier<Integer> f = () -> (int)(Math.random() * 100) +1;
Consumer<Integer> f = i -> System.out.println(i+" ");
Function<Integer> f = i -> i/10*10;
Predicate<Integer> f = i -> i%2==0;
```

### 매개변수가 2개인 함수형 인터페이스

| 함수형 인터페이스 | 메서드 | 설명 |
| --- | --- | --- |
| BiConsumer<T> | void accept(T t, U u) | 두개의 매개변수만 있고, 반환값이 없음 |
| BiFunction<T,R> | R apply(T t, U u) |  두개의 매개변수를 받아서 하나의 결과를 반환 |
| BiPredicate<T> | boolean test(T t, U u) |  표현하는데 사용됨.
매개변수는 두개, 반환 타입은 boolean |

### 매개변수의 타입과 반환타입이 일치하는 함수형 인터페이스

| 함수형 인터페이스 | 메서드 | 설명 |
| --- | --- | --- |
| UnaryOperator<T> | T apply(T t) | Function의 자손, Function과 달리 매개변수와 결과의 타입이 같다.(단항연산자) |
| BinaryOperator<T> | T apply(T t, T t) | BiFunction의 자손, BiFunction과 달리 매개변수와 결과의 타입이 같다(이항연산자) |

# Predicate의 결합

### and(), or(), negate()로 두 Predicate를 하나로 결합(default 메서드)

```java
Predicate<Integer> p = i -> i < 100;
Predicate<Integer> q = i -> i < 200;
Predicate<Integer> r = i -> i%2 == 0;
```

위와같이 Predicate가 작성되어있을 때,

```java
Predicate<Integer> notP = p.**negate**(); // i >=100
Predicate<Integer> all =  notP.**and**(q).**or**(r); // 100 <= i && i< 200 || i%2 == 0
Predicate<Integer> all2 = notP.**and**(q.**or**(r)); // 100 <= i && (i< 200 || i%2 == 0)
```

와 같은형태로 Predicate를 결합할 수 있다.

```java
System.out.println(all.**test**(2)); // true
System.out.println(all2.**test**(2)); // false
```

### 등가 비교를 위한 Predicate의 작성에는 isEqual()를 사용(static 메서드)

```java
Predicate<String> p = Predicate.**isEqual**(str1); //isEqual()은 static 메서드
Boolean result = p.**test**(str2); // str1과 str2가 같은지 비교한 결과를 반환

//inline
boolean result = Predicate.isEqal(str1).test(str2);
```

# 컬렉션 프레임워크와 함수형 인터페이스

### 함수형 인터페이스를 사용하는 컬렉션 프레임웍의 메서드(와일드카드 생략)

| 인터페이스 | 메서드 | 설명 |
| --- | --- | --- |
| Collection | boolean removeIf(Predicate<E> filter) | 조건에 맞는 요소를 삭제 |
| List | void replaceAll(UnaryOperator<E> operator) | 모든 요소를 변환하여 대체 |
| Iterable | void forEach(Consumer<T> action) | 모든 요소에 작업 action을 수행 |
| Map | V compute(K key, BiFunction<K,V,V> f) | 지정된 키 값에 작업 f 를 수행 |
|  | V computeIfAbsent(K key, Function<K,V> f) | 키가 없으면, 작업 f 수행 후 추가 |
|  | V computeIfPresent(K key, BiFunction<K,V,V> f) | 지정된 키가 있을 때, 작업 f 수행 |
|  | V merge(K key, BiFunction<V,V,V> f) | 모든 요소에 병합작업 f를 수행 |
|  | void forEach(BiConsumer<K, V> action) | 모든 요소에 작업 action을 수행 |
|  | void replaceAll(BiFunction<K, V, V> f) | 모든 요소에 치환작업 f를 수행 |

```java
list.forEach(i -> System.out.print(i+",")); // list의 모든 요소를 출력
list.removeIf(x-> x%2==0 || x%3==0); //2 또는 3의 배수를 제거
list.replaceAll(i->i*10) //모든 요소에 10을 곱

//map의 모든 요소를 {k,v}의 형식으로 출력
map.forEach((k, v) -> System.out.print("{"+k+" ,"+v+"},");
```

# 메서드 참조(method reference)

하나의 메서드만 호출하는 람다식은 ‘메서드 참조’로 더 간단히 할 수 있다.

| 종류 | 람다 | 메서드 참조 |
| --- | --- | --- |
| static 메서드 참조 | (x) → ClassName.method(x) | ClassName::method |
| 인스턴스 메서드 참조 | (obj, x) → obj.method(x) | ClassName::method |
| 특정 객체 인스턴스메서드 참조 | (x) → obj.method(x) | obj::method |

특정 객체 인스턴스메서드 참조는 거의 사용하지 않는다.

### static 메서드 참조

```java
Integer method(String s){ //그저 Integer.parseInt(String s)만 호출
	return Integer.parseInt(s);
}

//Lamda Expression
Function<String, Integer> f = (String s) -> Integer.parseInt(s);

//Method Reference
Function<String, Integer> f = Integer::parseInt;
```

# 생성자의 메서드 참조

- 생성자와 메서드 참조

```java
Supplier<MyClass> s = () -> new MyClass();

Supplier<MyClass> s = MyClass::new;

Function<Integer, MyClass> s = (i) -> new MyClass(i);
Function<Integer, MyClass> s = MyClass::new;

BiFunction<Integer, String, MyClass> s = (i,s) -> new MyClass(i);
BiFunction<Integer, String, MyClass> s = MyClass::new;
```

- 배열과 메서드 참조

```java
Function<Integer, int[]> f = x -> new int[x];
Function<Integer, int[]> f = int[]::new;
```