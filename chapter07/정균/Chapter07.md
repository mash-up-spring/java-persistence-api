# `상속 관계 매핑`

관계형 데이터베이스에는 객체지향 언어에서 다루는 상속이라는 개념이 없습니다. 대신에 `슈퍼타입 서브타입 관계`라는 모델링 기법이 객체지향의 상속과 유사합니다. 

<img width="1263" alt="스크린샷 2021-08-27 오후 4 47 42" src="https://user-images.githubusercontent.com/45676906/131092053-daa7cdbf-35fa-4dec-8093-52363215dd19.png">

슈퍼타입 서브 타입 논리 모델을 실제 물리 모델로 구현하는 방법은 3가지가 있습니다.

- 각각의 테이블로 변환: 각각을 모두 테이블로 만들고 조회할 때 조인을 사용합니다. JPA에서는 `조인 전략`이라 합니다.
- 통합 테이블로 변환: 테이블을 하나만 사용해서 통합합니다. JPA에서는 `단일 테이블 전략`이라고 합니다.
- 서브타입 테이블로 변환: 서브 타입마다 하나의 테이블을 만듭니다. 

<br> <br>

## `조인 전략`

<img width="1421" alt="스크린샷 2021-09-02 오후 3 35 39" src="https://user-images.githubusercontent.com/45676906/131794349-2bfa620d-8f02-427b-bd4f-2e96aaf1d873.png">

조인 전략은 위의 그림처럼 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본키 + 외래 키로 사용하는 전략입니다.

<br>

### 장점

- 테이블이 정규화 됩니다.
- 외래 키 참조 무결성 제약조건을 활용할 수 있습니다.
- 저장공간을 효율적으로 사용합니다.

<br>

### 단점

- 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있습니다.
- 조회 쿼리가 복잡합니다.
- 데이터를 등록할 INSERT SQL을 두 번 실행합니다.

<br>

```java
@Entity
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```

```java
@Entity
public class Book extends Item {

    private String author;
    private String isbn;
}

```

```java
@Entity
public class Album extends Item {

    private String artist;
}
```

```java
@Entity
public class Movie extends Item {

    private String director;
    private String actor;
}
```

위의 그림에 나오는 관계처럼 Item을 상속하는 `Movie`, `Album`, `Book` 클래스를 만들었습니다. 이 상태로 실행한 후에 JPA가 만들어주는 DDL문을 보겠습니다. 

![스크린샷 2021-09-02 오후 3 52 12](https://user-images.githubusercontent.com/45676906/131796495-9b2ea19f-a60e-4faa-98a5-0307e41c27d9.png)

기본 전략 자체가 Item 테이블 안에 모든 컬럼을 만든 것을 볼 수 있습니다. 즉, 한 테이블 안에 다 넣는 것이 기본 전략입니다.

<br>

![스크린샷 2021-09-02 오후 3 56 01](https://user-images.githubusercontent.com/45676906/131796974-bac77aee-53db-4e7b-ab87-0b3e7edf3219.png)

그래서 이번에는 strategy를 `JOINED`로 주고 실행을 해보겠습니다.

<br>

![스크린샷 2021-09-02 오후 3 57 39](https://user-images.githubusercontent.com/45676906/131797255-9d51f8a5-f8cd-42ff-8d98-7ce151c03ecd.png)

이번에는 테이블이 모두 각각 생긴 것을 볼 수 있습니다. 위에서 말한 `JOIN 전략`을 사용한 것입니다. 

<br>

![스크린샷 2021-09-02 오후 4 02 19](https://user-images.githubusercontent.com/45676906/131797796-fc281a0d-912b-47a9-bb37-013e5e256cb9.png)

그리고 위와 같이 Moview 객체를 저장한 후에 실행해보겠습니다. 

<br>

![스크린샷 2021-09-02 오후 4 03 24](https://user-images.githubusercontent.com/45676906/131797957-80a401f6-526b-4d0e-9459-c7d8b49517f9.png)

그러면 위와 같이 INSERT 쿼리가 2번 실행된 것을 볼 수 있습니다. 

<br>

<img width="233" alt="스크린샷 2021-09-02 오후 4 04 37" src="https://user-images.githubusercontent.com/45676906/131798077-8aed9905-1902-4c4c-b4c5-e2289e9b8a4f.png">

데이터도 테이블에 잘 들어간 것을 볼 수 있습니다. 

<br>

![스크린샷 2021-09-02 오후 4 07 58](https://user-images.githubusercontent.com/45676906/131798533-4378bc94-41d9-421e-876a-e8dc11ab7e9f.png)

그러면 이번에는 위와 같이 Movie 테이블을 조회하면 JPA가 어떤 쿼리를 만들어줄까요?

<br>

![스크린샷 2021-09-02 오후 4 09 12](https://user-images.githubusercontent.com/45676906/131798651-df5e67ad-9930-42d7-9808-4889ff8dd08e.png)

Movie, Item 테이블을 `INNER JOIN` 쿼리를 생성해주는 것을 볼 수 있습니다.  

<br>

![스크린샷 2021-09-02 오후 4 10 57](https://user-images.githubusercontent.com/45676906/131798876-a8a62835-58ca-4b5b-8027-dd3ca82e3ee6.png)

위의 조언 전략에 대한 그림에서 보면 `DTYPE` 이라는 것이 존재하는 것을 보았을텐데요. 이번에는 위와 같이 `@DiscriminatorColumn` 어노테이션을 사용해서 실행해보겠습니다. 

<br>

![스크린샷 2021-09-02 오후 4 12 45](https://user-images.githubusercontent.com/45676906/131799188-ca8c69c8-0c7b-4302-807a-d97c2cf5ee1e.png)

그러면 위와 같이 `DTYPE`으로 생성이 되는 것을 볼 수 있습니다. 

<br>

<img width="244" alt="스크린샷 2021-09-02 오후 4 14 36" src="https://user-images.githubusercontent.com/45676906/131799385-137f44e0-9ff8-4c28-94a5-07974d2a86ee.png">

`@DiscriminatorColumn` 어노테이션을 사용하면 DTYPE이 생성되어 Entity 이름이 저장되는 것을 볼 수 있습니다. 사용하여 어떤 엔티티를 통해서 저장된 것이 알기 편하기 때문에 사용하는 것이 좋습니다.

<br> <br>

## `단일 테이블 전략`

<img width="1151" alt="스크린샷 2021-09-02 오후 3 41 19" src="https://user-images.githubusercontent.com/45676906/131795092-3ea03c30-271b-40bf-8e30-f9f833c92add.png">

단일 테이블 전략은 이름 그대로 테이블을 하나만 사용합니다. ITEM 테이블에 있는 ID와 MOVIE 테이블에 ID랑 똑같은 키입니다.

<br>

### 장점

- 조인이 필요 없으므로 일반적으로 조회 성능이 빠릅니다.
- 조회 쿼리가 단순합니다.

<br>

### 단점

- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 합니다.
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있습니다. 그러므로 상황에 따라서는 조회 성능이 오히려 느려질 수 있습니다.

<br>

### `특징`

- 구분 컬럼을 꼭 사용해야 합니다. 따라서 `@DiscriminatorColumn`을 꼭 설정해야 합니다.
- 즉, 한 테이블에 모든 정보를 다 저장하고 DTYPE을 통해서 구분하는 것입니다.

<br>

![스크린샷 2021-09-02 오후 4 20 39](https://user-images.githubusercontent.com/45676906/131800207-e2c39f7c-b66c-44a4-bac2-95446a59be86.png)

위와 같이 strategy 하나만 변경하면 됩니다. 

<br>

![스크린샷 2021-09-02 오후 4 21 28](https://user-images.githubusercontent.com/45676906/131800304-12289d66-5802-4f17-9c96-70cd2d8096ba.png)

그러면 위와 같이 테이블 하나에 모든 정보를 다 생성하는 것을 볼 수 있습니다. (다른 테이블은 생성되지 않았습니다.)

<br>

![스크린샷 2021-09-02 오후 4 22 39](https://user-images.githubusercontent.com/45676906/131800497-9567ebe9-c003-4d93-9e54-bbac7ba35708.png)

그리고 실행되는 쿼리만 보아도 INSERT 쿼리도 1번만 실행되고 SELECT 쿼리도 JOIN 필요 없이 간단하게 가져오는 것을 볼 수 있습니다. 
참고로 `단일 테이블 전략은 @DiscriminatorColumn 어노테이션을 사용하지 않아도 DTYPE이 자동으로 생성됩니다.`

<br> <br>

## `구현 클래스마다 테이블 전략`

<img width="1413" alt="스크린샷 2021-09-02 오후 3 42 27" src="https://user-images.githubusercontent.com/45676906/131795228-237dd84a-a5ad-4b3c-8cbc-9127e2da426c.png">

구현 클래스마다 테이블 전략은 위와 같이 자식 엔티티마다 테이블을 만듭니다. 즉, Item 테이블을 없애고 NAME, PRICE 같은 속성들은 중복되도록 허용하는 것입니다.
(일반적으로 추천하지 않는 전략입니다.)

<br>

### 장점

- 서브 타입을 구분해서 처리할 때 효과적입니다.
- not null 제약 조건을 사용할 수 있습니다.

<br>

### 단점

- 여러 자식 테이블을 함께 조회할 때 성능이 느립니다.(SQL에 Union 사용..)
- 자식 테이블을 통합해서 쿼리하기 어렵습니다.

<br>

![스크린샷 2021-09-02 오후 4 27 51](https://user-images.githubusercontent.com/45676906/131801260-954bcd8b-68e7-45d6-bac6-5b0117072897.png)

위와 같이 `TABLE_PER_CLASS` 속성을 주면 됩니다. 그리고 Item 클래스는 지금까지 추상 클래스로 안만들었지만 추상 클래스로 사용해야 합니다.

<br>

![스크린샷 2021-09-02 오후 4 33 34](https://user-images.githubusercontent.com/45676906/131801963-51f06c6f-d367-41e7-9c7d-7c7298a6eab8.png)

위와 같이 Movie Key로 Item을 찾아오면 어떤 쿼리가 실행될까요?

<br>

![스크린샷 2021-09-02 오후 4 34 32](https://user-images.githubusercontent.com/45676906/131802086-f53cede7-ed0e-4722-9f65-d58009190215.png)

엄청나게 복잡한 UNION을 사용한 쿼리가 발생하는 것을 볼 수 있습니다. 즉, 이 전략은 추천하지 않기에 다른 전략을 사용하는 것이 좋습니다.(사용하면 안되는 전략..) 

<br> <br>

## `@MappedSuperclass`

<img width="1151" alt="스크린샷 2021-09-02 오후 4 37 07" src="https://user-images.githubusercontent.com/45676906/131802482-138987bd-f4dd-4a2f-812d-a2b89171b998.png">

지금까지 학습한 상속 관계 매핑은 부모 클래스와 자식 클래스를 모두 데이터베이스 테이블과 매핑했습니다. 부모 클래스는 테이블과 매핑하지 않고 부모 클래스를 상속 받는 자식 클래스에게 매핑 정보만 제공하고 싶으면 `@MappedSuperclass`을 사용합니다.

<br>

### 예제 코드

![스크린샷 2021-09-02 오후 4 41 43](https://user-images.githubusercontent.com/45676906/131803096-0816fc86-1c69-4848-b74e-f349c6fb5f01.png)

만약에 위와 같이 모든 테이블에 `생성 시간`, `변경 시간`을 저장해야 한다는 요구사항이 생겼다면 어떻게 할 수 있을까요? 위와 같이 모든 엔티티에 필드를 다 추가해야 할까요?

이럴 때 사용하면 좋은 것이 `@MappedSuperclass`입니다.

![스크린샷 2021-09-02 오후 4 43 52](https://user-images.githubusercontent.com/45676906/131803405-3ee21899-8715-474a-b3ae-855305cfd702.png)

위와 같이 `BaseEntiy`라는 부모 클래스를 만들고 `@MappedSuperclass` 어노테이션을 추가한 후에 공통적으로 사용할 컬럼들을 적어주면 됩니다.

<br>

![스크린샷 2021-09-02 오후 4 44 52](https://user-images.githubusercontent.com/45676906/131803518-65f5696d-d7fc-44dc-a8fe-8e30322f19e7.png)

그리고 위와 같이 공통 필드를 사용할 엔티티는 BaseEntity 클래스를 상속 받으면 됩니다. 

![스크린샷 2021-09-02 오후 5 37 48](https://user-images.githubusercontent.com/45676906/131811576-0f6e7fe3-c4f4-4ac1-a82c-f52f11383427.png)

그러면 위와 같이 Member 테이블을 생성할 때 자동으로 BaseEntity에 존재하는 필드들이 생성되는 것을 볼 수 있습니다. 

<br> <br>

## `@MappedSuperclass 특징`

- 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해 사용합니다.
- `@MappedSuperclass`로 지정한 클래스는 엔티티가 아니므로 `em.find()`나 JPQL에서 사용할 수 없습니다.
- 이 클래스를 직접 생성해서 사용할 일은 거의 없으므로 추상 클래스로 만드는 것을 권장합니다.

<br>

즉, `@MappedSuperclass`는 테이블과는 관계가 없고 단순히 엔티티가 공통으로 사용하는 매핑 정볼르 모아주는 역할을 할 뿐입니다.