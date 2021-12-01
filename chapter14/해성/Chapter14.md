# 14 컬렉션과 부가기능


## 14.1 컬렉션
@OneToMany, @ManyToMany 를 사용해서 일대다나 다대다 엔티티 관계를 매핑할 때
Collection, Set, List, Map 등 

## 14.1.1 JPA 와 컬렉션
```java
@Entity
public class Team {
    @Id
    private String id;
    @OneToMany
    @JoinColumn
    private Collection<Member> members = new ArrayList<>;
}
```

```
// before
persist: class java.util.ArrayList
// after
persist: class org.hibernate.collection.internal.PersistentBag 
```

출력 결과를 보면 원래 ArrayList 타입이었던 컬렉션이 엔티티를 영속 상태로 만든 직후에 하이버네이트가 제공하는 PersistentBag 타입으로 변경

하이버네이트는 컬렉션을 효율적으로 관리하기 위해 엔티티를 영속 상태로 만들 때 원본 컬렉션을 감싸고 있는 내장 컬렉션을 생성하고 이 내장 컬렉션을 사용하도록 참조를 변경한다. 

하이버네이트가 제공하는 내장 컬렉션은 래퍼 컬렉션으로도 부른다. 

이런 특징 때문에 컬렉션을 다음처럼 즉시 초기화해서 사용하는 것을 권장한다.

```java
Collection<Member> members = new ArrayList<Member>();
```

```java
// 인터페이스와 컬렉션 래퍼
// org.hibernate.collection.internal.PersistentBag
@OneToMany
Collection<Member> collection = new ArrayList<Member>();

// org.hibernate.collection.internal.PersistentBag
@OneToMany
List<Member> list = new ArrayList<Member>();

// org.hibernate.collection.internal.PersistentSet
@OneToMany
Set<Member> set = new HashSet<Member>();

// org.hibernate.collection.internal.PersistentList
@OneToMany
@OrderColumn
List<Member> orderColumnList = new ArrayList<Member>();
```

### 14.1.2 Collection, List
Collection, List 는 중복을 허용하는 컬렉션
PersistentBag 을 래퍼 컬렉션으로 사용
ArrayList 로 초기화하면 된다. 
엔티티를 추가할 때 중복을 검사하지 않고, 지연 로딩된 컬렉션을 초기화하지 않는다. 
```java
@Entity 
public class Parent {
    @Id
    @GeneratedValue
    private Long id;

    @OneToMany
    @JoinColumn
    private Collection<CollectionChild> collection = new ArrayList<CollectionChild>();

    @OneToMany
    @JoinColumn
    private List<ListChild> collection = new ArrayList<ListChild>();
}
```

### 14.1.3 Set
Set 은 중복을 허용하지 않는 컬렉션이다. 
PersistentSet 을 컬렉션 래퍼로 사용한다. 
HashSet 으로 초기화하면 된다. 
엔티티 추가시 중복비교가 필요하고, 지연로딩된 컬렉션을 초기화한다. 
```java
@Entity
public class Parent {
    @OneToMany
    @JoinColumn
    private Set<SetChild> set = new HashSet<SetChild>();
}
```

### 14.1.4 List + @OrderColumn
데이터베이스에 순서 값을 저장해서 조회할 때 사용한다는 의미

```java
@Entity
public class Board {
    @Id
    @GeneratedValue
    private Long id;

    private String title;
    private String content;
    
    @OneToMany(mappedBy = "board")
    @OrderColumn(name = "POSITION")
    private List<Comment> comments = new ArrayList<Comment>();
}

@Entity
public class Comment {
    @Id
    @GeneratedValue
    private Long id;
    
    private String comment;

    @ManyToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;
}
```

순서가 있는 컬렉션은 데이터베이스에 순서 값도 함께 관리한다. 
JPA 는 List 의 위치 값을 Comment 테이블의 POSITION 컬럼에 보관한다. 

#### @OrderColumn 단점
- POSITION 값을 update 하는 SQL 이 추가로 발생
- 중간에 데이터가 삭제된 경우 null 이 저장됨

### 14.1.5 @OrderBy
@OrderBy 는 데이터베이스의 order by 절을 사용해서 컬렉션을 정렬
모든 컬렉션에 사용 가능

```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "Team")
    @OrderBy("username desc, id asc")
    private Set<Member> members = new HashSet<Member>();
}

@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "MEMBER_NAME")
    private String username;

    @ManyToOne
    private Team team;
}
```

- @OrderBy 는 엔티티의 필드에 씀
- @OrderBy 의 값으로 username desc, id asc 로 정렬
- Set 에 @OrderBy 를 적용하면 HashSet 대신 LinkedHashSet 을 내부에서 사용함


## 14.2 @Converter
엔티티의 데이터를 변환해서 데이터베이스에 저장 가능

```java
@Entity
public class Member {
    @Id
    private String id;
    private String username;

    @Convert(converter = BooleanToYNConverter.class)
    private boolean vip;
}
```

```java
@Conveter(autoApply = true) // 글로벌 설정 가능
public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {
    /**
     * 엔티티 필드 -> 디비 컬럼
     */
    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return (attribute != null && attribute) ? "Y" : "N";
    }

    /**
     * 디비 컬럼 -> 엔티티 필드
     */
    @Override
    public Boolean convertToEntityAttribute(String dbData) {
        return "Y".equals(dbData);
    }
}
```

## 14.3 리스너
엔티티의 생명주기에 따른 이벤트를 처리 가능

### 14.3.1 이벤트 종류
- PostLoad : 엔티티가 영속성 컨텍스트에 조회된 직후 또는 refresh 를 호출한 후 (2차캐시에 저장되어 있어도 호출됨)
- PrePersist : persist 메서드를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출. 식별자는 아직 존재하지 않음. 새로운 인스턴스를 merge 할 때도 호출.
- PreUpldate : flush 나 commit 을 호출해서 엔티티를 데이터베이스에 수정하기 직전에 호출
- PreRemove : remove() 메서드를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출됨. 삭제명령어로 영속성 전이가 일어날 때도 호출. prphanRemoval 에 대해서는 flush 나 commit 시에 호출
- PostPersist : flush 나 commit 을 호출해서 엔티티를 데이터베이스에 저장한 직후 호출. 식별자가 항상 존재함. 
- PostUpdate : flush 나 commit 을 호출해서 엔티티를 데이터베이스에 수정한 직후 호출.
- PostRemove : flush 나 commit 을 호출해서 엔티티를 데이터베이스에서 삭제한 직후 호출.

### 14.3.2 이벤트 적용 위치
- 엔티티에 직접 적용
- 별도의 리스너 등록
- 기본 리스너 사용

#### 엔티티에 직접 적용
```java
@Entity
public class Duck {
    // ...
    @PrePersist
    public void prePersist() {

    }

    @PostPersist
    public void postPersist() {
        
    }
    // ...
}
```

#### 별도의 리스너 등록

리스너는 대상 엔티티를 파라미터로 받을 수 있다. 반환 타입은 void 로 설정해야 한다. 

```java
@Entity
@EntityListener(DuckListener.class)
public class Duck {
    // ...
}

public class DuckListener {
    @PrePersist
    private void prePersist(Object obj) {
        // ...
    }

    @PostPersist
    private void postPersist(Object obj) {
        // ...
    }
}

```

#### 기본 리스너 사용
여러 리스너를 등록했을 때 이벤트 호출 순서
1. 기본 리스너
2. 부모 클래스 리스너
3. 리스너
4. 엔티티

## 14.4 엔티티 그래프
엔티티 조회시 연관된 엔티티를 함께 조회하려면
1. FetchType=EAGER
2. 페치 조인 사용
3. 엔티티 그래프 

엔티티 그래프 기능은 엔티티 조회 시점에 연관된 엔티티들을 함께 조회하는 기능이다. 
1. 정적으로 정의하는 Named EntityGraph 와 
2. 동적으로 정의하는 엔티티 그래프가 있다. 

### 14.4.1 Named 엔티티 그래프
```java
@NamedEntityGraph(name = "Order.withMember", attributeNodes = (
    @NamedAttributeNode("member")
))
@Entity
@Table(name = "ORDERS")
public class Order {
    @Id 
    @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```

### 14.4.2 em.find() 에서 엔티티 그래프 사용
```java
EntityGraph graph = em.getEntityGraph("Order.withMember");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);
```

```sql
select o.*, m.*
from 
    ORDERS o
inner join 
    Member m
        on o.MEMBER_ID=m.MEMBER_ID
where
    o.ORDER_ID=?
```

### 14.4.3 subgraph
`Order -> OrderItem -> Item` 까지 조회하는 경우,
`Order -> OrderItem` 은 Order 가 관리하는 필드이지만
`OrderItem -> Item` 은 Order 가 관리하는 필드가 아님

```java
// Order.withAll 엔티티 그래프 정의
// Order -> Member, Order -> OrderItem, OrderItem -> Item 의 객체 그래프를 함께 조회
@NamedEntityGraph(name = "Order.withAll", attributeNodes = {
    @NamedAttributeNode("member"),
    @NamedAttributeNode(value = "orderItems", subgraph = "orderItems"),
    },
    // OrderItem -> Item 은 Order 의 객체그래프가 아님
    subgraphs = @NamedSubgraph(name = "orderItems", attributeNodes = {
        @NamedAttributeNode("item")
    })
)
@Entity
@Table(name = "ORDERS")
public class Order {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "MEMBER_ID")
    private Member member;      // 주문 회원

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<OrderItem>();
}

@Entity
@Table(name = "ORDER_ITEM")
public class OrderItem {
    @Id
    @GeneratedValue
    @Column(nmae = "ORDER_ITEM_ID")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "ITEM_ID")
    private Item item;
}

```


### 14.4.4 JPQL 에서 엔티티 그래프 사용

JPQL 에서 엔티티 그래프를 사용하는 방법은 em.find() 와 동일하게 힌트만 추가하면 된다. 

```java
List<Order> resultList = 
    em.creaeQuery("select o from Order o where o.id = :orderId",
        Order.class);
        .setParameter("orderId", orderId)
        .setHint("javax.persistence.fetchgraph", em.getEntityGraph("Order.withAll"))
        .getResultList();
```

```sql
# 실행된 sql
select o.*, m.*, oi.*, i.*
from
    ORDERS o
left outer join
    Member m
            on o.MEMBER_ID=m.MEMBER_ID
left outer join
    ORDER_ITEM oi
            on o.ORDER_ID=oi.ORDER_ID
left outer join
    Item i
        on oi.ITEM_ID=i.ITEM_ID
where
    o.ORDER_ID=?
```

> em.find() 에서 엔티티 그래프를 사용하면 하이버네이트는 필수 관계를 고려해서 SQL 내부 조인을 사용하지만 JPQL 에서 엔티티 그래프를 사용할 때는 항상 SQL 외부 조인을 사용한다. 만약 SQL내부 조인을 사용하려면 다음처럼 내부 조인을 명시하면 된다. 
```sql
select o 
from Order o 
join fetch o.member
where o.id = :orderId
```

### 14.4.5 동적 엔티티 그래프

```java
// 엔티티 그래프를 동적으로 구성하려면 createEntityGraph() 메소드를 사용하면 된다. 
publuc <T> EntityGraph<T> createEntityGraph(Class<T> rootType);
```

```java
// 동적으로 엔티티 그래프 생성 
EntityGraph<Order> graph = em.createEntityGraph(Order.class);
// Oder.member 속성을 엔티티 그래프에 포함
graph.addAttributeNodes("member");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);
```

```java
// 동적으로 엔티티 생성 및 그래프 subgraph 추가
EntityGraph<Order> graph = em.createEntityGraph(Order.class);
graph.addAttributeNodes("member");
Subgraph<OrderItem> orderItems = graph.addSubgraph("orderItems");
orderItems.addAttributeNodes("item");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);
```

### 14.4.6 엔티티 그래프 정리
- 엔티티 그래프는 항상 조회하는 엔티티의 ROOT 에서 시작해야 한다. 
- 영속성 컨텍스트에 해당 엔티티가 이미 로딩되어있으면 엔티티 그래프가 적용되지 않는다. 
- javax.persistence.fetchgraph : 엔티티 그래프에 선택한 속성만 함께 조회
- javax.persistence.loadgraph : 선택한 속성뿐만 아니라 FetchType.EAGER 로 설정된 연관관계도 포함
