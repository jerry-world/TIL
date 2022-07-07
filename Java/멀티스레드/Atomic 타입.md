# Atomic 타입

Java에서 변수의 동시성 문제를 해결하기 위해서, `java.util.concurrent.atomic` 패키지가 존재한다.

![Untitled](Atomic%20타입/Untitled.png)

이 Atomic 변수에 대해서 알아보려고한다.

프로그래밍을 하다보면 **멀티 스레드**를 사용하는 경우가 빈번하다. 멀티 스레드 프로그래밍에서 가장 중요하게 고려해야하는 부분이 `동시성 문제` 이다. 

이 Atomic 변수는 멀티 스레드 환경에서 최신 데이터 임을 보장하기 위해 제공되는 변수이다.

Atomic의 뜻을 풀이하면 원자라는 의미를 가지고 있는데, 이는 `더 이상 쪼갤 수 없는 가장 단위` 로 해석할 수 있다.

<aside>
💡 In programming, an atomic action is one that effectively happens all at once.

An atomic action cannot stop in the middle: it either happens completely, or it doesn't happen at all.

</aside>

인터넷 쇼핑몰에서 물건을 살 때의 데이터베이스에서는 결제 정보와 결제에 따른 상품 수량 감소 액션이 함께 맞물려 진행되어야 한다. 만약 결제는 되었는데 수량이 감소되지 않았다면, 다른 사용자는 수량이 존재하는 줄 알고, 상품을 구매하게 되어, 무결성의 문제가 발생한다.

단 하나의 스레드가 순차적으로 업무를 수행한다면, 이런 문제는 발생하지 않을 것이다. 그런데 멀티 스레드 환경에서 여러 스레드가 동시에 상품을 감소시키는 등의 작업을 수행하면 위와 같은 문제가 발생할 수 있다.

때문에 위의 예시처럼 액션이 여러개가 분리(비 원자성)되어 하나는 실행되고 하나는 실행되지 않는 문제를 없게 하려면 Atomic 이 필요하다.

Java 에서는 동시성을 보장하기 위해 `volatile`, `synchronized`, `atomic` 을 지원한다.

그중 이 atomic 타입은 단일 변수에 대해 원자성 작업을 지원한다. 참조 타입 또는 원시타입에 대한 Wrapper Class 형태이며, 사용 시 내부적으로 `CAS(Compare-And-Swap)` 알고리즘을 사용하여 lock 없이 동기화를 처리할 수 있다.

# Atomic 타입의 변수 사용해보기

Atomic 타입을 사용하는 방법은 간단하다. 일반적으로 알고 있는 참조타입 앞에 Atomic만 붙혀주면된다.

![Untitled](Atomic%20타입/Untitled%201.png)

그러면 동시성 문제가 발생하는 예시와 함께 Atomic 타입의 동작 원리를 이해해보자.

```java
private static int num = 0;
    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicInteger = new AtomicInteger(0);

        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                num++;
                atomicInteger.incrementAndGet();
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                num++;
                atomicInteger.incrementAndGet();
            }
        });

        thread1.start();
        thread2.start();

        Thread.sleep(5000);

        System.out.println("atomicInteger = " + atomicInteger.get());
        System.out.println("num = " + num);
    }
```

위의 예시를 살펴보자. Atomic은 동시성문제를 해결해준다고 했다. 각각의 thread1과 thread2는 for문을 돌면서 int 타입 num을 증가시키고, AtomicInteger 타입 atomicInteger를 증가시킨다.

![Untitled](Atomic%20타입/Untitled%202.png)

수행결과 원시타입 num은 예상과 다르게 161,012 값이 출력했다. 이문제는 가령 thread1와 thread2가 값이 동시에 증가되면서 발생할 수 있겠다.

예를 들자면, thread1번이 for문을 돌았을 때의 num 값이 15333이고, 때마침 thread1이 값을 증가시키기전에 thread2번이 for문을 돌았을 때의 num 값이 15333이어서 둘다 증가시켜도 동일하게 15334로 마친 것이다. 다른 경우도 있겠지만 여하튼, 이는 동기화가 지켜지지 않아 발생한 문제이다.

사용하고 있는 대상의 num 값이 항상 최신의 것을 유지하고 있다면 이런 동시성 문제는 발생하지 않을 것이다.

그래서 사용한 것이 Atomic 타입의 변수인 AtomicInteger이다. 위에서 언급한 바와 같이 Atomic타입은 CAS(Compare And Swap) 알고리즘을 사용하여, 데이터를 동기화해준다. 즉, 항상 최신 데이터를 유지할 수 있도록 도와준다.

그러면 AtomicInteger에서 어떻게 CAS 알고리즘을 사용하고 있는지 알아보자.

위의 예제에서 AtomicInteger의 값을 증가시키기 위해, getAndIncrement를 사용하였다. getAndIncrement 메서드는 getAndAddInt 메서드를 호출하는데, getAndAddInt 메서드를 보면 원자적으로 전달받은 주어진 값을 증가시킨다고 되어있다.

여기서 getInVolatile은 CPU 캐시가 아닌 메인 메모리의 값을 가지고 오는 것이다. 해당 메서드의 내용을 살펴보면 메인 메모리의 값을 가져오고 가져온 값이 입력받은 값과 동일한지 비교후, 일치 할때까지 do-while문을 반복하여 일치한 경우에 값을 증가한 뒤 리턴하게 되어있다.

이말인즉슨, 전달받은 값이 최신이면 값을 증가시키겠다는 뜻으로 돌려 해석할 수 있다.

![Untitled](Atomic%20타입/Untitled%203.png)

![Untitled](Atomic%20타입/Untitled%204.png)

![Untitled](Atomic%20타입/Untitled%205.png)

위의 그림은 예시를 그림으로 그려본 내용이다.(내가 이해한 대로..)

Thread-1과 Thread-2가 AtomicInteger타입의 객체에 대해 incrementAndGet을 for문을 통해 각 10만번씩 호출하고 있다. 서로 다른 스레드 즉, 멀티 스레드 환경에서 동일한 AtomicInteger 대상의 값을 매번 증가 요청을 하고있다. 이런 비 원자성 상황에서 원자성을 보장하기 위해, incrementAndGet 메서드는 getAndAddInt 메서드를 호출하고 이 getAndAddInt 메서드는 메인 메모리에 작성된 값이 요청된 값과 일치하는지 확인하고 값을 증가시킨다.

다른 타입의 Atomic 변수들도 위와 비슷하다. 가령 AtomicBoolean의 경우에는 들어온 값이 true일 경우 현재 메인 메모리상의 값이 true가 맞는지 확인하고 값을 변경해준다.

# CAS 알고리즘

CAS 알고리즘은 말그대로 비교하고 교체하는 알고리즘이다.

CAS 알고리즘을 검색해보면 아래의 그림이 많이 나온다.

![Untitled](Atomic%20타입/Untitled%206.png)

알고있는 현재 값과 비교대상(실제값) 값 그리고 변경 값 3가지가 주어졌을 때, 알고있는 현재 값과 비교대상 값이 일치하는 경우에 변경할 값으로 값을 swap한다. 그렇지 않으면 변경되지 않는다. 

이렇게 한번 확인하고 변경하는 방식의 패턴을 ‘체크-액트’ 패턴이라고 부르기로 약속했다.

여기서 현재값과 실제값이 의미하는 바가 메인메모리 상의 값과 CPU 캐시 상의 값이다. CPU 캐시에 값이 올려진 찰나에 메인 메모리 상의 값이 바뀌면 두 값을 일치하지 않는다. 그래서 그럴 경우 다시 재시도하여 항상 최신의 값을 유지한 상태로 값을 증가할 수 있게 해준다.

먼가 내용도 전문적이지 않고, 두서없이 작성된 것 같다. 

결과적으로는 Atomic 변수는 CAS 알고리즘을 통해 항상 최신의 것을 보장하도록(원자성 보장) 도와주고 이를 통해, 여러 스레드가 동시에 해당 변수에 접근하여 작업하더라도, 항상 동시성 문제를 해결하면 작업될 수 있다.
