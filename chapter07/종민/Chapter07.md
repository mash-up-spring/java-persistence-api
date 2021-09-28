# Chapter 07 고급 매핑

## 7.1 상속 관계 매핑
슈퍼타입 서브타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때
- 각각의 테이블로 변환: 각각을 테이블로 만들고 조회할 때 조인 사용
- 통합 테이블로 변환: 테이블을 하나만 사용해서 통합. '단일 테이블 전략'
- 서브타입 테이블로 변환: 서브타입마다 하나의 테이블을 만든다. 

### 7.1.1 조인 전략
- 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략. 조인을 많이 이용함
- 타입을 구분하기 위해 DTYPE 필드를 임의로 해주어야 함
1) @Inheritance(strategy= InheritanceType.JOINED: 상속 매핑은 '부모 클래스'에 @Inheritance를 사용해야 함. 
2) @DiscriminatorColumn(name = "DTYPE"): 부모 클래스에 구분 칼럼을 지정한다. 기본 값은 DTYPE
3) @DiscriminatorValue("M"): 엔티티를 저장할 때 구분 칼럼에 입력할 값을 지정한다.
4) 기본적으로 자식 테이블은 부모 테이블의 ID 컬럼명을 그대로 사용하는데 바꾸고 싶으면 @PrimaryKeyJoinColumn을 사용

장점
- 테이블이 정규화된다
- 외래키 참조 무결성 제약 조건을 활용할 수 있따.
- 저장공간을 효율적으로 사용한다.

단점
- 조회할때 조인이 많이 사용되므로 성능이 저하될 수 있음
- 조회 쿼리가 복잡함
- 데이터를 등록할떄 부모 테이블, 자식테이블로 INSERT SQL을 두번 실행

- Hibernate 구현체에서는 @DiscrininatorColumn 없이도 동작함

### 7.1.2 단일 테이블 전략
단일 테이블 전략에서는 테이블을 하나만 사용하고 DTYPE으로 어떤 자식 데이터가 저장되었는 지 구분한다. join을 사용하지 않으므로 가장 빠르다

장점
- 조인이 필요 없으므로 조회 성능이 빠르다
- 조회 쿼리가 단순하다

단점
- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다.
- 단일 테이블에 모든 것을 저장 하므로 테이블이 커질 수 있다. 조회 성능이 오히려 느려질 수 있다.

종민's 궁금증
: 이 전략 쓰다가 갑자기 새 엔티티가 생기면? 모든 row 구조가 다 바뀌어야?

### 7.1.3 구현 클래스마다 테이블 전략
장점
- 서브 타입을 구분해서 처리할 때 효과적
- not null 사용 가능

단점
- 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION 사용)
종민's 궁금증: UNION을 쓰면 자유로운 형변환이 어렵지 않나?
ex) Items로 가져와서 isinstanceof(Movie.class) -> (Movie)item 과 같이

- 자식 테이블을 통합해서 쿼리하기 어렵다.
ex) ITEM을 모두 쿼리해와서 사용해야 할때

## 7.2 @MappedSuperclass
단순히 매핑 정보를 상속할 목적
부모로부터 물려받은 매핑 정보를 재정의하려면 @AttributeOverrides나 @AttributeOverride를 사용하고 연관관계를 재정의하려면 @AssociationOverrides나 @AssociationOverride를 사용

- MappedSuperclass를 사용하면 등록일자, 수정일자, 등록자, 수정자 같은 여러 엔티티에서 공통으로 사용하는 속성을 효과적으로 관리

## 7.3 복합 키와 식별 관계 매핑
### 7.3.1 식별관계 vs 비식별 관계
데이터베이스 테이블 사이에 관계는 왼래 키가 기본 키에 포함되는 지 여부에 따라 식별 관계와 비식별 관계로 관계로 구분한다.

## 식별관계
부모 테이블의 기본 키를 내려 받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계
근데 왜 씀?

## 비식별관계
부모 테이블의 기본 키를 받아서 자식 테이블의 외래 키로만 사용하는 관계

- Serializable 인터페이스를 구현해야 함
- equals hashcode를 구현해야 함
- 기본 생성자가 있어야 하고 식별자 클래스는 public이어야 한다.

## 7.4 조인 테이블
조인 테이블을 사용하는 방법은 연관 관계를 관리하는 조인 테이블 을 추가하고 여기서 두 테이블의 외래 키를 가지고 연관관계를 관리

Q. 다대다 관계에서 조인 테이블을 사용하는건 국룰인데 일대다, 다대일에서는 조인 테이블을 사용해야 할 이유가 있을까?

## 7.5 엔티티 하나에 여러 테이블 매핑
- @SecondaryTable을 사용하면 한 엔티티에 여러 테이블을 매핑할 수 있다.
``` 
@Table(name = BOARD)
@SecondaryTable(name="BOARD_DETAIL", pkJoinColumns = @PrimaryKeyJoinColumn(name="BOARD_DETAIL_ID"))


@Column(table="BOARD_DETAIL")
```


+) 추가
상속과 다형성 같은게 많다면 mongodb써보는 것도 추천! 상속이 typealias로 깔끔하게 잘 되어 있다.

```
@Document(collection = "ALARM")
public class Alarm {
    @Id
    private ObjectId id;

    private ObjectId memberId;

    private String data;

    @Builder.Default
    @Indexed(expireAfter = "30d")
    private LocalDateTime createdAt = LocalDateTime.now();

    @Builder.Default
    private LocalDateTime expireAt = LocalDateTime.now().plusDays(30);

    private AlarmCategory category;
}

@Document(collection = "ALARM")
@TypeAlias("NoticeAlarm")
public class NoticeAlarm extends Alarm {
    private String redirectUrl;
}

@Document(collection = "ALARM")
@TypeAlias("FriendAlarm")
public class FriendAlarm extends Alarm {
    private ObjectId friendId;
}
```

![image](https://user-images.githubusercontent.com/46064193/135054169-f4e3c20a-860f-4228-b163-a20b14572d82.png)