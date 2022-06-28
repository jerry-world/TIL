# Fulltext Index

Fulltext Searh(전문 검색)은 게시물의 내용 제목 등과 같이 문장에서 특정 키워드를 검색하는 기능을 의미한다. 가령 “안녕하세요 박선용입니다” 라는 내용이 DB에 저장되어 있을 때, 박선용 이라는 키워드만 검색하려면 이 전문 검색을 수행해야한다.

이는 [Sargable (Search ARgument ABLE)](./Sargable%20(Search%20ARgument%20ABLE).md) 에서 Sargable하지 않은 like 검색(값 맨 앞에 %가 붙는 경우)을 수행할 때 Index를 사용하지 못할 수도 있지만, 전문 검색은 일부만 검색해도 인덱스를 사용할 수 있기 때문에 `Like %내용%` 와 같은 조회를 처리할때 보다 효과적으로 처리할 수 있다. 예를 들어 돈까스의 까스를 검색하고 싶었는데 문장 내용중에 가까스로 의 까스가 함께 검색되는 부분들도 이 Fulltext Index를 사용하면 보다 효율적으로 처리할 수 있다.

Fulltext Index는 불용어(stop word)나 띄워쓰기 등을 제외하고 난 나머지의 내용을 기준으로 인덱스를 구축하는 방식이다. 

## 불용어

여기서 불용어라 함은 은, 는, 이, 가, 을, 를 뿐만 아니라 또는, 그리고, 이를테면 등 실제 데이터를 찾기에는 필요가 없는 대상들을 불용어로 분류할 수 있다.

MariaDB에서는 이 불용어를 영어로만 제공하고있다.

```sql
select * from INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD;
```

때문에, 한국어의 불용어를 정의하기 위해서는 사용자 정의 stopword를 MySQL에 주입해주어야 한다.

사용자 정의 불용어 테이블을 만들때는 varchar를 갖는 value 컬럼을 추가하고 테이블을 정의해주면된다. 그리고 set global innodb_ft_sever_stopword_table = “db/table”; 로 불용어 테이블을 지정해주면된다.

```sql
---test DB의 불용어 테이블을 korean_stopword 라고 가정
set global innodb_ft_server_stopword_table='test/korean_stopword';
```

외부 파일을 지정할 수 있는 방법도 있으나 그부분은 내용의 주된 부분이 아니기 때문에 생략.

# Fulltext Index 생성 방법

다음은 전문 검색을 위한 인덱스 생성 방법이다.

테이블을 생성할 때 아래와 같이 직접 넣어줄 수 있다.

```sql
create table board(
	b_id int not null,
	b_title varchar(1000) not null,
	b_contents text,
	primary key(b_id),
	fulltext key ft_board(b_title, b_contents)
) ENGINE=InnoDB;
```

또는 create index 명령을 통해서도 만들 수 있다.

```sql
CREATE FULLTEXT INDEX ft_board_idx ON board(b_title, b_contents);
```

Alter문을 사용할 수도 있다.

```sql
ALTER TABLE board ADD FULLTEXT(b_title, b_contents);
```

가급적이면 테이블을 생성하는 과정에서 전문 검색을 필요로 할 것을 미리 예측하는 편이 좋지만 그렇지 못할 경우에는 사용자의 트래픽이 많이 몰리지 않는 시기에 index를 작성해주는 것이 좋다. 데이터가 많으면 index 생성 시간이 오래걸릴 수 있기때문에…

## 전문 검색 방법

fulltext index를 사용하기 위해서는  동등 비교나 크기 비교등의 연산을 수행하는 것이 아니라 `MATCH(..) AGAINST(..)`를 일반적으로 사용한다. 이때, AGINST(..) 뒤에 `IN BOOLEAN MODE` 나 `IN NATURAL LANGUAGE MODE` 등을 명시하여 검색 모드를 지정할 수도 있다.

전문검색의 예시를 한번 살펴보자.

board

| b_title | b_contents |
| --- | --- |
| 선용 방법 | 그는 여가를 선용하여 텃밭을 일궜다. |
| 박선용 입니다. | 안녕하세요 백엔드 지원자 박선용입니다. |
| 권력과 기득권 | 권력 및 기득권의 선용은 사회 정의 구현과 민족 화합을 이루기 위한 선결 조건이다. |
| 안녕하세요 박선용 입니다. | 안녕하세요 프론트엔드개발자 박선용 입니다. |
1. 위의 예시에서 ‘선용’ 을 검색했을 때

```sql
select * from board where match(b_title, b_contents) against('선용') IN BOOLEAN MODE);
```

![Untitled](Fulltext%20Index/Untitled.png)

개별적으로 불용어를 지정해주지 않았기 때문에, title 또는 contents에서 선용 이라는 단어가 혼자 독립적으로 분리되어있는 경우에만 결과가 검색되었다.

1. 위의 예시에서 ‘박선용’을 검색했을 때

```sql
select * from board where match(b_title, b_contents) against('박선용' IN BOOLEAN MODE);
```

![Untitled](Fulltext%20Index/Untitled%201.png)

b_title에 ‘박선용 입니다’와 b_contents ‘개발자 박선용 입니다’ 의 내용에서 박선용이 조회되어 검색결과로 위 두가지의 데이터가 나왔다.

1. 백엔드 지원자인 박선용만 검색하고 싶을 때

```sql
select * from board where match(b_title, b_contents) against('박선용 +백엔드' IN BOOLEAN MODE);
```

![Untitled](Fulltext%20Index/Untitled%202.png)

박선용이라는 텍스트와 +백엔드 를 입력하여 두 글자 모두 포함된 것이 결과로 나왔다.

1. 백엔드 지원자가 아닌 박선용만 검색하고 싶을 때

```sql
select * from board where match(b_title, b_contents) against('박선용 -백엔드' IN BOOLEAN MODE);
```

![Untitled](Fulltext%20Index/Untitled%203.png)

박선용이라는 내용의 결과에서 백엔드가 들어간 대상을 제외한 것이 결과로 나왔다.

위예시에서 `프론트엔드 개발자 박선용` 입니다를 `프론트엔드개발자 박선용` 입니다로 바꿔보자. 그리고 이 아래의 케이스를 한번 확인해보자.

1. 프론트엔드 지원자인 박선용만 검색하고 싶을때

```sql
select * from board where match(b_title, b_contents) against('박선용 +프론트엔드' IN BOOLEAN MODE);
```

![Untitled](Fulltext%20Index/Untitled%204.png)

결과는 아무것도 나오지 않는다. 왜냐면 프론트엔드개발자 가 하나의 단어로 인식되었기 때문이다. 이럴땐 `*` 을 사용하는 것이다. 뒤에 내용이 붙어도 인지할 수 있다.

```sql
select * from board where match(b_title, b_contents) against('박선용 +프론트엔드*' IN BOOLEAN MODE);
```

![Untitled](Fulltext%20Index/Untitled%205.png)

테스트하면서 느끼는건데 전문검색은 게시판 형태나 댓글과 같은 테이블에서 굉장히 유용하게 쓰일 것 같다. 이 내용에 더하여 불용어 처리까지 함께가져가면 굉장히 큰 효율을 갖는 검색을 할 수 있을 것 같다는 생각이 들었다.