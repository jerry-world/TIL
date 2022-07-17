# Stream

다양한 데이터 소스(컬렉션, 배열 등)를 표준화된 방법으로 다루기 위한 것

데이터의 연속적인 흐름

# 스트림 이해하기

## 스트림 만들기

```java
Stream<Integer> intStream=list.stream(); //컬렉션
        Straam<String> strStream=Stream.of(new String[]{"a","b","c"}; //배열
        Stream<Integer> evenStream=Stream.iterate(0,n->n+2); // 0,2,4,6,8
        Stream<Double> randomStream=Stream.generate(Math::random); // 람다식
        IntStream intStream=new Random().ints(5); //난수 스트림(크기가 5)
```

> 💡 데이터소스 → 스트림만들기 → 중간연산(여러번 가능) → 최종연산

## 스트림이 제공하는 기능 - 중간 연산과 최종 연산

- 중간 연산 - 연산결과가 스트림인 연산. 반복적으로 적용 가능
- 최종 연산 - 연산결과가 스트림이 아닌 연산. 단 한번만 적용 가능(스트림의 요소를 소모)

```java
stream.distinct().limit(5).sorted().forEach(System.out::println)
```

```java
String[]strArr={"dd","aaa","CC","cc","b"};
        Stream<String> stream=Stream.of(strArr); // 문자열 배열이 소스인 스트림
        Stream<String> filteredStream=stream.filter(); // 걸러내기 (중간연산)
        Stream<String> distinctedStream=stream.distinct(); // 중복제거 (중간연산)
        Stream<String> sortedStream=stream.sort(); // 정렬(중간연산)
        Stream<String> limitedStream=stream.limit(5); // 정렬(중간연산)
        int total=stream.count(); //요소 개수 세기 (최종연산)
```

## 스트림의 특징

- **스트림은 데이터 소스로부터 데이터를 읽기만할 뿐 변경하지 않는다.**

```java
List<Integer> list=Arrays.asList(3,1,5,4,2);
        List<Integer> sortedList=list.stream().sroted().collect(Collectors.toList());
        System.out.println(list); // [3,1,5,4,2]
        System.out.println(sortedList); //[1,2,3,4,5]
```

- **스트림은 Iterator처럼 일회용이다.(필요하면 다시 스트림을 생성해야 함). 최종 연산을 수행하면 스트림은 요소를 소모한다.**

```java
strStream.forEach(System.out::println); //모든 요소를 화면에 출력(최종연산)
        int numOfStr=**strStream.count()**; //**에러**. 스트림이 이미 닫혔음.
```

- **최종 연산 전까지 중간연산이 수행되지 않는다. (지연된 연산)**

```java
IntStream intStream=new Random().ints(1,46); //1~45범위의 **무한 스트림**
        intStream.**distinct().limit(6).sorted()**         //**중간 연산**
        .forEach(i->System.out.print(i+",");  //최종 연산
```

- **스트림은 작업을 내부 반복으로 처리한다.**

```java
for(String str:strList)
        System.out.println(str);

        stream.forEach(System.out::println);
```

⇒ 성능은 다소 떨어질 수 있지만, 코드의 간결성이 높아진다.

- **스트림의 작업은 병렬로 처리 - 병렬스트림**

멀티스레드로 병렬처리

```java
Stream<String> strStream=Stream.of("dd","aaa","CC","cc","b");
        int sum=strStream.**parallel()** // **병렬 스트림**으로 전환(속성만 변경)
        .mapToInt(s->s.length()).sum(); // 모든 문자열의 길이의 합
```

- **기본형 슽릠 - IntStream, LongStream, DoubleStream**

오토박싱&언박싱의 비효율이 제거됨(Stream<Integer> 대신 IntStream을 사용)

숫자와 관련된 유용한 메서드를 Stream<T>보다 더 많이 제공

데이터 소스가 기본형일때 사용함

# 스트림 만들기

## 컬렉션 스트림 만들기

Collection인터페이스의 stream()으로 컬렉션을 스트림으로 변환

```java
Stream<E> stream(); //Collection인터페이스 메서드
```

```java
List<Integer> list=Arrays.asList(1,2,3,4,5);
        Stream<Integer> intStream=list.stream(); //list를 스트림으로 변환

        intStream.forEach(System.out::println); // 12345 (최종연산)
```

## 배열 스트림 만들기

- 객체 배열로부터 스트림 생성하기

```java
Stream<T> Stream.of(T..values)
        Stream<T> Stream.of(T[])
        Stream<T> Arrays.stream(T[])
        Stream<T> Arrays.stream(T[],int startInclusive,int endInclusive)
```

```java
Stream<String> strStream=Stream.of("a","b","c");;
        Stream<String> strStream=Stream.of(new String[]{"a","b","c"});
        Stream<String> strStream=Arrays.stream(new String[]{"a","b","c"});
        Stream<String> strStream=Arrays.stream(new String[]{"a","b","c"},0,3);
```

- 기본형 배열로부터 스트림 생성하기

```java
**IntStream**IntStream.of(int...values)
        **IntStream**IntStream.of(int[])
        **IntStream**Arrays.sream(int[])
        **IntStream**Arrays.sream(int[],int startInclusive,int endInclusive)
```

기본형 스트림을 사용하면, 스트림 대상이 어떤 기본형인지 알고 있기 때문에 제공되는 메서드가 많다.

## 임의의 수 스트림 만들기 - Random

- 난수를 요소로 갖는 스트림 생성하기

```java
IntStream intStream=new Random().ints(); //무한 스트림
        intStream.**limit(5)**.forEach(System.out::println);

        IntStream intStream=new Random().ints(5).forEach(System.out::println);
```

- 지정된 범위의 난수를 요소로 갖는 스트림을 생성하는 메서드 (Random클래스)

```java
//무한 스트림
IntStream ints(int begin,int end)
        LongStream longs(long begin,long end)
        DoubleStream doubles(double begin,double end)

//유한 스트림
        IntStream ints(long streamSize,int begin,int end)
        LongStream longs(long streamSize,long begin,long end)
        DoubleStream doubles(long streamSize,double begin,double end)
```

- 특정 범위의 정수를 요소로 갖는 스트림 생성하기(IntStream, LongStream)

```java
IntStream mIntSTream.range(int begin,int end)//end 미포함
        IntStream mIntSTream.rangeClosed(int begin,int end)//end 포함
```

## 람다식으로 스트림 만들기 - interate(), generate()

- 람다식을 소스로 하는 스트림 생성하기

```java
static<T> Stream<T> iterate(T seed,UnaryOperateor<T> f) // 이전 요소에 종속적
static<T> Stream<T> generate(Supplier<T> s) // 이전 요소에 독립적
```

- iterate()는 이전 요소를 seed로 해서 다음 요소를 계산한다.

```java
Stream<Integer> evenStream=Stream.iterate(0,n->n+2);
```

| n → n+2 |
| --- |
| 0 → 0+2 |
| 2 → 2+2 |
| 4 → 4+2 |
| 6 → 6+2 |

- generate()는 seed를 사용하지 않는다.

```java
Stream<Double> randomStream=Stream.generate(Math::random);
        Stream<Integer> oneStream=Stream.generate(()->1);
```

## 파일 스트림과 비어있는 스트림 만들기

- 파일을 소스로 하는 스트림 생성하기

```java
Stream<Path> Files.list(Path dir) //Path는 파일 또는 디렉터리
```

```java
Stream<String> Files.lines(Path path)
        Stream<String> Files.lines(Path path,Charset cs)
        Stream<String> lines() // BufferedReader 클래스의 메서드
```

- 비어있는 스트림 생성하기

```java
Stream emptyStream=Stream.empty(); //empty()는 빈 스트림을 생성해서 반환한다.
        long count=emptyStream.count(); //count의 값은 0
```

# 스트림의 연산

중간연산은 연산 결과가 스트림이며, n번 수행가능하다.

최종연산은 연산 결과가 스트림이 아닌 연산이며, 1번 수행가능하다.

## 중간연산

| 중간 연산                                                   | 설명                                               |
|---------------------------------------------------------|--------------------------------------------------|
| Stream<T> distinct()                                    | 중복을 제거                                           |
| Stream<T> filter(Predicate<T> predicate)                | 조건에 안맞는 요소 제외                                    |
| Stream<T> limit(long maxSize)                           | 스트림의 앞부분부터 일부를 잘라낸다.                             |
| Stream<T> skip(long n)                                  | 스트림의 앞부분 일부를 건너뛴다.                               |
| Stream<T> peek(Consumer<T> action)                      | 스트림의 요소에 작업을 수행한다. 작업 중간에 잘 처리되고있는지 확인할 때 사용함.   |
| Stream<T> sorted()| 스트림의 요소를 정렬한다. Comparator를 작성하여 정렬 기준을 지정할 수 있다. |
Stream<T> sorted(Comparator<T> comparator)              | 스트림의 요소를 정렬한다. Comparator를 작성하여 정렬 기준을 지정할 수 있다. |
| Stream<R> map(Function<T,R> mapper)                     | 스트림의 요소를 변환한다.                                   |
| DoubleStream mapToDuble(ToDoubleFunction<T> mapper)     | 스트림의 요소를 변환한다.                                   |
| IntStream mapToInt(ToIntFunction<T> mapper)             | 스트림의 요소를 변환한다.                                   |
LongStream mapToLing(ToLongFunction<T> mapper)          | 스트림의 요소를 변환한다.                                   |
| Stream<R> flatMap(Function<T,R> mapper)                 | 스트림의 요소를 변환한다.                                   |
| DoubleStream flatMapToDuble(ToDoubleFunction<T> mapper) | 스트림의 요소를 변환한다.                                   |
| IntStream flatMapToInt(ToIntFunction<T> mapper)         | 스트림의 요소를 변환한다.                                   |
| LongStream flatMapToLing(ToLongFunction<T> mapper       | 스트림의 요소를 변환한다.                                   |

> 💡 중간 연산의 핵심은 map과 flatMap이다.

### 스트림 자르기 - skip(), limit()

```java
Stream<T> skip(long n) //앞에서부터 n개 건너뛰기
        Stream<T> limit(long maxSize) // maxSize 이후의 요소는 잘라냄
```

```java
IntStream intStream=IntStream.rangeClosed(1,10); // 1 2 3 (**4 5 6 7 8)** 9 10
        intStream.skip(3).limit(5).forEach(System.out::print) // 45678
```

### 스트림의 요소 걸러내기 - filter(), distinct()

```java
Stream<T> filter(Predicate<? super T>predicate) //조건에 맞지 않는 요소 제거)
        Stream<T> distinct() //중복 요소 제거
```

```java
IntStream intStream=Instream.of(1,2,2,3,4,5,6,6,6);
        intStream.distinct().forEach(System.out::print); // 123456
```

```java
IntStream intStream=IntStream.rangeClosed(1,10); // 1 2 3 4 5 6 7 8 9 10
        intStream.filter(i->i%2==0).forEach(System.out::print); //2 4 6 8 10 필터 조건에 만족하는 대상들을 제외한 나머지는 제거)
```

```java
IntStream intStream=IntStream.rangeClosed(1,10); // 1 2 3 4 5 6 7 8 9 10
//2의 배수와 3의배수를 필터링하는 연산
        intStream.filter(i->i%2!=0).filter(i->i%3!=0).forEach(System.out::print) // 1 5 7
//아래처럼 사용할 수 있음
        intStream.filter(i->i%2!=0&&i%3!=0).forEach(System.out::print)
```

### 스트림 정렬하기 - sorted()

```java
Stream<T> sorted() //스트림 요소의 기본 정렬(Comparable)로 정렬
        Stream<T> sorted(Comparator<? super T>comparator) //지정된 Comparator로 정렬
```

| 문자열 스트림 정렬 방법                                                       | 출력 결과 |
|---------------------------------------------------------------------| --- |
| strStream.sorted()//기본 정렬                                           |CCaaabccdd |
| strStream.sorted(Comparator.naturalOrder())                         |CCaaabccdd |
| strStream.sorted((s1, s2) → s2.compareTo(s1))                       |CCaaabccdd |
| strStream.sorted(String::CompareTo)                                 | CCaaabccdd |
| strStream.sorted(Comparator.reverseOrder()) //역순 정렬                 | ddccbaaaCC |
| strStream.sorted(Comparator.<Strring>naturalOrder().reversed())     | ddccbaaaCC |
| strStream.sorted(String.CASE_INSENSITIVE_ORDER) // 대소문자 구분 안함       | aaabCCccdd |
| strStream.sorted(String.CASE_INSENSITIVE_ORDER.reversed())          | ddCCccbaaa |
| strStream.sorted(Comparator.comparing(String::length)) // 길이 순 정렬  | bddCCccaaa | 
|  strStream.sorted(Comparator.comparingInt(String::length)) //no오토박싱 | bddCCccaaa |
| strStream.sorted(Comparator.comparing(String::length).reversed())   | aaaddCCccb |

```java
**static**Comparator<String> CASE_INSENSITIVE_ORDER=new CaseInsensitiveComparator();
```

- Comparator의 comparing()으로 정렬 기준을 제공

```java
strStream.sorted(Function<T, U> keyExtractor)
        strStream.sorted(Function<T, U> keyExtractor,Comparator<U> keyComparator)
```

```java
studentStream.sorted(Comparator.comparing(s->s.getBean()).forEach(System.out::println);
        studentStream.sorted(Comparator.comparing(Student::getBan()).forEach(System.out::println);

// Comparator.comparing(Student::getBean() 의 반환타입은 Comparator 이다
```

- 추가 정렬 기준을 제공할 때는 thenComparing()을 사용

```java
thenComparing(Comparator<T> other)
        thenComparing(Function<T, U> keyExtractor)
        thenComparing(Function<T, U> keyExtractor,Comparator<U> keyComparator)
```

```java
studentStream.sorted(Comparator.comparing(Student::getBan) // 반별로 정렬
        .thenComparing(Student::getTotalScore) // 총점 별로 정렬
        .thenComparing(Student::getName)) //이름별로정렬
        .forEach(System.out::println);
// -> 반으로 정렬 후, 총점으로 정렬하고 마지막 이름으로 정렬함
```

### 스트림의 요소 변환하기 - map()

```java
Stream<R> map(Function<? super T,?extends R>mapper) //Stream<T> -> Strea<R>
```

```java
**Stream<File>**fileStream=Stream.of(new File("Ex1.java"),new File("Ex1"),
        new File("Ex1.bak"),new File("Ex2.java"),
        new File("Ex1.txt"));

        **Stream<String>**filenameStream=fileStream**.map(File::getName);** //파일 스트림을 파일명 스트림으로 변경
        filenameStream.forEach(System.out::println);//파일 명 모두 출력
```

![Untitled](Stream/Untitled.png)

ex) 파일 스트림(Stream<File>)에서 파일 확장자(대문자)를 중복없이 뽑아내기

```java
fileStream.map(File::getName) // 파일스트림 가져와서
        .filter(s->s.indexOf('.')!=-1) // 확장자 없는 것들 필터링 하고
        .map(s->s.subString(s.indexOf('.')+1) // '.' 기준으로 그 이후의 내용으로 자른 대상 확장자 스트림 만들고
        .map(String::toUpperCase) // 확장자 스트림의 모든 요소를 대문자로 변환한다음
        .distinct() // 그 확장자 스트림 요소에서 중복제거하고
        .forEach(System.out::print)//출력 : (결과) JAVABAKTXT
```

### 스트림의 요소를 소비하지 않고 엿보기 - peek()

```java
Stream<T> peek(Consumer<? super T>action) //중간연산(스트림을 소비X)
        void forEach(Consumer<? super T>action) //최종연산(스트림을 소비O)
```

### 스트림의 스트림을 스트림으로 변환 - flatMap()

```java
Stream<String[]>strArrStrm=Stream.of(new String[]{"abc","def","ghi"},
        new String[]{"ABC","GHI","JKLMN"});
```

```java
Stream<**Stream<String>**>strStrStrm=strArrStrm.**map(Arrays::stream)**;
//Stream<**Stream<String>**> strStrStrm = strArrStrm.**map(arr -> Arrays.stream(arr))**;
```

![Untitled](Stream/Untitled%201.png)

```java
Stream<**String**>strStrStrm=strArrStrm**.flatMap(Arrays::stream**); // Arrays.stream(T[])
```

![Untitled](Stream/Untitled%202.png)

## 최종 연산

| 최종 연산                                                        | 설명 |
|--------------------------------------------------------------| --- |
| void forEach(Consumer<? super T> action
void forEachOrdered(Consumer<? super T> action               | 각 요소에 지정된 작업 수행
Ordered는 요소의 순서를 유의하여 작업한다.                                  |
| long count()                                                 | 스트림 요수의 개수를 반환 |
| Optional<T> max(Comparator<? super T> comparator)
Optional<T> min(Comparator<? super T> comparator)            | 스트림의 최대값/최소값을 반환
정렬 기준에 짜라 max와 min을 반환한다.                                    |
| Optional<T> findAny() //병렬 처리할 때 사용 filter와 함께 사용함|
Optional<T> findFirst() // 직렬 처리할 때 사용                       | 스트림의 요소 아무거나 하나를 반환하거나, 첫번째 요소를 반환 |
| boolean allMatch(Predicate<T> p)                             | 주어진 조건을 모두 만족하는지 여부|
| boolean anyMatch(Predicate<T> p)                             | 하나라도 만족하는지 여부|
boolean noneMatch(Predicate<T> p)                            | 하나도 만족하지 않는지 여부 |
| Object[] to Array() A[] toArrayt(IntFunction<A[]> generator) | 스트림의 모든 요소를 배열로 반환 |
| Optional<T> reduce(BinaryOperator<T> accumulator) |스트림의 요소를 하나씩 줄여가면서(리듀싱) 계산한다.|             

T reduce(T identity, BinaryOperator<T> accumulator)
U reduce(U identity, BiFunction<U,T,U> accumulator,
BinaryOperator<U> combiner) |
예를들어 모든 요소를 sum할 때, 하나씩 줄여가면서 sum하는 것(?) |
| R collect(Collector<T,A,R> collector)
R collect(Supplier<R> supplier, BiConsumer<R,T>
accumulator, BiConsumer<R,R> combiner) | 스트림의 요소를 수집한다.
주로 요소를 그룹화하거나 분할한 결과를 컬렉션에 담아 반환하는데 사용된다. |

> 💡 reduce와 collect가 최종연산의 핵심이다. 특히나 reduce가 핵심이다.
> 최종연산은 스트림 요소를 소모하면서 스트림이 닫히기 때문에 단 한번만 수행할 수 있다.

### 스트림의 모든 요소에 지정된 작업을 수행 - forEach(), forEachOrdered()

```java
void forEach(Consumer<? super T>action // 병렬스트림인 경우 순서가 보장되지 않음
        void forEachOrdered(Consumer<? super T>action //병렬 스트림인 경우에도 순서가 보장됨
```

- 직렬스트림

```java
IntStream.range(1,10).**sequential**().forEach(System.out::print); // 123456789
        IntStream.range(1,10).**sequential**().forEachOrderded(System.out::print); //123456789
```

- 병렬스트림(데이터가 많은 경우, 여러 스레드가 나눠서 처리하기 위해 사용)

```java
IntStream.range(1,10).**parallel**().forEach(System.out::print); // 291837465(순서보장x)
        IntStream.range(1,10).**parallel**().forEachOrderded(System.out::print); //123456789
```

### 스트림의 모든 요소에 조건 검사를 수행 - allMatch(), anyMatch(), noneMatch()

```java
boolean allMatch(Predicate<? super T>predicate) //모든 요소가 조건식에 만족 : true
        boolean anyMatch(Predicate<? super T>predicate) //요소 중 하나라도 조건식에 만족 : true
        boolean noneMatch(Predicate<? super T>predicate) //모든 요소가 조건식에 만족하지 않으면 : true
```

```java
boolean hasFailedStu=stuStream.anyMatch(s->s.getTotalStore()<=100; //100점 밑 학생이 한명이라도 있어?
```

### 조건에 일치하는 요소 찾기 - findFirst(), findAny()

찾는 요소가 null일 수 있기때문에 Optional로 반환한다.

```java
Optional<T> findFirst() //첫 번째 요소를 반환. 순차 스트림에 사용
        Optional<T> findAny() //아무거나 하나 반환. 병렬 스트림에 사용
```

```java
**Optional<Student>**result=**stuStream**.filter(s->s.getTotalStore<=100).**findFirst**();
        **Optional<Student>**result=**parallelStream**.filter(s->s.getTotalStore()<=100).**findAny**();
```

### 스트림의 요소를 하나씩 줄여가며 누적연산(accumulator) 수행 - reduce()

```java
Optional<T> reduce(BinaryOperator<T> accumulator) //초기값이 없어 누산 결과가 없을 경우 null이 될 수 있어, Optional로 반환함.
        T reduce(T identity,BinaryOperator<ZT>Z accumulator)
        U reduce(U identity,BiFunction<U, T, U> accumulator,BinaryOperator<U> combiner)
```

- identity : 초기값
- accumulator : 이전 연산결과와 스트림의 요소에 수행할 연산
- combiner : 병렬처리된 결과를 합치는데 사용할 연산 (병렬 스트림)

```java
//int reduce(int identity, IntBinaryOperator op)

int count=intStream.reduce(0,(a,b)->a+1); //count()
        int sum=intStream.reduce(0,(a,b)->a+b);   //sum()
        int max=intStream.reduce(Integer.MIN_VALUE,(a,b)->a>b?a:b); // max()
        int min=intStream.redcue(Integer.MAX_VALUE,(a,b)->a<b ?a:b); // min()
```

```java
int a=identity;
        for(int b:stream)
        a=a+b; //sum
```

> 💡 Stream의 최종연산은 reduce()로 만들어진 것이다.

### collect()와 Collectors

- **collect()**는 Collector를 매개변수로 하는 스트림의 최종연산

```java
Object collect(Collector collect) //Collector를 구현한 클래스의 객체를 매개변수로
        Object collect(Supplier supplier,BiConsumer accumulator,BiConsumer combiner)//잘 안쓰임
```

- **Collector** 인터페이스는 수집(collect)에 필요한 메서드를 정의해 놓은 인터페이스

```java
public interface Collector<T, A, R> { //**T(요소)를 A에 누적한 다음, 결과를 R로 변환해서 반환**
    Supplier<A> supplier(); //StringBuilder::new 누적할 곳 **(핵심)**

    BiConsumer<A, T> accumulator(); //(sb, s) -> sb.append(s) 누적 방법 **(핵심)**

    BinaryOperator<A> combiner(); //(sb1, sb2) -> sb1.append(sb2) 결합 방법(병렬 작업을 수행했을 때 각 스레드가 수행한 결과를 합치는 행위)

    Function<A, R> finisher(); //sb -> sb.toString() 최종변환

    Set<Characteristics> characteristics(); //컬렉터의 특성이 담긴 Set을 반환
  ...
}
```

- **Collectors** 클래스는 다양한 기능의 컬렉터(Collector를 구현한 클래스)를 제공

```java
변환-mapping(),toList(),toSet(),toMap(),toCollection(),...
        통계-counting(),summingInt(),averagingInt(),maxBy(),minBy(),summarizingInt(),...
        문자열 결합-joining()
        리듀싱-reducing()
        그룹화와 분할-groupingBy(),partitioningBy(),collectingAndThen()
```

### 스트림을 컬렉션, 배열로 변환

- 스트림을 컬렉션으로 변환 - toList(), toSet(), toMap(), toCollection()

```java
List<String> names=stuStream.map(Student::getName) // Stream<Student> -> Stream<String>
        .collect(Collectors.toList()); // Stream<String> -> List<String>

        ArrayList<String> list=names.stream()
        .collect(Collectors.toCollection(ArrayList::new); //Stream<String> -> ArrayList<String>

        Map<String, Person> map=personStream
        .collect(Collection.toMap(p->p.getRegId(),p->p)); // Stream<Persion> -> Map<String, Persion>
```

- 스트림을 배열로 변환 - toArray()

```java
Student[]stuNames=studentStream.toArray(Student[]::new); // OK
        Student[]stuNames=studentStream.toArray(); //에러
        Object[]stuNames=studentStream.toArray(); //OK
```

### 스트림의 통계 - counting(), summingInt()

- 스트림의 통계정보 제공 - counting(), summingInt(), maxBy(), minBy(), …

```java
long count=stuStream.count(); // 전체 요소를 대상으로 카운팅
        long count=stuStream.collect(Collectors.counting()); //그룹별로 나눠서 카운팅 가능
```

```java
long totalScore=stuStream.mapToInt(Student::getTotalScore).sum();
        long totalScore=stuStream.collect(Collectors.summingInt(Student::getTotalScore));
```

```java
OptionalInt topScore=studentStream.mapToInt(Student::getTotalScore).max();
        Optional<Student> topStudent=stuStream
        .max(Comparator.comparingInt(Student::getTotalScore));
        Optional<Student> topStudent=stuStream
        .collect(maxBy(Comparator.comparingInt(Student::getTotalScore)));
```

### 스트림을 리듀싱 - reducing()

- 스트림을 리듀싱 - reducing()

```java
Collector reducing(BinaryOperator<T> op)
        Collector reducing(T identity,BinaryOperator<T> op)
        Collector reducing(U identity,Function<T, U> mapper,BinaryOperator<U> op) //map + reduce
```

```java
IntStream intStrema=new Random().ints(1,46).distinct().limit(6);

        OptionalInt max=intStream.reduce(Integer::max); // 전체 리듀싱
        OptionalInt max=intStream.**boxed()**.collect(Collectors.reducing(Integer::new));//그룹별 리듀싱 가능
```

```java
long sum=intStream.reduce(0,(a,b)->a+b);
        long sum=intStream.**boxed()**.collect(Collectors.reducing(0,(a,b)->a+b));
```

```java
int grandTotal=stuStream.map(Student::getTotalScore).reduce(0,Integer::sum);
        int grandTotal=stuStream.collect(reducing(0,Student::getTotalScore,Integer::sum));
```

- 문자열 스트림의 요소를 모두 연결 - joining()

```java
String studentNames=stuStream.map(Student::getName).collect(joining());
        String studentNames=stuStream.map(Student::getName).collect(joining(","));
        String studentNames=stuStream.map(Student::getName).collect(joining(",","[","]"));
        String studentInfo=stuStream.collect(Collectors.joining(","));
```

### `reduce()` vs `Collect()`

`reduce()`는 전체 요소를 대상으로 리듀싱을 수행한다.

`collect()`는 전체 요소가 아닌 그룹별로 리듀싱을 수행한다.

### 스트림의 그룹화와 분할

- Collectors.partitioningBy()는 스트림을 **2분할**한다.

```java
Collector partitioningBy(Predicate predicate)
        Collector partitioningBy(Predicate predicate,Collector downstream)
```

- Collectors.groupingBy()는 스트림을 **n분할**한다.

```java
Collector groupingBy(Function classifier)
        Collector groupingBy(Function classifier,Collector downstream)
        Collector groupingBy(Function classifier,Supplier mapFactory,Collector downstream)
```

- 스트림의 분할 - partitioningBy()

스트림의 요소를 2분할

```java
Map<Boolean, List<Student>>stuBySex=stuStream
        .collect(partitioningBy(Student::isMale)); //학생들을 성별로 분할
        List<Student> maleStudent=stuBySex.get(true); //Map에서 남학생 목록을 얻는다.
        List<Student> maleStudent=stuBySex.get(false); //Map에서 여학생 목록을 얻는다.
```

```java
Map<Boolean, Long> stuNumBySex=stuStream
        .collect(partitioningBy(Student::isMale,counting()));//분할 + 통계
        System.out.println("남학생 수 : "+stuNumBySex.get(true));
        System.out.println("여학생 수 : "+stuNumBySex.get(false));
```

```java
Map<Boolean, Optional<Student>>topScoreBySex=stuStream //분할 + 통계
        .collect(partitioningBy(Student::isMale,maxBy(comparingInt(Student::getScore))));
        System.out.println("남학생 1등 :"+topScoreBySex.get(true));
        System.out.println("남학생 1등 :"+topScoreBySex.get(false)); 
```

```java
Map<Boolean, Map<Boolean, List<Student>>>failedStuBySex=stuStream //다중 분한
        .collect(partitioningBy(Student::isMale, //1.성별로 분할(남/녀)
        partitioningBy(s->s.getScore()< 150))); //2.성적으로 분할(불합격/합격)
        List<Student> failedMaleStu=failedStuBySex.get(true).get(true) // 불합격 남자
        List<Student> failedMaleStu=failedStuBySex.get(false).get(true) // 불합격 여자
```

- 스트림 분할 - groupingBy()

스트림의 요소를 n분할하여 그룹화

```java
Collector groupingBy(Function classifier)
        Collector groupingBy(Function classifier,Collector downstream)
        Collector groupingBy(Function classifier,Supplier mapFactory,Collector downstream)
```

```java
Map<Integer, List<Student>>stuByBan=stuStream //학생을 반별로 그룹화
        .collect(**groupingBy**(Student::getBan,toList());//toList()는 생략가능(기본값)
```

```java
Map<Integer, Map<Integer, List<Student>>>stuByHakAndBan=stuStream //다중 그룹화
        .collect(**groupingBy**(Student::getHak,**groupingBy**(Student::getBan)))
```

```java
stuStream
        .collect(
        groupingBy(Student::getHak,groupingBy(Student::getBan,
        **mapping**(s->{
        if(s.getScore>=200)return Student.Level.HIGH;
        else if(s.getSocre>=100)return Student.Level.MID;
        else return Student.Level.LOW;
        },toSet())
        ))
        );
```

![Untitled](Stream/Untitled%203.png)

![Untitled](Stream/Untitled%204.png)