# Index(1)-Clustered Index와 Secondary Index

# MySQL의 Index 구조

먼저, MySQL의 Index는 `B-Tree`를 기반으로 Index를 제공한다. 이 Index는 `Clusted Index`와 `Secondary Index`로 구분한다.

Clusterd Index는 모든 테이블마다 각 1개씩 존재할 수 있다. 별도로 Clustered Index를 명시적 생성하는 것이 아니라, `PRIMARY KEY` 가 존재하거나, `Null을 허용하지 않는 Unique Key`가 존재하는 컬럼이 Clustered Index가 된다. 물론 둘이 함께 존재하는 경우 테이블 마다 1개씩만 존재할 수 있기 때문에 PRIMARY KEY를 기준으로 Clustered Index가 된다. 기본적으로 이 Clustered Index는 별도로 지정하지 않는 경우, Index이 Key를 기준으로 `오름차순` 정렬된다. Index 생성에 따라 내림차순으로 정렬할 수도 있다.

# Clustered Index

Clustered Index는 Primary Key가 존재하거나 Null을 허용하지 않는 Unique Key가 존재할 경우 자동으로 인덱스가 만들어진다.

### Primary Key Index 예시

1. **Clustered Index가 있는 경우와 없는 경우의 정렬**

1) 아래의 그림과 같이 **ID 필드를 PRIMARY KEY**로 지정한 테이블을 작성하였다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled.png)

2) 그결과, PRIMARY KEY 라는 이름의 인덱스가 생성되었다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled%201.png)

4) 예시로 총 3개의 데이터를 넣었다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled%202.png)

5) 순서는 1, 3, 2 순서로 INSERT 쿼리를 실행하였지만, SELECT 결과는 id의 오름차순으로 출력되었다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled%203.png)

1. **Clustered Index가 없는 경우의 정렬**

1) 다시 인덱스를 제거하고 동일하게 INSERT 후 SELECT 쿼리를 실행하였다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled%204.png)

2) 그 결과, 위에서 PRIMARY KEY 인덱스가 있었던 경우와 다르게 1, 3, 2 순서로 SELECT 되었다. 즉, 입력된 순서대로 데이터가 출력되는 것을 확인할 수 있었다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled%205.png)

> 💡 Clustered Index는 PRIMARY KEY이거나 not null인 Unique KEY가 존재하는 경우 자동으로 생성되며, Key를 오름차순(Default) 정렬한다.

### Clustered Index의 인덱스 구조

그러면 이 Clustered Index의 인덱스 구조는 어떻게 되어있을까?

Clustered Index의 B트리 구조는 아래의 그림과 같다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled%206.png)

Clustered Index는 루트페이지와 리프페이지의 구성으로 이중 구조로 이루어져있다. 루트페이지에서 검색기준에 따라 리프페이지에 접근하여 실제 데이터를 조회한다.

루트 페이지는 리프페이지는 실제 데이터를 가지고있는 페이지 즉 데이터 페이지 라고 부른다.

예를들어 id 1번을 가진 데이터를 찾으려면, 루트페이지에서 1번의 위치를 확인하고 1000번지에 해당하는 페이지로 이동하여 1번 id에 해당하는 데이터를 조회한다.

또한 값이 추가 삭제 변경되면, 이에 따라 인덱스의 구조가 자동으로 정렬된다. 따라서 추가 삭제 변경 작업이 성능에 영향을 미치기도한다.

이미 만들어져있는 많은 row를 가진 테이블에 clustered index를 적용하게되면, 그 안에서도 정렬작업이 이루어지기 때문에, 많은 데이터가 있는 테이블에서 꼭 clustered index를 적용해야한다면, 트래픽이 적은 시간에 행해야한다.

# Secondary Index

보조인덱스 또는 non-clustered Index라고 불리우는 인덱스이다. 해당 인덱스는 Clustered Index와 다르게 한개 이상 만들 수 있다. 하지만 많은 인덱스를 갖는 경우에 성능에 영향을 미치기 때문에 너무 많은 인덱스를 만들지 않는 것이 좋다.

보조 인덱스는 unique key를 가지고잇는 필드가 존재하면 자동으로 만들어지고, 사용자가 직접 `create index` 명령을 통해 만들 수도 있다.

보조 인덱스는 clustered Index와는 다르게 보조 인덱스를 만든다고해서 데이터가 정렬되거나 하지는 않는다. 그 이유는 보조 인덱스에서는 리프페이지에서 데이터 페이지의 위치를 가르키는 값이 저장되기 때문이다. 단순히 위치만 가르키면 된다.

### Secondary Index 구조

Secondary Index는 루트 페이지, 리프 페이지, 데이터 페이지 구조로 3중 구조를 이루고있다.

그리고 리프 페이지는 항상 실제 데이터 값을 찾아가기 위한 번지값을 가지고 있다. 위의 Clustered Index의 경우에는 리프 페이지가 곧 데이터페이지여서, 루트 페이지부터 찾아나가기 시작하면 결국 리프페이지에서 실제 데이터를 획득할 수있었지만, 보조 인덱스에서의 리프페이지는 실제 데이터페이지의 번지값과 위치를 식별하는 값을 가지고 있다. 그리고 보조 인덱스 내에서는 값 별로 정렬이 되어있지만 데이터페이지는 그렇지 않다.

![Untitled](Index(1)-Clustered%20Index와%20Secondary%20Index/Untitled%207.png)