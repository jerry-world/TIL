# 스레드(Thread)

<aside>
💡 [프로세스와 스레드](https://www.notion.so/4f6c07f7f28745239ec0b5c68e4ee599) 내용을 읽고 먼저 프로세스와 스레드의 차이를 이해하자.

</aside>

# 스레드 이해하기

스레드를 구현하는 방법으로는 2가지가 존재한다.

1. Thread 클래스 상속
2. Runnable 인터페이스 구현(Better)

![Untitled](스레드(Thread)/Untitled.png)

인터페이스를 구현하냐, 부모클래스를 상속받아 처리하냐의 차이이다. 필요에 따라 선택해서 사용한다.

두가지 모두 run 메서드를 작성하여 스레드가 할 작업을 만들어준다. 작성된 스레드의 인스턴스를 생성하고, 인스턴스의 start메서드를 호출하여 해당 스레드를 동작시킨다.

물론 start 메서드를 호출했다고해서, 바로 실행되는 것은 아니다. 실행 가능상태가 되는 것이다.

실행가능 상태가 되면 OS 스케줄러가 이를 보고 실행시킨다.

OS 스케줄러는 여러 스레드가 동작중일 때, 스레드를 동작하는 것을 스케줄링해준다. 총 2개의 스레드가 동작중이고 각 스레드가 0과 1을 각각 출력하는 스레드라면, OS 스케줄러는 각 스레드를 번갈아 가면서 실행시킨다.

OS에 종속적이다…

출력하면 00001111001010100011100 이런식으로 예측불가하게 결과가 출력될 수 있다.

## 동작원리

![Untitled](스레드(Thread)/Untitled%201.png)

1. 메인메서드에서 스레드 인스턴스의 start 메서드를 호출했다.
2. 스레드가 동작하면서 새로운 call stack이 생성되고 start 메서드가 종료되었다.
3. 서로 독립적으로 작업할 수 있는 멀티 스레드가 되었다. 

## main 스레드

- main 메서드의 코드를 수행하는 스레드이다.

자바 애플리케이션이 구동되면서 자바 인터프리터가 최초에 실행시키는 우리가 흔히 작성하는 main 메서드가 결국엔 main 스레드가 된다. 위에서 언급한 바와같이 프로세스는 반드시 하나 이상의 스레드로 동작하는데, 자바는 그 대상이 최초 main 스레드가 된다.

- 스레드는 **사용자 스레드**와 **데몬 스레드**가 존재한다.

사용자 스레드는 main 스레드를 의미하고, 데몬 스레드는 사용자 스레드를 보조하는 즉 main 스레드를 보조하는 역할을 수행하는 스레드를 의미한다.

<aside>
💡 프로그램은 실행 중인 사용자 스레드가 하나도 없을 때 프로그램이 종료된다.
즉, 싱글 스레드 환경에서는 main 스레드가 모두 종료되면 프로그램은 종료된다.

</aside>

# 멀티스레드

## 멀티 스레드환경에서의 종료 시점

![Untitled](스레드(Thread)/Untitled%202.png)

위 그림처럼 멀티 스레드 환경에서 메인 스레드가 종료되더라도 우측 스레드가 살아있는 상태이기 때문에 프로그램은 종료되지 않는다. 우측 스레드가 종료되어야 프로그램이 종료된다.

Thread의 join메서드는 main 스레드가 join메서드를 호출한 스레드가 종료될 때 까지 기다린다.

```java
public class ThreadExample02 {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        Thread t1 = new ThreadEx();
        Thread t2 = new Thread(new RunnableEx());

        t1.start();
        t2.start();

        try {
            //Thread의 join메서드는 main 스레드가 join메서드를 호출한 스레드가 종료될 때 까지 기다린다.
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        long end = System.currentTimeMillis();

        System.out.println("소요 시간 : " + (end-start));

        System.out.println("t2 = " + t2);

    }
}
```

위의 소스코드 예제에서 t1과 t2의 join 메서드가 호출되어있기 때문에, main 스레드는 t1과 t2가 종료될 때까지 종료되지 않는다. 만약 해당 try-catch 구문이 없다면, main 스레드가 t1,t2를 기다려주지 않기 때문에, 소요시간이 이상하게 나올 것이다.

## 멀티 스레드 환경에서의 문맥교환

![Untitled](스레드(Thread)/Untitled%203.png)

멀티스레드환경에서는 OS 스케줄러에 의해 서로다른 스레드끼리 번갈아가면서 작업을 수행하는데, 이렇게 서로 다른 스레드로 작업 대상이 교체되는 것을 **문맥 교환(Context Switching)**이라고 부른다.

이 문맥교환 작업은 시간이 소요되기 때문에, 멀티 스레드 환경은 싱글 스레드 환경보다 작업 종료 시간이 보다 소요될 수 있다.

하지만, 두가지 작업을 동시에 수행할 수 있다는 장점을 갖고있다.

예로 파일 전송과 동시에 채팅을 수행할 수 있다는 장점을 가지고 있다.

## 스레드의 I/O 블락킹(blocking)

입출력시 작업이 중단되는 것을 I/O 블락킹이라고 부른다.

![Untitled](스레드(Thread)/Untitled%204.png)

싱글 스레드 환경에서는 I/O블락킹이 발생하면, 모든 작업이 대기하게 되지만, 멀티 스레드환경에서는 특정 스레드가 I/O 블락킹이 발생하더라도 다른 스레드가 동작하기때문에 **자원을 보다 효율적으로 사용**할 수 있다.

## 스레드의 우선순위(prioriy of thread)

작업의 중요도에 따라 스레드에 우선순위를 다르게 해서, 특정 스레드가 더 많은 작업시간을 갖도록할 수 있다.

스레드의 우선순위는 1~10 사이의 값으로 결정되며, 기본값은 5 이다.

상수로도 정의되어있다.

![Thread 클래스 내부에 존재하는 상수](스레드(Thread)/Untitled%205.png)

Thread 클래스 내부에 존재하는 상수

`Thread`의 `setPriority(int)` 메서드를 사용하여, 우선순위를 변경할 수 있으며, `getPriority`를 호출하면 우선순위 값을 반환받을 수 있다.

![Untitled](스레드(Thread)/Untitled%206.png)

하지만, 이 스레드의 우선순위는 희망사항 정도이다. OS 스케줄러에게 **“난 th1이 좀더 빨리 끝났으면 좋겠어.”** 라고 말하더라도 OS 스케줄러는 **“참고해볼게”** 하는 정도의 수준이라는 것이다…

## 스레드 그룹

서로 관련된 스레드를 그룹으로 묶어서 다루기 윈 것이다.

모든 스레드는 반드시 하나의 스레드 그룹에 포함되어 있어야한다. 즉, 별도의 스레드 그룹을 지정하지 않으면 main 스레드 그룹에 포함된다. 왜냐하면 자신을 생성한 스레드(부모 스레드)의 구룹과 우선순위를 상속 받기 때문이다.

```java
Thread(ThreadGroup group, String name)
Thread(ThreadGroup group, Runnable target)
Thread(ThreadGroup group, Runnable target, String name)
Thread(ThreadGroup group, Runnable target, String name, long stackSize)
```

```java
ThreadGroup getThreadGroup() //스레드 자신이 속한 스레드 그룹을 반환
void uncaughtException(Thread t, Throwable e)// 처리되지 않은 예외에 의해 스레드 그룹의 스레드가 실행이 종료되었을 때, JVM에 의해 이 메서드가 자동 호출된다.
```

## 데몬 스레드

일반 스레드(non-daemon thread)의 작업을 돕는 보조적인 역할을 수행한다.

일반 스레드가 모두 종료되면 자동적으로 종료된다.

가비지 컬렉터, 자동 저장, 화면 자동갱신 등에 사용된다.

데몬 스레드는 보편적으로 무한 루프로 작성한다. 일정기간 대기하도록 하고, 특정 작업을 만족할 때 작업을 수행할 수 있도록, 분기처리한다.

![Untitled](스레드(Thread)/Untitled%207.png)

Thread 메서드의 setDaemon 메서드를 호출해서 대상 스레드를 데몬 스레드로 만들 수 있다.

isDaemon메서드를 호출하면 대상 스레드가 데몬 스레드인지 확인할 수 있다.

<aside>
💡 반드시 start메서드가 호출되기 전에 setDaemon메서드를 호출해야한다.

</aside>

## 스레드의 상태

| NEW | 스레드가 생성되고 아직 start()가 호출되지 않은 상태 |
| --- | --- |
| RUNNABLE | start 메서드가 호출되어 실행 중 또는 실행 가능 한 상태 |
| BLOCKED | 동기화 블럭에 의해서 일시정지된 상태(lock이 풀릴 때까지 기다리는 상태 |
| WAITING,
TIME_WAITING | 스레드의 작업이 종료되지는 않았지만 실행가능하지 않은(unrunnable) 일시정지상태. TIME_WAITING은 일시정지시간이 지정된 경우를 의미 |
| TERMINATED | 스레드의 작업이 종료된 상태 |

![Untitled](스레드(Thread)/Untitled%208.png)

스레드가 start()메서드를 만나면 실행 대기상태가 된다.

실행 대기상태가 된 스레드는 자기 차례가 오면 스레드를 시작하고, 자기 차례가 종료되면 다시 실행 대기 상태에 줄서기를 한다.

만약 일시정지 상태가 될 수 있는 명령을 수행하면 일시 정지 했다가, 일시정지를 푸는 명령을 수행하면 다시 실행 대기 상태가 된다.

스레드가 종료되면 소멸된다.

## 스레드의 실행제어

스레드의 실행을 제어할 수 있는 메서드가 제공된다.

| 메서드 | 설명 |
| --- | --- |
| static void sleep(long millis)
static void sleep(long millis, int nanos) | 지정된 시간동안 스레드를 일시정지시킨다. 지정한 시간이 지나고 나면, 자동적으로 다시 실행대기 상태가 된다. |
| void join()
void join(long millis)
void join(long millis, int nanos) | 지정된 시간동안 스레드가 실행되도록 한다. 지정된 시간이 지나거나 작업이 종료되면 join()을 호출한 스레드로 다시 돌아와 실행을 계속한다. |
| void interrupt() | sleep()이나 join()에 의해 일시정지상태인 스레드를 깨워서 실행대기상태로 만든다. 해당 스레드에서는 InterruptedException이 발생함으로써 일시정지 상태를 벗어나게 된다. |
| void stop() | 스레드를 즉시 종료시킨다. |
| void suspend() | 스레드를 일시정지시킨다. resume()을 호출하여 다시 실행대시 상태가 된다. |
| void resume() | suspend()에 의해 일시정지상태에 있는 스레드를 실행대기상태로 만든다. |
| static void yield() | 실행 중에 자신에게 주어진 실행시간을 다른 스레드에게 양보하고 자신은 실행대기상태가 된다. |

<aside>
💡 `static` 키워드가 붙은 메서드는 자기 자신에게만 호출가능하다.

</aside>

### **sleep()**

현재 스레드를 지정된 시간동안 멈추게 한다.

```java
static void sleep(long millis)
static void sleep(long millis, int nanos)
```

예외 처리를 반드시 해주어야한다.(`InterruptedException`이 발생하면 깨어남)

```java
try{
	Thread.sleep(1000);
}catch(InterruptedException e){}
```

`sleep()` 은 시간이 종료되거나, `interrupted()` 가 호출하면 즉, 누군가 깨우면 끝난다.

`InturrptedExeption`은 누군가 깨워서 발생한 것이지 문제가 발생한  것은 아니다.

그런데 매번 이렇게 try-catch문을 작성해서 InterruptedException 예외를 처리해주기 불편하다.

그래서 아래와 같이 별도로 메서드를 작성하기도 한다.

```java
something(){
	delay(1000);
}

void delay(long millis){
	try{
		Thread.sleep(millis);
	}catch(InterruptedException e){}
}
```

![Untitled](스레드(Thread)/Untitled%209.png)

### **interrupt()**

**대기상태(WAITING)**인 스레드를 **실행 대기 상태(RUNNABLE)**로 만든다.

```java
void interrupt() // 스레드의 interrupted상태를 false에서 true로 변경
boolean isInterrupted() // 스레드의 interrupted 상태를 반환
static boolean interrupted() // **현재** 스레드의 interrupted 상태를 알려주고, false로 초기화
```

### **suspend(), resune(), stop()**

스레드의 실행을 일시정지, 재개, 완전정지 시킨다.

```java
@Deprecated

void suspend() //스레드를 일시정지 시킨다.
void resume() //suspend()에 의해 일시정지된 스레드를 실행대기상태로 만든다.
void stop() //스레드를 즉시 종료시킨다.
```

![Untitled](스레드(Thread)/Untitled%2010.png)

<aside>
💡 데드락(교착상태)의 문제가 발생할 수 있어 Deparecated 되었다.

</aside>

직접 구현할 수도 있다.

```java
public class MyThread implements Runnable{

    private Thread th;
    volatile private boolean suspended = false; // volatile 쉽게 바뀌는 변수 ( cpu에 있는 코어들이 메모리에있는 값을 복사해서 사용하지 않고 메모리 원본을 그대로 사용)
    volatile private boolean stopped = false;

    public MyThread(String name) {
        th = new Thread(this, name);
    }

    void start(){
        th.start();
    }

    void stop(){
        stopped=true;
    }

    void suspend() {
        suspended = true;
    }

    void resume() {
        suspended = false;
    }

    @Override
    public void run() {
        while(!stopped){
            if(!suspended){
                //do something
            }
        }
    }
}
```

### **join()**

지정된 시간동안 특정 스레드가 작업하는 것을 기다린다.

```java
void join() // 작업이 모두 끝날때까지 대기
void join(long millis) //천분의 일초동안 대기
void join(long millis, int nanos) // 천분의 일초 + 나노초 동안 대기
```

join을 언제쓰는지 예시를 기억해보자.

아래의 예시를 떠올려보자.

![Untitled](스레드(Thread)/Untitled%2011.png)

### yield()

남은 시간을 다음 스레드에게 양보하고, 자신(현재 스레드)은 실행대기한다.

![Untitled](스레드(Thread)/Untitled%2012.png)

<aside>
💡 static 메서드로 자기 자신한테만 사용할 수 있는 메서드

</aside>

suspend() 가 true면 false일때까지 계속 while이 돌껀데, 다시 resume()될때까지 이렇게 계속 돌면서 대기하고 있는 상태를 **busy-waiting** 상태라고 한다. 이럴때는 차라리 다른 스레드에게 양보를 하는편이 좋다.

yield()와 interrupte()를 적절히 사용하면, 응답성과 효율을 높일 수 있다.

```java
public class MyThread implements Runnable{

    private Thread th;
    volatile private boolean suspended = false; // volatile 쉽게 바뀌는 변수 ( cpu에 있는 코어들이 메모리에있는 값을 복사해서 사용하지 않고 메모리 원본을 그대로 사용)
    volatile private boolean stopped = false;

    public MyThread(String name) {
        th = new Thread(this, name);
    }

    void start(){
        th.start();
    }

    void stop(){
        stopped=true;
        th.interrupt();
    }

    void suspend() {
        suspended = true;
        th.interrupt();
    }

    void resume() {
        suspended = false;
    }

    @Override
    public void run() {
        while(!stopped){
            if(!suspended){
                //do something
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {}
            }else{
                Thread.yield();
            }
        }
    }
}
```

yield()도 일전의 스레드 우선순위처럼 OS 스케줄러에게 통보하는 식이다. OS 스케줄러에게 **“내가 스레드의 작업시간을 양보할테니 다른 스레드가 일을 할 수 있게 처리해주면 좋을 것 같아.”** 라는 식으로 말하면 OS 스케줄러는 **“그래 참고해볼게.”** 정도 인 것이다.

쓸때와 안쓸때가 성능에 큰 영향을 미치진 않지만, 미약하게나마…

## 스레드의 동기화

Thread에서 동기화란 한 Thread가 진행중인 작업을 다른 Thread에서 간섭하지 못하도록 막는 것이다.

동기화하려면 간섭받지 안아야 하는 문장들을 **임계 영역(Critical Section)**으로 설정해야 한다.

다른 Thread는 누군가 임계 영역에 접근하고 있으면 접근할 수 없다.

임계영역은 **락을 얻은 단 하나의 Thread만 출입**이 가능하다.

락은 객체마다 단 한개만 가지고 있는다.

```java
// 메서드 전체를 임계영역으로 지정
pbulic synchronized void calcSum(){ ... } // ... 영역이 임계영역

// 특정한 영역을 임계 영역으로 지정
synchronized(객체의 참조변수) { ... } 
```

임계 영역은 한번에 1개의 Thread만 출입할 수 있기 때문에 영역을 최소화해야 한다. 임계 영역이 크면 성능이 떨어진다. 때문에, 메서드 전체보단 특정한 영역을 임계 영역으로 지정하는 편이 좋다.

<aside>
💡 은행 계좌에서 출금 로직을 생각해보자

</aside>

### **wait() 과 notify()**

동기화의 효율을 높이기 위해 wait(0, notify()를 사용

Object클래스에 정의되어 있으며, 동기화블록 내에서만 사용할 수 있다.

동기화를 보다 효율적으로 하기 위해 존재하는 메서드이다. 

```java
wait() // 객체의 lock을 풀고 스레드를 해당 객체의 waiting pool에 넣는다.
notify() // waiting pool에서 대기중인 스레드 하나를 깨운다.
notifyAll() // waiting pool에서 대기중인 스레드 모두를 깨운다.
```

**예시**

테이블에 두명의 손님이 있고, 손님은 특정 음식을 먹는다. 요리사는 특정 음식을 만들고, 테이블에 최대 5개까지 음식이 올라갈 수 있다.

**요리사**

음식을 계속 테이블에 추가

```java
package thread.resturant;

public class Cooker implements Runnable{
    private Table table;

    public Cooker(Table table) {
        this.table = table;
    }

    @Override
    public void run() {
        while (true) {
            int idx = (int) (Math.random() * table.dishNum());
            table.add(table.dishNames[idx]);
						try {
                Thread.sleep(10);
            } catch (InterruptedException e) {}
        }
    }
}
```

**손님**

테이블에 놓인 음식을 계속 소비

```java
package thread.resturant;

public class Customer implements Runnable{
    private Table table;
    private String food;

    public Customer(Table table, String food) {
        this.table = table;
        this.food = food;
    }

    @Override
    public void run() {
        while (true) {
            try {Thread.sleep(1000);} catch (InterruptedException e) {}

            String name = Thread.currentThread().getName();

            table.remove(food);
            System.out.println(name + " ate a " + food);

        }
    }
}
```

**테이블**

추가되거나 소비될 수 있는 음식이 올라가는 테이블

**add()**

음식은 최대 6개까지 테이블에 올라갈 수 있다.

음식을 계속 추가하다가 최대 음식 수에 도달하면 `wait()` 메서드를 호출한다. → 요리사가 waiting pool로 진입

**remove()**

음식은 테이블에 놓여진 음식 수 만큼 소비될 수 있다.

각 두명의 소비자는 도너와 햄버거를 계속 소비를 하는데, 만약 테이블에 놓인 접시가 없으면 소비를 멈추고 기다린다.

```java
package thread.resturant;

import java.util.ArrayList;
import java.util.List;

public class Table {
    public String[] dishNames = {"burger", "donut", "donut"};
    final int MAX_FOOD = 6;
    private List<String> dishes = new ArrayList<>();

    public synchronized void add(String dish) {
        while (dishes.size() >= MAX_FOOD) {
            String name = Thread.currentThread().getName();
            System.out.println(name + " is waiting");
            try {
                wait(); // 음식이 가득찼으니 요리사는 쉬겠습니다
                Thread.sleep(500);
            } catch (InterruptedException e) {
            }
        }
        dishes.add(dish);
        notify(); // 손님 음식 다됐어요~
        System.out.println("Dishes:" + dishes.toString());
    }

    public void remove(String dishName) {
        synchronized (this) {
            String name = Thread.currentThread().getName();

            while (dishes.size() == 0) {
                System.out.println(name + " is waiting.");
                try {
                    wait(); //음식이 없어요 손님.
                    Thread.sleep(500);

                } catch (InterruptedException e) {
                }
            }

            while (true) {
                for (int i = 0; i < dishes.size(); i++) {
                    if (dishName.equals(dishes.get(i))) {
                        dishes.remove(i);
                        notify(); // 요리 다먹었어요 또만들어주세요
                        return;
                    }
                }

                try {
                    System.out.println(name + " is waiting.");
                    wait(); // 음식이 없네요 기다려주세요.
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                }
            }
        }
    }

    public int dishNum() {
        return dishNames.length;
    }
}
```

```java
public class RestaurantExample {
    public static void main(String[] args) throws Exception {
        Table table = new Table();

        new Thread(new Cooker(table), "COOK").start();
        new Thread(new Customer(table, "donut"), "CUST1").start();
        new Thread(new Customer(table, "burger"), "CUST2").start();

        Thread.sleep(10000);
        System.exit(0);
    }
}
```

<aside>
💡 위의 소스를 보면, **notify** 와 **wait** 가 누구를 지칭하는 것이 없다. 그냥 기다리고 누군가를 깨우는 행위를 하는 것이다. 그 누군가는 아무 스레드가 된다. 이러한 부분을 개선하고자 `Lock & Condition` 이 존재한다.

</aside>