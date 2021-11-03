# 객체지향 쿼리 언어

나이가 30살 이상인 회원을 모두 검색하고자 하는 등.
ORM을 사용하면 데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 개발하므로 검색도 테이블이 아닌 엔티티 객체를 대상으로 하는 방법이 필요하다.

jpql은 이런 문제를 해결하기 위해 만들어졌는데 다음과 같은 특징이 있다

- 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리다.
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.

SQL이 데이터베이스 테이블을 대상으로 하는 데이터 중심의 쿼리라면 JPQL은 엔티티 객체를 대상으로 하는 객체지향 쿼리다.

- JPQL(java persistence query language)
- Criteria 쿼리: JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
- 네이티브 SQL: JPA에서 JPQL 대신 직접 SQL을 사용할 수 있다.
- QueryDS: Criteria 쿼리처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음, 비표준 오픈소스 프레임워크다.
- JDBC 직접 사용, MyBatis 같은 SQL 매퍼: 필요하면 JDBC를 직접 사용할 수 있음

## JPQL 

JPQL은 엔티티 객체를 조회하는 객체지향 쿼리다.
JPQL은 SQL을 추상화해서 특정 데이터베이스에 의존하지 않는다.
JPQL은 SQL보다 간결하다. 엔티티 직접 조회, 묵시적 조인, 다형성 지원으로 SQL보다 코드가 간결하다.

```
select m from Member as m where m.username = 'kim'
```

## Criteria

Criteria는 JPQL을 생성하는 빌더 클래스이다. 문자가 아닌 query.select(m).where(...)처럼 프로그래밍 코드로 JPQL을 작성할 수 있다는 점이다.
JPQL은 문자열이다보니 런타임에 실행되면서 에러가 발생하지만 criteria는 컴파일 시에 오류를 발견할 수 있다.

```
Root<Member> m = query.from(Member.class);
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username), "kim));
```

이건 큰 장점인거 같다. 지금 mongodb jpa쓰면서 raw 쿼리 고려를 해봤었는데 항상 실행 해보고 쿼리날려보고 에러 나나 안나나 확인해보고

## QueryDSL 소개

QueryDSL의 장점은 코드 기반이면서 단순하고 사용하기 쉽다.

```
List<Member> members = query.from(member).where(member.username.eq("kim)).list(member);
```
QueryDSL도 어노테이션 프로세서를 사용해서 쿼리 전용 클래스를 만들어야 한다. QMember는 Member 엔티티 클래스를 기반으로 생성한 QUeryDSL 쿼리 전용 클래스다.

## 네이티브 SQL 소개

특정 데이터베이스에 의존하는 기능을 사용해야 할 때. Connect by 특징 같은 것. 네이티브 SQL의 단점은 특정 데이터베이스에 의존하는 SQL을 작성해야 한다는 것이다.

## JDBC 직접 사용, 마이바티스 

JDBC나 마이바티스를 JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제로 플러시해야한다. 둘다 같이 사용하면 영속성 컨텍스트와 데이터베이스를 불일치 상태로 만들 수 있다. 이런 이슈를 해결하는 방법은 JDBC로 바로 SQL을 실행하기 전에 영속성 컨텍스트를 수동으로 플러시해서 데이터베이스와 영속성 컨텍스트를 동기화하면 된다.

# JPQL
- JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다.
- JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.

## SELECT 문
```
SELECT m FROM Member AS m where m.username = 'Hello'
```
- 대소문자 구분: 엔티티와 속성은 대소문자를 구분한다.
- 엔티티 이름: JPQL에서 사용한 Member는 클래스명이 아니라 엔티티명이다. 엔티티 명을 지정하지 않으면 클래스 명을 기본값으로 사용
- 별칭은 필수: JPQL은 별칭을 필수로 사용해야 한다.

## 파라미터 바인딩
JPQL은 이름 기준 파라미터 바인딩을 지원한다.

- 이름 기준 파라미터
```
String usernameParam = "User1";
em.createQuery("select m from Member m where m.username = :username", Member.class);
query.setParameter("username", usernameParam);
```
메소드 체인 방식으로도 작성 가능
```
creaetquery.setParameter("username", usernameParam).getResultList();
```

- 위치 기준 파라미터
```
em.createQuery("SELECT m FROM Member m where m.username = 1?", Member.class).setParametmer(1, usernameParam).getResultList();
```

## 프로젝션
1) 엔티티 프로젝션
2) 임베디드 타입 프로젝션
3) 스칼라 타입 프로젝션
4) 여러 값 조회(TypeQuery를 사용할 수 없고 Query를 사용해야 함)

## 집합과 정렬
- COUNT, MAX, MIN, AVG, SUM
- NULL값은 무시하므로 통계에 잡히지 않는다
- 값이 없는데 SUM, AVG, MAX, MIN을 사용하면 NULL이 되고 COUNT는 0이 된다.
- DISTINCT를 집합 함수 안에 사용해서 중복된 값을 제거하고 나서 집합을 구할 수 있다.
' select COUNT(DISTINCT m.age) from Member m '
- DISTINCT를 COUNT에서 사용할 때 임베디드 타입은 지원하지 않는다
- GROUP BY는 통계 데이터를 구할 때 특정 그룹끼리 묶어준다
- HAVING도 사용 가능

## JPQL 조인

### INNER JOIN
```
SELECT m FROM Member m INNER JOIN m.team t where t.name = :teamName
```
JPQL 조인의 가장 큰 특징은 연관 필드를 사용한다는 것이다. m.team이 연관 필드인데 연관 필드는 다른 엔티티와 연관관계를 가지기 위해 사용하는 필드이다.

.
.
.
### 페치 조인
페치 조인은 SQL에서 이야기하는 조인의 종류는 아니고 JPQL에서 성능 최적화를 위해 제공하는 기능이다. 연관된 엔티티나 컬렉션을 한번에 조회하는 기능인데 join fetch 명령어로 사용할 수 있다.

select m from Member m join fetch m.team

회원과 팀을 지연 로딩으로 설정했을 때 회원을 조회할 때 팀도 함께 조회했으므로 연관되니 팀을 사용해도 지연 로딩이 일어나지 않는다

### 페치 조인과 일반 조인의 차이
JPQL은 결과를 반환할 때 연관관계까지 고려하지 않는다. 따라서 팀 엔티티만 조회하고 연관된 회원 컬렉션은 조회하지 않는다.

페치조인을 사용하면 SQL 호출 횟수를 줄여 성능을 최적화 할 수 있다.
최적화를 위해 글로벌 로딩 전략을 즉시 로딩으로 설정하며 애플리케이션 전체에서 항상 즉시 로딩이 일어난다 물론 일부는 빠를 수는 있지만 전체로 보면 사용하지 않는 엔티티를 자주 로딩하므로 오히려 성능에 악영향을 미칠 수 있다. 따라서 글로벌 로딩 전략은 지연로딩을 사용하고 최적화가 필요하면 페치조인을 사용하자 