# Sargable (Search ARgument ABLE)

Index는 많은 양의 데이터에서 자주 조회하는 컬럼에 Index를 적용한다.

DBMS 엔진이 만들어 둔 Index를 사용하여 더 쾌적한 쿼리를 실행하려면, 쿼리의 서술논리절이 인덱스를 사용해야하는데, 이를 사거블하다고 표현한다.

사거블 되지 않는 경우에 초점을 맞춰 어떤 경우가 사거블이 되는지를 이해할 필요가 있다. 그래야 보다 Index를 사용하여 성능 향상된 쿼리를 수행할 수 있다.

## where, order by, group by 등에는 index가 걸린 컬럼을 대상으로 수행하라.

너무나 당연한 이야기다. where나 order by, group by 등의 키워드를 index가 걸려있지 않는 대상에 지정하고 쿼리를 날리면 Full Table Scan이 발생하게 된다.

## where 절에 함수, 연산, Like(시작 부분에 %)문은 사거블 하지 않다.

예를들어,

### 연산

```sql
select * from Emp where dept * 10 = 50 order by name;
```

과 같은 쿼리가 있을 때, dept 컬럼에 10을 곱하여 연산을 수행하였기 때문에, 이런 경우에는 Index를 타지 않는다.

```sql
select * from Emp where dept = 50/10 order by name;
```

과 같은 형태로 바꿔줘야한다.

### 함수

함수를 사용하는 경우도 마찬가지이다. 아래와 같이 dept 컬럼에 함수가 적용되어있기 때문에 index를 타지 않는다.

```sql
-- mod(a,b) = a를 b로 나눈 나머지 값
select * from Emp where mod(dept, 10) = 5 order by name;
```

### like 연산

like 연산의 경우 비교 앞글자를 `%` 로 작성하는 경우에도 index를 타지 않는다.

```sql
select * from Emp where dept = 5 and name like '%천%';
```

## between, like, 대소비교(>,< 등) 범위가 큰 경우에는 사거블하지 않다.

지정된 범위가 큰 경우에는 Index를 사용하는 것이 좋지 않다. 오히려 Full Table Scan을 사용하는 편이 더 낫다. 만약 범위를 찾아야하는 경우에는 범위가 작은 경우에 쓰는 것이 좋다.

## or 연산자는 필터링의 반대 개념(로우수를 늘려가는)이므로 사거블이 아니다.

or 연산은 필터링되지 않고, 추가 연산될 대상들이 늘어나기 떄문에 사거블하지 않다.

```sql
select * from Emp where id > 10 or id < 5;
```

## limit을 걸 때, offset이 길어지면 사거블하지 않다.

아래와 같이 오프셋이 없으면 10개만 가져오면 되니깐 문제가 되지는 않는다. 그리고 offset이 작은경우에도 크게 문제가되지 않는다.

```sql
select * from Emp order by name limit 10;
select * from Emp order by name limit 10 offset 5;
```

하지만 offset의 크기가 크면, 건너 뛴 대상을 모두 읽어야 한다. offset이 500이면 500개를 건너뛰는 과정에서 500개를모두 읽어서 offset에 해당하는 값을 읽어내리기 시작해야하는데, 그때 Index는 비효율적이다. 때문에 offset이 크면 사거블하지 않다.

```sql
select * from Emp order by name limit 10 offset 500; 
```

## 범위 보다는 in 절을 사용하는게 좋고, in 보다는 exists가 더 좋다.

위에서 언급한바와 같이 범위는 최대한 작아야하고, 작은 범위를 검색할 때는 in절을 사용하는 편이 좋다. in 절을 사용하는 것은 eq를 여러번 수행하는 것과 같기 때문에, 인덱스 첫번째 컬럼을 in절로 수행하는 것이 범위로 검색하는 것보다 효과적이라는 뜻이다. 만약 서브쿼리를 사용해야하는 경우에는 in절보다는 exists를 사용하는 것이 더 좋다.

<aside>
💡 in 연산자
비교할 컬럼 값에 비교 대상 값 중 하나라도 포함되어 있다면 모두 조회하는 연산자.

</aside>

## 꼭 필요한 경우가 아니라면 서브쿼리보다는 조인(Join)을 사용하자.