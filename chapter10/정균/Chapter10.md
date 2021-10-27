# `JPQL 소개`

`JPQL은 엔티티 객체를 조회하는 객체지향 쿼리이고, 문법은 SQL과 비슷합니다. JPQL은 SQL을 추상화해서 특정 데이터베이스에 의존하지 않습니다.` 그리고 데이터베이스 방언만 변경하면 JPQL을 수정하지 않아도 자연스럽게 데이터베이스를 변경할 수 있습니다. 

![스크린샷 2021-09-03 오전 5 46 14](https://user-images.githubusercontent.com/45676906/131913834-86972d00-7c3d-48fb-8a94-843b9b26af71.png)

가령, 위와 같이 JPQL 코드를 보면 Member는 테이블이 아니라 엔티티를 대상으로 쿼리를 작성한 것을 볼 수 있습니다. 즉, 위의 코드를 실행하면 아래와 같은 쿼리가 만들어집니다.

<br>

![스크린샷 2021-09-03 오전 5 47 46](https://user-images.githubusercontent.com/45676906/131913989-be6f9bf2-2856-4b1e-81e1-c438482e0706.png)

<br> <br>

## `Criteria 소개`

위의 JPQL 코드를 보면 단순한 String 이라는 것을 알 수 있습니다. 이렇게 단순한 String 이라면 `동적 쿼리를 만들기가 매우 어렵다`라는 단점을 가지고 있는데요. 동적쿼리를 JPQL로 사용한다면 단순히 문자열로 된 JPQL 쿼리들을 +로 연결해서 해야 하는데, 이러면 버그가 날 확률도 높고 관리하기가 상당히 쉽지 않습니다. 그래서 대안으로 나온 것이 `Criteria` 입니다. 

![스크린샷 2021-09-03 오전 5 54 34](https://user-images.githubusercontent.com/45676906/131914825-5e87f23c-2d73-4823-8392-1f167d14b4a1.png)

사용법은 위와 같이 할 수 있습니다. 이것은 쿼리를 문자열로 작성하는 것이 아니라 메소드로 작성하여 좀 더 편할 순 있지만 여전히 `동적 쿼리를 작성하는 것은 어렵고, 가독성도 좋지 않다고 생각합니다.`

<br> <br>

## `QueryDSL 소개`

```java
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;

// 쿼리, 결과조회
List<Member> members =
    query.from(member)
        .where(member.username.eq("kim"))
        .list(member);
```

위의 코드는 QueryDSL로 작성한 코드입니다. 훨씬 가독성도 좋고 사용성도 편리하고 좋다는 것을 느낄 수 있습니다. 자바 코드로 JPQL을 작성할 수 있고, 동적 쿼리도 편리하게 작성할 수 있다는 장점이 있습니다.  

<br> <br>

## `네이티브 SQL 소개`

JPA는 SQL을 직접 사용할 수 있는 기능을 지원하는데 이것을 `네이티브 SQL`이라 합니다. JPQL을 사용해도 가끔은 특정 데이터베이스에 의존하는 기능을 사용해야 할 때가 있습니다.

![스크린샷 2021-09-03 오전 6 05 10](https://user-images.githubusercontent.com/45676906/131916067-65b1a0c8-83d9-4dc2-b57a-561e89ea5504.png)

<br> <br>

## `JPQL이란?`

- JPQL은 객체지향 쿼리 언어입니다. 따라서 테이블을 대상으로 쿼리 하는 것이 아니라 `엔티티 객체를 대상으로 쿼리`합니다.
- JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않습니다.
- JPQL은 결국 SQL로 변환됩니다.

<br>

<img width="709" alt="스크린샷 2021-09-03 오전 6 08 32" src="https://user-images.githubusercontent.com/45676906/131916450-7d8bc5ed-a842-4d06-92f1-eefb441d7ac2.png">

그리고 위의 모델을 가지고 예제 코드를 보면서 JPQL에 대해서 좀 더 알아보겠습니다. 먼저 JPQL 문법 특징에 대해서 공부하겠습니다. 

<br> <br>

## `JPQL 문법`

- `select m from Member as m where m.age > 18` 엔티티와 속성은 대소문자를 구분합니다.(Member, age)
- JPQL 키워드는 대소문자 구분하지 않습니다.(SELECT, FROM, where)
- `엔티티 이름 사용, 테이블 이름이 아닙니다.`
- `별칭은 필수입니다.`

<br> <br>

## `TypeQuery, Query`

- TypeQuery: `반환 타입이 명확할 때` 사용합니다.
- Query: `반환 타입이 명확하지 않을 때` 사용합니다.

<br>

![스크린샷 2021-09-03 오전 6 22 36](https://user-images.githubusercontent.com/45676906/131918011-fb02df07-843f-4965-a423-85ea0919b629.png)

위의 코드를 보면 type1은 Member 전체를 가져오기 때문에 Member 타입으로 TypeQuery를 사용할 수 있고, type2는 username만 가져오기 때문에 String 타입으로 type2를 가져올 수 있습니다. type3는 username은 String이고 age는 int 이기 때문에 타입을 정할 수 없습니다. 그래서 이럴 때는 `Query`를 사용합니다.

<br> <br>

## `결과 조회`

![스크린샷 2021-09-03 오전 6 25 40](https://user-images.githubusercontent.com/45676906/131918340-a49d54b8-9e02-4b35-91f8-ac7e9e9d5554.png)

위와 같이 `getResultList()`를 사용하면 바로 `List<타입>` 형태로 가져올 수 있습니다.

<br>

![스크린샷 2021-09-03 오전 6 27 24](https://user-images.githubusercontent.com/45676906/131918535-566d32ed-01f0-4b32-b989-6155f14fa137.png)

그리고 `getSingleResult()`라는 메소드도 존재하는데 이것은 정확히 하나만 가져올 때 사용합니다. 이 메소드에는 주의할 점이 있습니다.

- `getSingleResult()`로 조회했을 때 `결과가 정확히 하나`여야 하는데, 결과가 없다면 `javax.persistence.NoResultException` 에러가 발생합니다.
- 둘 이상이면 `javax.persistence.NonUniqueResultException` 에러가 발생합니다.

<br> <br>

## `파라미터 바인딩`

바인딩은 `이름 기준`, `위치 기준`이 존재합니다.

### 이름 기준

![스크린샷 2021-09-03 오전 6 32 49](https://user-images.githubusercontent.com/45676906/131919134-e488fd78-421d-4c7e-a824-d99cbfefbd21.png)

<br>

![스크린샷 2021-09-03 오전 6 33 30](https://user-images.githubusercontent.com/45676906/131919188-fb80de36-f442-4c27-8629-0394b194a2a9.png)

위와 같이 username으로 제대로 바인딩이 잘 된 것을 볼 수 있습니다.

<br>

### 위치 기준

![스크린샷 2021-09-03 오전 6 35 48](https://user-images.githubusercontent.com/45676906/131919447-7a773a93-7102-4bc5-b116-63c5c04054fe.png)

하나 추가되면 다 바꿔야 하기 때문에 위치 기준은 웬만하면 사용하지 않는 것이 좋습니다. 갑자기 새로운 것이 추가되면 순서들이 다 꼬이기 때문에 나중에 문제가 생길 확률이 높습니다. 사용법은 `?1`을 사용하면 됩니다.

<br> <br>

## `프로젝션`

SELECT 절에 조회할 대상을 지정하는 것을 `프로젝션`이라 하고, [SELECT {프로젝션 대상} FROM] 으로 대상을 선택합니다.

- SELECT m FROM Member m -> 엔티티 프로젝션
- SELECT m.team FROM Member m -> 엔티티 프로젝션
- SELECT m.address FROM Member m -> 임베디드 프로젝션 (Address는 Embedded 타입이어서)

<br>

![스크린샷 2021-09-09 오후 2 07 36](https://user-images.githubusercontent.com/45676906/132626183-704d0032-78fd-4055-a106-ac1c37cba579.png)

위와 같이 JPQL을 통해서 Member를 꺼냈을 때 이게 영속성 컨텍스트에서 관리가 되는지 확인해보겠습니다. 위에서 `em.flush()`, `em.clear()`를 통해서 영속성 컨텍스트를 비운 후에 JPQL로 다시 쿼리를 조회해보겠습니다. 
그리고 `setAge(20)`을 통해서 Member의 값이 바뀐다면 영속성 컨텍스트에서 관리되고 있는 것인데요. 실행해서 테스트를 해보겠습니다.

![스크린샷 2021-09-09 오후 2 10 52](https://user-images.githubusercontent.com/45676906/132626463-ca505eb6-f7b9-462d-b786-8687e16aa819.png)

<br>

<img width="228" alt="스크린샷 2021-09-09 오후 2 11 16" src="https://user-images.githubusercontent.com/45676906/132626488-237ac79a-bf46-4a33-b2be-1e7f0cadd365.png">

age 값이 20으로 바뀐 것을 확인할 수 있습니다. `즉, Member는 영속성 컨텍스트에서 관리되고 있다는 것을 확인할 수 있습니다.` 

<br> <br>

## `엔티티 프로젝션`

![스크린샷 2021-10-04 오후 8 38 54](https://user-images.githubusercontent.com/45676906/135845190-6be231fb-4a29-4192-bd72-39efc35d31ad.png)

그리고 이번에는 Member와 연관관계를 맺고 있는 Team을 위의 JPQL 처럼 조회했을 때 어떤 쿼리가 실행되는지 보겠습니다. 

![스크린샷 2021-10-04 오후 8 40 31](https://user-images.githubusercontent.com/45676906/135845345-0882f213-672f-46de-a097-f451274f22ab.png)

실행되는 쿼리를 보면 `INNER JOIN`을 통해서 Team을 가져오는 것을 알 수 있습니다.

<br>

![스크린샷 2021-10-27 오전 10 54 26](https://user-images.githubusercontent.com/45676906/138986692-b0fe057c-e5d4-457d-abd6-513cfcc19657.png)

여기서 저는 `@ManyToOne`이 `default`로 `EAGER`로 되어 있으니, `INNER JOIN`으로 되는구나 라고 생각했는데요. 그래서 위와 같이 `FetchType.LAZY`로 잡고 다시 실행해보았습니다. 그런데 그래도 `INNER JOIN`으로 출력이 됩니다. 

그래서 좀 찾아보니 `JPA 프로젝션에서는 무조건 EAGER 로딩`이라고 합니다. 그래서 무조건 `INNER JOIN`이 되어 쿼리가 실행된 것입니다.(궁금) 

<br> <br>

## `임베디드 프로젝션`

![스크린샷 2021-10-04 오후 8 44 28](https://user-images.githubusercontent.com/45676906/135845812-5fce6601-75b9-4fa8-aece-f45af71fed51.png)

<br>

![스크린샷 2021-10-04 오후 8 44 37](https://user-images.githubusercontent.com/45676906/135845840-9635962d-414a-4ff6-aa16-3c086aa4ab3b.png)

이번에는 Order가 `Embedded` 타입의 Address를 가지고 있습니다. 이 때 JPQL을 사용해서 조회해보겠습니다. 

![스크린샷 2021-10-04 오후 8 46 57](https://user-images.githubusercontent.com/45676906/135846132-fe15cfe4-a0fe-4d70-a303-e9a5b2d5c9be.png)

이번에 Order를 통해서 Address를 조회한 상태인데요. 실행된 쿼리는 아래와 같습니다. 

![스크린샷 2021-10-04 오후 8 48 06](https://user-images.githubusercontent.com/45676906/135846225-6714985e-4c78-46f8-86af-6d57388565a5.png)

<br>

![스크린샷 2021-10-04 오후 8 49 02](https://user-images.githubusercontent.com/45676906/135846364-d6870c44-6442-4fbb-b218-87d4df7b59ef.png)

그리고 위의 보이는 것처럼 `임베디드 타입`은 조회의 시작점이 될 수 없다는 제약을 가지고 있습니다. 

<br> <br>

## `프로젝션 - 여러 값 조회`

엔티티를 대상으로 조회하면 편리하겠지만, 꼭 필요한 데이터들만 선택해서 조회해야 할 때도 있습니다. 방법은 아래와 같이 3가지가 존재합니다.

- Query 타입으로 조회
- Object[] 타입으로 조회
- new 명령어로 조회

<br> 

![스크린샷 2021-09-09 오후 2 21 02](https://user-images.githubusercontent.com/45676906/132627411-812f0deb-3ed0-4110-adf4-8e0b3b770dbd.png)

두 번째 방법부터 보면 Object[]을 사용해서 위와 같이 여러 값을 조회할 수도 있습니다. 

<br> <br>

### `NEW 명령어`

![스크린샷 2021-09-09 오후 2 25 32](https://user-images.githubusercontent.com/45676906/132627790-51157205-dbd3-4380-ad05-9e5b66d60771.png)

그리고 마지막 방법인 NEW 명령어 방식은 위와 같이 먼저 Dto를 만들어 보겠습니다. 

<br>

![스크린샷 2021-09-09 오후 2 26 11](https://user-images.githubusercontent.com/45676906/132627886-2198b250-d625-43d1-8ae5-c708121a5a6c.png)

두 번째 방법에서는 타입을 지정할 수 없으므로 `Object[]`로 반환하고, 귀찮은 형변환 작업들을 해주어야 했는데요. 그래서 세 번째 방법으로 `DTO처럼 의미있는 객체로 결과 값을 반환`해서 받을 수도 있습니다. 즉, TypeQuery를 사용할 수 있어서 지루한 객체 형변환 작업을 줄일 수 있습니다. 하지만 이것 역시 new 와 함께 dto 패키지 명을 다 적어주어야 한다는 단점이 존재합니다. 

<br> <br>

## `페이징 API`

페이징 처리용 SQL을 작성하는 일은 지루하고 반복적입니다. 더 큰 문제는 데이터베이스마다 페이징을 처리하는 SQL 문법이 다르다는 점입니다. 그래서 JPA는 페이징을 다음 두 API를 추상화해놓았습니다.

- `setFirstResult(int startPosition)`: 조회 시작 위치(0 부터 시작)
- `setMaxResults(int maxResult)`: 조회할 데이터 수

<br>

![스크린샷 2021-09-09 오후 2 36 28](https://user-images.githubusercontent.com/45676906/132628982-ee68dee1-8fc3-4fba-92ad-9795a888648e.png)

위와 같이 100명의 Member를 INSERT 한 후에 페이징 쿼리를 작성했을 때 어떻게 결과가 나오는지 실행해보겠습니다. 

<br>

![스크린샷 2021-09-09 오후 2 39 46](https://user-images.githubusercontent.com/45676906/132629222-e8be4a4d-de0b-4efc-821f-bf9e4f50eb9d.png)

그러면 위와 같이 100명 중에서 10개의 페이징을 처리해서 결과로 반환해준 것을 볼 수 있습니다. 

<br>

![스크린샷 2021-09-09 오후 2 42 08](https://user-images.githubusercontent.com/45676906/132629450-bdf9d8ce-8094-4055-948c-620169fd20eb.png)

만약에 방언을 Oracle로 바꾸게 되면 위와 같은 페이징 쿼리를 JPA가 만들어줍니다. 상당히 복잡한 것을 알 수 있는데요. 이러한 복잡한 쿼리를 JPA를 추상화 하여 쉽게 사용할 수 있도록 만들어놓았다는 장점이 있습니다.

<br>

![스크린샷 2021-09-09 오후 2 46 08](https://user-images.githubusercontent.com/45676906/132629847-ba31239b-7de2-4453-b05a-2f21180912d4.png)

그리고 MySQL 쿼리는 위와 같이 생성해줍니다. 이렇게 모든 DB 방언마다 쿼리가 다르다는 문제점도 JPA를 통해서 해결할 수 있습니다. 

<br> <br>

## `JPQL 조인`

### 내부 조인

```java
String query = "select m from Member m inner join m.team t";
List<Member> result = em.createQuery(query, Member.class)
        .getResultList();
```

위와 같이 SQL 문법과 비슷하긴 한데 조금 다른 것을 볼 수 있습니다. Member가 Team을 참조하고 있기에 위와 같이 `inner join m.team`으로 `JOIN` 하는 것을 볼 수 있습니다.

<br>

![스크린샷 2021-10-05 오후 1 31 45](https://user-images.githubusercontent.com/45676906/135960431-5d3a8779-0262-4a3f-a6bf-73fe5596810b.png)

실행되는 쿼리를 보면 위와 같이 `inner join`으로 실행되는 것을 볼 수 있고, `마지막에 SELECT 쿼리가 한번 더 실행되는 것을 볼 수 있습니다.` 왜 한번 더 실행되는 것일까요? 8장에서 배운 `즉시 로딩` 때문인데요. 

- 질문: JOIN 할 때 Team도 가져오면 될 것 같은데.. JPQL SELECT 절에 컬럼 명시해주는 곳에 m 만 했기 때문에 EAGER 여도 `INNER JOIN`으로 Member만 가져오고 그 이후에 Team을 가져온다로 이해하면 되는지??

<br>

![스크린샷 2021-10-05 오후 1 33 29](https://user-images.githubusercontent.com/45676906/135960567-9e14567d-4401-4851-a3f8-eb79a6fc4f62.png)

위와 같이 `LAZY 로딩`으로 설정해주면 마지막 SELECT 쿼리가 실행되지 않는 것을 확인할 수 있습니다. 

<br> <br>
 
### `외부 조인`

```java
String query = "select m from Member m outer left join m.team t";
List<Member> result = em.createQuery(query, Member.class)
        .getResultList();
```

외부 조인도 `outer`를 사용하면 됩니다. SQL과 같음!!

<br> <br>

## `JOIN ON 절`

JPA 2.1 부터 조인할 때 `ON 절`을 지원합니다. ON 절을 사용하면 조인 대상을 필터링 하고 조인할 수 있습니다. 또한 연관관계 없는 엔티티 외부 조인도 가능합니다.

<br>

### `회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인`

```java
// JPQL
SELECT m.t FFROM Member m LEFT JOIN m.team t on t.name = 'A'
```

```sql
--  SQL
SELECT m.*, t.*
FROM Membmer m 
LEFT JOIN Team t
ON m.team_id = t.id AND t.name = 'A'
```

그래서 실제로 JPQL에서 ON을 추가해서 JOIN을 해보겠습니다.

<br>

```java
String query = "select m from Member m left join m.team t on t.name = 'teamA'";
List<Member> result = em.createQuery(query, Member.class)
        .getResultList();
```

위와 같이 임의로 t.name에 `teamA`를 준 후에 실행해보겠습니다.

![스크린샷 2021-09-09 오후 3 43 41](https://user-images.githubusercontent.com/45676906/132636137-9d8fb568-79e6-4d64-9567-57738348f584.png)

그러면 위와 같이 `LEFT OUTER JOIN`과 `ON`이 쿼리로 만들어 졌고, 조건을 준 값을 `AND`로 추가된 것을 볼 수 있습니다. 

<br> <br>

### `연관관계 없는 엔티티 외부 조인`

```java
// JPQL
SELECT m, t FROM Member m LEFT JOIN Team t ON m.username = t.name
```

```sql
--  SQL
SELECT m.*, t.* FROM Member m
LEFT JOIN Team t ON m.username = t.name
```

위의 코드는 회원의 이름과 팀의 이름이 같은 대상 외부 조인입니다. 이렇게 연관관계가 없는 것도 조인이 가능합니다. 

<br> <br>

## `서브 쿼리`

JPQL도 SQL처럼 서브 쿼리를 지원합니다. 하지만 몇 가지 제약이 있는데, 서브 쿼리를 WHERE, HAVING 절에서만 사용할 수 있고 SELECT, FROM 절에서는 사용할 수 없습니다. 

<br>

### `나이가 평균보다 많은 회원`

```sql
SELECT m
FROM Member m
WHERE m.age > (SELECT avg(m2.age) FROM Member m2)
```

서브쿼리의 Member는 m2로 별칭을 주었고 위의 Member는 m으로 별칭을 준 것을 볼 수 있습니다. 이처럼 서로 관련이 없게 서브 쿼리를 작성해야 성능이 더 잘나옵니다. 

<br>

### `한 건이라도 주문한 고객`

```sql
SELECT m
FROM Member m
WHERE (SELECT COUNT(o) FROM Order o WHERE m = o.member) > 0
```

이번 쿼리는 Member의 별칭 m이 서브쿼리에서도 사용되는 것을 볼 수 있는데, 이러면 성능 이슈가 있을 수 있습니다. 

<br> <br>

## `서브 쿼리 함수`

### `EXISTS`

서브쿼리에 결과가 존재하면 참입니다. 

```sql
SELECT m FROM Member m
WHERE exists (SELECT t FROM m.team t WHERE t.name = '팀A')
```

<br>

### `IN`

```sql
SELECT t FROM Team t
WHERE t IN (SELECT t2 FROM Team t2 JOIN t2.members m2 WHERE m2.age >= 20)
```

서브쿼리의 결과 중 하나라도 같은 것이 있으면 참입니다. 

<br> <br>

## `JPA 서브 쿼리 한계`

- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- 하이버네이트를 사용한다면 SELECT 절도 가능
- `FROM 절의 서브 쿼리는 현재 JPQL에서 불가능(대부분 조인으로 풀 수 있긴 한데 조인으로도 안되면 불가능..)`

<br> <br>

## `경로 표현식`

경로 표현식이라는 것은 쉽게 이야기해서 `.`을 찍어 `객체 그래프를 탐색`하는 것입니다. 

```sql
SELECT m.username    -- 상태 필드
FROM Member m 
JOIN m.team t        -- 단일 값 연관 필드
JOIN m.orders o      -- 컬렉션 값 연관 필드
WHERE t.nmae = '팀A'
```

여기서 m.username, m.team, m.orders, t.name 모두 경로 표현식을 사용한 예입니다.

<br> <br>

## `경로 표현식 용어 정리`

### 상태 필드

- 단순히 값을 저장하기 위한 필드 (ex: m.username)

<br>

### 연관 필드

- 연관관계를 위한 필드
- 단일 값 연관 필드(@ManyToOne, @OneToOne): 대상이 엔티티일 때(ex: m.team)
- 컬렉션 값 연관 필드(@OneToMany, @ManyToMany): 대상이 컬렉션일 때(ex. m.orders)

<br> <br>

## `경로 표현식 특징`

- 상태 필드: 경로 탐색의 끝입니다. 더는 탐색할 수 없습니다.
- 단일 값 연관 경로: `묵시적으로 내부 조인`이 일어납니다. 단일 값 연관 경로는 계속 탐색할 수 있습니다. 
- 컬렉션 값 연관 경로: `묵시적으로 내부 조인`이 일어납니다. 더는 탐색할 수 없습니다.

<br>

결론은 `묵시적 조인`을 웬~만하면 사용하지 않는 것이 좋습니다. (쿼리를 예상하기가 매우 어려움)

<br>

![스크린샷 2021-09-20 오후 9 07 26](https://user-images.githubusercontent.com/45676906/133999476-d60456da-6273-49ec-b0fc-e615fbdf5948.png)

Member와 Team의 연관관계는 `N:1`인데요. 여기서 위와 같이 Member에서 Team을 찾아오는 JPQL을 적으면 어떤 쿼리가 나가게 될까요?

<br>

![스크린샷 2021-09-20 오후 9 09 06](https://user-images.githubusercontent.com/45676906/133999755-cbac564f-325a-4134-ab59-cea2903881c0.png)

객체 입장에서는 Member에서 Team을 조회한 것이지만 DB 입장에서는 Member에서 Team을 조인하려면 내부적으로 `inner join`을 해서 가져와야 한다는 것을 알 수 있습니다. 이것을 `묵시적 내부 조인이 일어난다` 라고 합니다. 즉, 개발을 할 때 조심히 써야 한다는 것을 알 수 있습니다.

<br>

![스크린샷 2021-10-05 오후 2 47 38](https://user-images.githubusercontent.com/45676906/135967407-6b0148a0-d189-42c1-ba2f-f90fb0a143d6.png)

반대로 Team은 List<Member>를 가지고 있는데요. 위의 JPQL처럼 Team에서 member를 참조하는 것은 컬렉션이기 때문에 불가능합니다. 즉, 컬렉션을 참조하기 위해서는 아래와 같이 명시적 JOIN을 사용해야 합니다.

<br>

![스크린샷 2021-10-05 오후 2 49 46](https://user-images.githubusercontent.com/45676906/135967616-36ac1b66-f324-48b1-9145-968361d62c18.png)

위처럼 명시적 JOIN을 통해서 Member의 별칭 m을 얻은 후에 m으로 username을 참조할 수 있습니다. 

<br> <br>

## `경로 표현식 - 예제`

- SELECT o.member.team FROM Order o (가능)
- SELECT t.members FROM Team (가능)
- SELECT t.members.username FROM Team t (불가능)
- SELECT m.username FROM Team t JOIN t.members m (가능)

<br> <br>

## `경로 탐색을 사용한 묵시적 조인 시 주의 사항`

- 항상 `내부 조인`을 사용합니다. 
- 컬렉션은 경로 탐색의 끝입니다. 컬렉션에서 경로 탐색을 하려면 명시적으로 조인해서 별칭을 얻어야 합니다. 
- 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM 절에 영향을 줍니다. 

<br> <br>

## `페치 조인`

페치 조인은 SQL에서 이야기하는 조인의 종류는 아니고 JPQL에서 `성능 최적화`를 위해 제공하는 기능입니다. 연관된 엔티티나 `컬렉션을 SQL 한 번에 조회하는 기능`입니다.

<br> <br>

## `엔티티 페치 조인`

페치 조인을 사용해서 회원 엔티티를 조회하면서 연관된 팀 엔티티도 함께 조회하는 SQL과 JPQL을 보겠습니다. 

```sql
SELECT M.*, T.* 
FROM MEMEBER M
INNER JOIN TEAM T ON M.TEAM_ID = T.ID
```

```sql
SELECT m FROM Member m 
JOIN fetch m.team
```

위의 예제를 보면 JOIN 다음에 `fetch`를 적으면 연관된 엔티티나 컬렉션을 함께 조회할 수 있습니다. 즉, fetch를 적으면 위의 보이는 SQL 처럼 실행되게 됩니다.  

<img width="1240" alt="스크린샷 2021-09-20 오후 9 22 58" src="https://user-images.githubusercontent.com/45676906/134001534-992e11ba-8146-4fc0-952c-aa47ca8e605a.png">

위의 그림을 보면 JOIN과 fetch의 특징을 볼 수 있습니다. 

<br>

![스크린샷 2021-10-05 오후 3 04 54](https://user-images.githubusercontent.com/45676906/135969479-99d8a28a-ac0e-4cd6-83b8-62f1bd100b54.png)

위와 같이 Team A, B를 만들고 Member1(TeamA), Member2(TeamA), Member3(TeamB)에 저장한 후에 Member List를 조회하고 각 멤버들의 이름, 팀 이름을 출력하는 코드를 작성했습니다. 이 때 어떤 쿼리들이 실행되는지 알아보겠습니다. 

<br>

![스크린샷 2021-10-05 오후 3 09 38](https://user-images.githubusercontent.com/45676906/135969758-5b1bc0d4-9460-45b9-aa93-5b81848c0c17.png)

실행되는 커리를 보면 3번의 쿼리가 실행된 것을 볼 수 있습니다. 어떤 실행 과정을 거쳐서 이렇게 실행이 되었을까요? (참고로 Member는 `지연 로딩`으로 설정되어 있습니다.)

```java
List<Member> result = em.createQuery("SELECT m FROM Member m", Member.class)
                    .getResultList();
```

처음에 위의 코드를 통해서 Member를 조회하기 때문에 첫 번째 SELECT 쿼리가 실행된 것을 알 수 있습니다. 이 때 Member는 1차 캐시에도 저장이 되고 영속성 컨텍스트 위에 올라가게 되었을텐데요. 

그리고 두 번째 SELECT 쿼리는 for문 첫 루프에 해당할 때 실행이 될 것입니다. 

```java
for (Member member : result) {
    System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());
}
```

가장 첫 for문이 실행될 때 Member가 속한 Team의 이름을 출력하는 코드가 있는데, 위에서 말했듯이 `지연 로딩`을 사용하고 있기 때문에 Team이 사용될 때 Team을 SELECT 해서 가져오는 것을 볼 수 있습니다. 즉, 이 때 TeamA도 1차 캐시에 저장이 될 것입니다. 여기서 1차 캐시에 TeamA가 저장되었기 때문에 두 번째 루프인 Member2는 SELECT 쿼리 없이 1차 캐시에서 조회가 된 것입니다. 

그리고 마지막으로 Member3의 Team은 TeamB인데요. TeamB는 1차 캐시에 없기 때문에 다시 SQL을 통해서 조회가 되어 마지막 SELECT 쿼리가 실행된 것을 볼 수 있습니다. 이렇게 해서 총 3번의 SELECT 쿼리가 실행되었습니다. 현재는 Member 3명에 2명이 같은 팀이기 때문에 3번의 쿼리만 실행되었는데 Member 100명이 모두 다른 팀이라면 101번의 쿼리가 실행될 것인데요. `서비스가 커지면 커질 수록 상당히 문제가 많은 상태가 될 것입니다.`

<br> <br>

## `컬렉션 페치 조인`

이러한 문제를 해결할 때 `fetch join`을 사용하면 되는데요. 어떻게 해결할 수 있는지 알아보겠습니다. 

![스크린샷 2021-10-05 오후 3 19 45](https://user-images.githubusercontent.com/45676906/135970858-990c5987-2d44-4ca2-b581-c9fa7b768a20.png)

위와 같이 `fetch join`을 통해서 Member-Team을 조인하면 아래와 같은 쿼리가 실행됩니다.

<br>

![스크린샷 2021-10-05 오후 3 20 00](https://user-images.githubusercontent.com/45676906/135970952-71d97c78-914c-4509-8a33-4ce2c9ca8ecc.png)

fetch join을 사용하니 Member와 Team을 JOIN 해서 한번에 가져오게 되고 쿼리도 1번만 실행된 것을 볼 수 있습니다. 즉, Team도 처음에 가져올 때부터 1차 캐시에 올라가기 때문에 지연로딩을 하더라도 Team이 Proxy 객체가 아니라 진짜 객체를 사용하게 됩니다. 

<br> <br>

## `컬렉션 페치 조인`

일대다 관계인 컬렉션을 페치조인하는 경우를 말합니다. 

```sql
-- JPQL
SELECT t
FROM Team t JOIN FETCH t.members
WHERE t.name = '팀A'
```

```sql
SELECT T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
WHERE T.NAME = '팀A'
```

위와 같이 Team은 List<Member>를 가지고 있는데 이렇게 Team에서 Member로 가는 참조의 경우를 말합니다.  

<br>

![스크린샷 2021-10-05 오후 3 31 48](https://user-images.githubusercontent.com/45676906/135971974-1fe3b225-a01d-4a7b-8a12-4a6c2799f098.png)

그래서 이번에 Team에서 Member로 fetch join을 한 후에 어떻게 실행되는지 보겠습니다. 

<br>

![스크린샷 2021-10-05 오후 3 33 21](https://user-images.githubusercontent.com/45676906/135972145-f9bda6f0-2044-462a-b5fe-8e77e5ad6a51.png)

마지막 결과를 보면 TeamA에 속한 Member가 2명이다 보니 중복되어서 2번 출력되는 것을 볼 수 있는데요. 이 부분을 컬렉션에서 조심해야 합니다. 

<br>

<img width="563" alt="스크린샷 2021-10-05 오후 3 34 47" src="https://user-images.githubusercontent.com/45676906/135972254-05eff7c2-e075-478e-8c29-11dd1945ae37.png">

일다다 조인이기 때문에 TeamA에 2명의 멤버가 존재하기 때문에 위의 그림에서 볼 수 있듯이 결과가 2개가 나오게 될 수 밖에 없습니다. 

<br> <br>

## `페치 조인과 DISTINCT`

SQL의 DISTINCT는 중복된 결과를 제거하는 명령어입니다. JPQL의 DISTINCT 명령어는 SQL에 DISTINCT를 추가하는 것은 물론이고 애플리케이션에서 한 번 더 중복을 제거합니다. 

![스크린샷 2021-10-05 오후 3 42 04](https://user-images.githubusercontent.com/45676906/135973112-cb2d156d-8d4d-44e7-b610-6cb06927ade5.png)

위와 같이 `distinct`를 사용하면 되는데요. JPQL에서의 DISTINCT는 위에서 말했듯이 애플리케이션에서 한번 더 중복을 제거해줍니다. 

<br>

<img width="554" alt="스크린샷 2021-10-05 오후 3 43 43" src="https://user-images.githubusercontent.com/45676906/135973290-8531d144-5e12-448c-a3fd-6a9e7dc7c2bf.png">

즉, 위와 같이 Team의 중복된 엔티티도 제거해주는 특징이 있습니다. 그래서 DISTINCT를 적용하고 결과를 보면 아래와 같습니다. 

<br>

![스크린샷 2021-10-05 오후 3 44 32](https://user-images.githubusercontent.com/45676906/135973416-417f3587-4c24-4d11-8947-02e52266d078.png)

이번에는 중복이 제거되어서 결과가 2개만 나온 것을 볼 수 있습니다. 

<br> <br>

## `페치 조인과 일반 조인의 차이`

- JPQL은 결과를 반환할 때 연관관계를 고려하지 않습니다.
- 단지 SELECT 절에 지정한 엔티티만 조회할 뿐입니다.
- 여기서는 팀 엔티티만 조회하고, 회원 엔티티는 조회하지 않습니다.
- `페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)`
- `페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념`

<br>

일반 조인 실행시 연관된 엔티티를 함께 조회하지 않습니다. 한번 일반 JOIN으로 실행된 쿼리는 어떻게 출력되는지 알아보겠습니다. 

![스크린샷 2021-10-05 오후 3 51 30](https://user-images.githubusercontent.com/45676906/135974198-d714f730-41cb-47a2-83bf-5541704d1deb.png)

<br>

![스크린샷 2021-10-05 오후 3 51 00](https://user-images.githubusercontent.com/45676906/135974149-b6bac8fb-b68f-4c17-a9ae-73c04748bf53.png)

그러면 INNER JOIN으로 가져오는 것은 똑같지만 가져오는 컬럼을 보면 t만 적었기 때문에 Team에 관련된 것들만 가져오는 것을 볼 수 있습니다. 그리고 Member에 대한 데이터도 로딩이 되지 않았기 때문에 Member를 조회할 때 여러 번의 쿼리가 실행되는 것도 볼 수 있습니다.

<br>

![스크린샷 2021-10-05 오후 3 53 39](https://user-images.githubusercontent.com/45676906/135974454-f92379cf-00a2-41b1-b2b1-25d419427473.png)

<br> <br>

## `페치 조인의 특징과 한계`

- 페치 조인 대상에는 별칭을 줄 수 없습니다.

fetch join 자체가 연관된 데이터를 다 가져오는 것이기 때문에 별칭을 잘못 사용했을 때 연관된 데이터 수가 달라져서 데이터 무결성이 깨질 수 있으므로 조심해서 사용해야 합니다. 

<br> 

- 둘 이상의 컬렉션을 페치할 수 없습니다. 

만약에 Team이 List<Member>도 가질 수 있고, List<Order>도 가질 수 있는 경우를 말합니다. 이럴 때 fetch join을 사용하면 1 x N x M 이 되므로 사용하는 것을 주의해야 합니다. 

<br>

- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없습니다.

일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징이 가능합니다.(데이터가 늘어나지 않기 때문에) 반면에 일대다 같은 경우는 조인했을 때 데이터가 늘어나기 때문에 페이징을 할 수 없습니다. 

![스크린샷 2021-10-06 오후 12 37 23](https://user-images.githubusercontent.com/45676906/136136867-56aa1247-b50b-4f79-a824-159405ff91fa.png)

<br>

![스크린샷 2021-10-06 오후 12 37 50](https://user-images.githubusercontent.com/45676906/136136917-7bb38d73-7138-440f-a76b-cdb5dc9c0f0a.png)

일대다 관계에서 페이징을 사용한 후에 실행하면 위와 같이 `applying in memory`라고 경고가 뜨는 것을 볼 수 있습니다. 즉, 모든 연관된 데이터들을 메모리에 올리고 페이징 하는 것이기 때문에 데이터가 엄청나게 많다면 위험한 상황이 될 수 있습니다.

<br>

![스크린샷 2021-10-06 오후 12 40 29](https://user-images.githubusercontent.com/45676906/136137131-ac62c375-dac1-41d0-a554-54fb6386d13e.png)

이러한 상황에서 페이징을 하기 위해서는 반대로 `Member -> Team`으로 `다대일` 관계로 `fetch join`을 하는 방법이 있습니다. 

<br> 

페치조인에서 페이징을 하는 또 다른 방법에 대해서도 정리해보겠습니다. 

![스크린샷 2021-10-06 오후 1 00 30](https://user-images.githubusercontent.com/45676906/136138759-7aceec25-9ecf-4f33-94e5-2a97ef8cba4f.png)

위와 같이 현재 `지연 로딩`으로 설정 되어 있고, Team을 페이징 쿼리를 통해서 가져오고 있습니다. 

<br>

![스크린샷 2021-10-06 오후 1 04 56](https://user-images.githubusercontent.com/45676906/136139084-18076fc0-72fc-4096-ab39-acacb7495db9.png)

실행되는 쿼리를 보면 위와 같은데요. 첫 번째 쿼리는 Team을 가져올 때 실행되는 쿼리이고, 두, 세 번째 쿼리는 지연로딩이기 때문에 for문을 돌면서 Member에 접근할 때 실행되는 쿼리입니다. 즉, N + 1 쿼리가 실행되고 있는 것을 볼 수 있습니다. 여기까지는 계속 살펴보았던 내용인데요. 여기서 `문제가 컬렉션의 페치조인에서는 페이징 쿼리를 사용할 수 없다.` 였는데요.  이러한 문제를 아래와 같이 해결할 수도 있습니다. 

<br>

![스크린샷 2021-10-06 오후 1 09 32](https://user-images.githubusercontent.com/45676906/136139387-18164ac5-d938-4b06-a751-a1c5151ad902.png)

Team에서 `@BatchSize(size = 100)`으로 설정하는 방법인데요.(사이즈는 1000 이하의 적당한 값을 주기) 설정한 후에 실행하면 어떤 쿼리들이 실행되는지 알아보겠습니다. 

![스크린샷 2021-10-06 오후 1 11 32](https://user-images.githubusercontent.com/45676906/136139648-03394ebb-935a-4c48-95da-a41dcbd283af.png)

이번에는 Batch size로 준 값만큼 IN 쿼리를 사용해서 한번에 같이 가져오는 것을 볼 수 있습니다. 이렇게 페치조인 컬렉션 관계에서 페이징 쿼리의 N + 1 문제를 해결할 수 있습니다. 

<br>

![스크린샷 2021-10-06 오후 1 15 45](https://user-images.githubusercontent.com/45676906/136139828-8e9a4313-1115-4c95-ab77-1b509a4c2bf8.png)

글로벌적으로 설정하는 방법은 `persistence.xml`에서 위와 같이 `default-batch-size` 값을 지정하면 똑같은 결과를 얻을 수 있습니다. 


<br> <br> 

## `페치 조인 정리`

- 연관된 엔티티들을 SQL 한번으로 조회합니다. (성능 최적화)
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선합니다.(@OneToMany(fetch=FetchType.LAZY))
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩을 지향합니다. 
- 최적화가 필요한 곳은 페치조인을 적용하면서 해결해나갑니다.
- 모든 것을 페치 조인으로 해결할 수는 없습니다. 
- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적입니다.

<br> <br>

## `엔티티 직접 사용 - 기본 키 값`

```jpaql
SELECT COUNT(m.id) FROM Member m // 엔티티의 아이디를 사용
SELECT COUNT(m) FROM Member m    // 엔티티를 직접 사용
```

JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용합니다.

<br>

### `엔티티를 파라미터로 전달`

```java
String jpql = "SELECT m FROM Member m WHERE m = :member";
List resultList = em.createQuery(jpql)
                    .setParameter("member", member)
                    .getResultList();
```

<br>

### `식별자를 직접 전달`

```java
String jpql = "SELECT m FROM Meber m WHERE m.id = :memberId";
List resultList = em.createQuery(jpql)
                    .setParameter("memberId", memberId)
                    .getResultList()
```

<br>

### `실행된 SQL`

```sql
SELECT m.* FROM Member m WHERE m.id = ?
```

엔티티 자체로 조회하더라도 식별자 값으로 조회하는 것처럼 동작하게 됩니다.

<br> <br>

## `Named 쿼리`

JPQL 쿼리는 크게 `동적 쿼리`와 `정적 쿼리`로 나눌 수 있습니다. 

- 동적 쿼리 : em.createQuery("SELECT ...) 처럼 JPQL을 문자로 완성해서 직접 넘기는 것을 동적 쿼리라고 합니다. 
- 정적 쿼리 : 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용할 수 있는데 이것을 `Named 쿼리` 라고 합니다. 

<br>

`Named 쿼리는 애플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱해둡니다.` 따라서 오류를 빨리 확인할 수 있고, 사용하는 시점에는 파싱된 결과를 재사용하므로 성능상 이점도 있습니다. 

![스크린샷 2021-10-06 오후 2 20 55](https://user-images.githubusercontent.com/45676906/136144919-29deedc3-d4c7-4d20-9380-762b6011e48d.png)

<br>

![스크린샷 2021-10-06 오후 2 38 37](https://user-images.githubusercontent.com/45676906/136146356-28d2f295-988f-489a-bdab-7fc5ca53f24d.png)

<br> <br>

## `JPQL 벌크 연산`

### `벌크 연산`

- 재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면?

<br>

JPA 변경 감지 기능으로 실행하려면 너무 많은 SQL을 실행해야 합니다. 즉, 변경된 데이터가 100건이라면 100번의 UPDATE SQL을 실행해야 합니다. 

<br> <br>

## `벌크 연산 예제`

쿼리 한 번으로 여러 테이블 로구 변경하는 것을 말합니다. 

![스크린샷 2021-10-06 오후 2 45 56](https://user-images.githubusercontent.com/45676906/136147147-a4ed98fd-d895-4b85-9928-e6a111acc7d0.png)