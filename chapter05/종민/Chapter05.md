# 연관관계 매핑 기초

## 단방향 연관관계
- 객체 연관관계
회원 객체는 Member.team 필드로 팀 객체와 연관관계를 맺는다. 회원 객체외 팀 객체는 단방향 관계이다. Member -> team만 가능
- 테이블 연관관계
회원 테이블과 티 ㅁ테이블은 Team_id로 양방향 관계를 맺고 있다. Member join team, team join member 둘 다 가능하다.
- 객체연관관계와 테이블 연관관계의 차이
참조를 통한 연관관계는 항상 단방향이므로, 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해서 참조를 보관해야 한다.
- 결론적으로 양방향 관계가 아니라 서로 다른 다방향 관계 2개

### 순수한 객체 연관관계
- 객체입장에서는 참조를 통해 연관관계를 탐색할 수 잇는데 이를 객체 그래프 탐색이라고 한다.
### 테이블 연관관계
데이터베이스는 왜래 키를 사용해서 연관관계를 탐색할 수 있는데 이를 JOIN이라고 한다.
### 객체 관계 매핑
```
@Entity
@Setter
public class Member{
    @Id
    @Column(name="MEMBER_ID")
    private String id;

    private String username;

    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
}

@Getter
@Setter
@Entity
public class Team{
    @Id
    @Column(name="TEAM_ID")
    private String id;

    private String name;
}
```

- @ManyToOne: 다대일 관계라는 매핑정보
fetch: 글로벌 페치 전략을 설정한다. FetchType.EAGER, FetchType.LAZY


- @JoinColumn: 조인 칼럼은 외래 키를 매핑할 때 사용한다. 회원과 팀 테이블은 TEAM_ID 외래키로 연관관계를 맺는다. """이 어노테이션은 생략할 수 있다."""
- JoinColumn을 생략하면 외래 키를 찾을 때 필드명_칼럼명 으로 찾는다
- 그냥 결국에는 JoinColumn에서 명시를 하는게 낫겠다

## 연관관계 사용

### 저장
저장은 그냥 member.setTeam(team) 해서 저장하면 된다.

### 조회
- 객체 그래프 탐색(객체 연관관계를 사용한 조회)
member.getTeam();

- 객체지향 쿼리 사용 JPQL
"select m from Member m join m.team t where t.name=:teamName"

### 수정
setter로 set해놓으면 영속성 관리가 되고 마지막에 update가 전송됨

### 제거
member.setTeam(null)

### 연관된 엔티티 삭제
연관된 엔티티를 삭제하려면 연관관계를 미리 삭제하고 제거해야 한다.
그렇지 않으면 키 제약조건으로 에러가 난다.

## 양방향 연관관계
```
@Getter
@Setter
@Entity
public class Team{
    @Id
    @Column(name="TEAM_ID")
    private String id;

    private String name;

    @OneToMany(mappedBy="team")
    private List<Member> members = new ArrayList<Member>();
}
```

### 일대다 컬렉션 조회
객체 그래프 탐색으로 team.getMembers()

## 연관관계의 주인
mappedBy
객체에는 양방향 연관과계라는 것이 없다. 
엔티티를 단방향으로 매핑하면 참조를 하나만 사용하므로 참조로 외래키를 관리하면 된다. 
엔티티를 양방향 관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나다.
JPA에서는 두 객체 연관과계 중 하나를 정해서 테이블의 외래키를 관리해야 하는데 이를 연관관계의 주인이라고 한다.

### 양방향 매핑의 규칙: 연관관계의 주인
- 연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래키를 관리(등록, 수정, 삭제)할 수 있다. 주인이 아닌 곳은 읽을 수 만 있따.
연관관계를 어떤걸 주인으로 할 지는 mappedBy로 사용

### 연관관계의 주인은 외래 키가 있는 곳
연관관계의 주인은 외래 키가 있는 곳으로 정해야 한다. 여기서는 회원이 외래 키를 가지고 있으므로 Member.team이 주인이 된다.
그래서 mappedBy는 연관관계 주인의 "필드"를 주면 된다.

## 양방향 연관관계 저장
team.getMembers.add(member)는 연관관계의 주인이 아니므로 데이터베이스에 저장할 때 무시된다
member.setTeam(team)이 되어야 한다.

<이 문제 답 모르면 제발 JPA 쓰지 마세요. 공부를 하거나>

<이 문제 답 모르면 JPA 6장 발표하세요>
https://www.youtube.com/watch?v=brE0tYOV9jQ


## 양방향 연관관계의 주의점
### 순수한 객체까지 고려한 양방향 연관관계
원래는 연관관계의 주인에만 set해줘도 JPA관점에선 DB에 잘들어간다.
그러나, 객체관점에서는 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.

### 연관관계 편의 메소드
member.setTeam(team)과 team.getMembers().add(member)를 각각 호출하다보면 둘중에 하나만 호출하는 실수를 할 수 있으므로
두 코드가 하나인 거처럼 묶어서 사용하는게 낫다.

### 편의 메소드 작성 시 주의 사항
member.setTeam(anotherTeam)을 하게 되면 member와 teamA와의 관계는 삭제되지 않았다. 그래서 기존 팀과의 관계를 제거 하고 넣어주도록 해야 한다.

객체 관점에서 고려하기가 어렵다! 이거 매번 개발하면서 수정함...