# 다대일
- 일대 다 관계에서 외래 키는 항상 다쪽에 있음
- 연관관계 주인은 항상 다쪽이다.

## 다대일 단방향
```
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
}

@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;

    private String name;

}
```
- 회원은 Member.team으로 팀 엔티티를 참조할 수 있지만 반대로 Team에서 Member를 참조하는 필드는 없다.

## 다대일 양방향
```
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
}

@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy="team")
    private List<Member> members = new ArrayList<Member>();

}
```

- 양방향은 외래 키가 있는 쪽이 연관관계의 주인: Member.team 이 연관관계의 주인
- 양방향 연관관계는 항상 서로를 참조해야 한다: 편의 메소드를 작성하는 것이 좋은데 양쪽 다 작성하면 무한루프에 빠지므로 조심

# 일대다
## 일대다 단방향
- 팀 엔티티의 Team.members로 회원 테이블의 TEAM_ID 외래키를 관리한다.
- 일대다 관계에서 외래 키는 항상 다쪽 테이블에 있는데, Member에는 외래 키를 매핑할 수 있는 참조 필드가 없다.
- 그래서 Team.members에서 반대편 테이블의 외래 키를 관리

```
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name="TEAM_ID")
    private List<Member> members = new ArrayList<Member>();
}
```
- 일대다 관계를 매핑할 때는 @JoinColumn을 명시해야 한다. 그렇지 않으면 기본적으로 조인 테이블 전략을 사용
![image](https://user-images.githubusercontent.com/46064193/133374994-6938e827-9151-45dc-be31-ca6280bdea41.png)
- 단점은 객체가 관리하는 외래 키가 다른 테이블에 있다는 것. 추가적인 UPDATE SQL이 필요

## 일대다 양방향
- 양방향 매핑에서 @OneToMany는 연관관계의 주인이 될 수 없다.
- @OneToMany, @ManyToOne 둘 중에 연관관계의 주인은 항상 다쪽인 @ManyToOne을 사용한 곳이므로 mappedBy 속성이 없다
mappedBy 키워드로 연관관계 매핑이 되었다. 라는 것을 의미한다. 그래서 JoinColumn도 없다.

주인은 mappedBy 속성 사용 하지 않고, @ JoinColumn
주인이 아니면 mappedBy 속성으로 주인을 지정한다.
출처: https://ict-nroo.tistory.com/122 [개발자의 기록습관]

# 일대일
- 일대일 관계는 그 반대도 일대일이고 주테이블이나 대상 테이블 중 어느 곳이나 외래키를 가질 수 있다.
- 주테이블에 외래키: 외래 키를 객체 참조와 비슷하게 사용할 수 있어서 객체지향 개발자들이 선호
- 대상 테이블에 외래 키: 일대일에서 일대다로 변경할때 테이블 구조를 그대로 유지할 수 있다.

주테이블의 외래키 장점....? 이해가안간다.

## 다대다
- 관계형 데이터베이스는 다대다를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다.

### 다대다 단방향
```
@Entity
public class Member{
    @Id @Column(name="MEMBER_ID")
    private String id;

    private String username;

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT", joinColums = @JoinColumn(name="MEMBER_ID"),
    inversedJoinColumns = @JoinColumn(name = "PRODUCT_ID")))
    private List<Product> products = new ArrayList<Product>();
}
```

### 다대다 양방향
```
@Entity
public class Product{
    @Id
    private String id;

    @ManyToMany(mappedBy = "products") //역방향 추가
    private List<Member> members;
}

Member.class
public void addProduct(Product product){
    products.add(product);
    product.getMembers().add(this);
}
```

### 다대다: 매핑의 한계와 극복, 연결엔티티 사용
- 회원이 상품을 주문하는데 연결 테이블에 단순히 주문한 회원 아이디와 상품 아이디만 담는 것이 아니라 주문 수량 컬럼이나 주문한 날짜가 필요하면?
- 연결 테이블을 매핑하는 연결 엔티티를 만들고 추가한 컬럼들을 매핑해야 함
- MemberProduct가 외래 키를 다 가지고 있으므로 연관관계의 주인이고, 회원 / 상품에서는 mappedBy를 사용한다.

- 복합 기본키를 사용
```
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct{
    @Id
    @ManyToOne
    @JoinColumn(...)
    private Member member;

    @Id
    @ManyToOne
    @JoinColumn(...)
    private Product product;

    private int orderAmount;
}

public class MemberProductId implements Serializable{
    private String member;
    private String product;
}
```
- 복합키는 별도의 식별자 클래스로 만들어야 한다.
- Serializable을 구현해야 한다.
- equals, hashcode를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클래스는 public이어야 한다.
- @IdClass를 사용하는 방법 외에 @EmbeddedId를 사용하는 방법도 있다.

Q) 음 이 방법 별로인거같은데 어떻게 생각하는지

### 다대다: 새로운 기본키 사용
- 데이터베이스에서 자동 생성해주는 대리키 Long값으로 사용

### 다대다 연관관계 정리
- 식별 관계: 받아온 식별자를 기본키 + 외래키로 사용
- 비식별관계: 받아온 식별자는 외래키로만 사용하고 새로운 식별자를 추가한다.
- 객체 입장에서보면 2번처럼 비식별 관계를 사용하는 것이 낫다