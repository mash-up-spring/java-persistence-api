# `값 타입 분류`

JPA의 데이터 타입을 크게 분류하면 `엔티티 타입`과 `값 타입`으로 나눌 수 있습니다. 엔티티 타입은 @Entity로 정의하는 객체이고, 값 타입은 int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체를 말합니다.

엔티티 타입은 식별자를 통해 지속해서 추적할 수 있지만, 값 타입은 식별자가 없고 숫자나 문자같은 속성만 있으므로 추적할 수 없습니다.

- 기본 값 타입(int, Integer, String)
- 임베디드 타입(embedded type)
- 컬렉션 값 타입(collection value type)

<br> <br>

## `기본 값 타입`

```java
@Entity
public class Member {
    
    @Id @GeneratedValue
    private Long id;
    
    private String name;
    private int age;
    
}
```

Member 엔티티는 id라는 식별자 값을 가지고 생명주기도 있지만 값 타입인 name, age 속성은 식별자 값도 없고 생명주기도 회원 엔티티에 의존합니다. 즉, 회원을 삭제하면 이름, 나이 필드도 함께 삭제됩니다.
또 하나 중요한 점은 `값 타입은 공유하면 안된다는 것입니다. 예를들어, 회원 이름을 변경했는데 다른 회원의 이름이 함께 변경되면 문제가 되기 때문입니다.`

<br> <br>

## `임베디드 타입`

새로운 값 타입을 직접 정의해서 사용할 수 있는데, JPA에서는 이것을 `임베디드 타입`이라 합니다. 중요한 것은 임베디드 타입도 int, String 처럼 값 타입이라는 것입니다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    private LocalDateTime startDate;
    private LocalDateTime endDate;
    
    private String city; 
    private String street;
    private String zipcode;

}
```

만약 위의 Member 엔티티를 보면 `이름`, `근무 시작, 종류일`, `주소 도시, 번지, 우편번호`를 가지고 있습니다. 하지만 이렇게 이것은 단순히 정보만 나열해서 풀어둔 것입니다. 

왜냐하면 `근무 시작일과 우편번호는 서로 아무 관련이 없습니다.` 회원이 상세한 데이터를 그대로 가지고 있는 것은 객체지향적이지 않으며 응집력만 떨어뜨립니다. 즉, 이것을 수정하면 `회원 엔티티는 이름, 근무 기간, 집 주소`를 가진다고 하면 좀 더 명확하게 표현할 수 있습니다.

<img width="1226" alt="스크린샷 2021-09-02 오후 9 55 44" src="https://user-images.githubusercontent.com/45676906/131847336-17ad4b5a-e343-48d3-9c57-bcbe8c3a2ca9.png">

위와 같이 쉽게 말하면 Period, Address 클래스를 따로 만들어서 사용하고 Member가 id, name, Period, Address를 가지는 것입니다. 이것을 JPA에서는 아래와 같이 사용할 수 있습니다. 

<br> <br>

## `임베디드 타입`

- `@Embeddable`: 값 타입을 정의하는 곳에 표시
- `@Embedded`: 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수

<br> <br>

## `임베디드 타입과 테이블 매핑`

<img width="1085" alt="스크린샷 2021-09-02 오후 10 00 16" src="https://user-images.githubusercontent.com/45676906/131847966-71537bc2-d501-42fb-b683-62ce9a6d613c.png">

데이터베이스 테이블 입장에서는 임베디드를 쓰나 안쓰나 똑같습니다. 매핑만 잘 해주면 됩니다.

<br> <br>

## `예제 코드`

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAddress;

}
```
```java
@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

}
```
```java
@Embeddable
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;

}
```

위와 같이 Address, Period 클래스로 분리한 후에 `@Embeddable` 어노테이션을 사용해주면 됩니다. 임베디드 타입 덕분에 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능합니다. 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많습니다.
(`임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같습니다.`)

![스크린샷 2021-09-02 오후 10 06 48](https://user-images.githubusercontent.com/45676906/131848985-c8a61157-a762-43df-8416-265f9017f255.png)

<br> <br>

## `임베디드 타입과 연관관계`

<img width="1351" alt="스크린샷 2021-09-02 오후 10 11 59" src="https://user-images.githubusercontent.com/45676906/131849634-877c8ae5-bd9a-4c78-a3b3-f19b930b720c.png">

임베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 있습니다. (엔티티는 공유될 수 있으므로 참조한다는 표현을 사용하고, 값 타입은 특정 주인에 소속되고 논리적인 개념상 공유되지 않으므로 표현한다고 표현했습니다.)

<br> <br>

## `@AttributeOrride: 속성 재정의`

만약에 회원 엔티티에서 `회사주소`, `집 주소`가 필요해서 아래와 같이 필드를 하나 추가했다고 가정하겠습니다. 

![스크린샷 2021-09-02 오후 10 15 34](https://user-images.githubusercontent.com/45676906/131850184-5c025be3-f0b6-4f36-89f4-8eea1bd4049d.png)

위와 같이 중복된 임베디드 타입을 가진 상태로 실행하게 되면 어떻게 될까요?

<br>

![스크린샷 2021-09-02 오후 10 16 10](https://user-images.githubusercontent.com/45676906/131850270-e6681591-a6f1-4694-9619-34576cccb93f.png)

그러면 위와 같이 `Repeated column in mapping for entity` 라는 에러가 발생합니다. 이럴 때 사용하는 것이 `@AttributeOrride` 입니다.

<br> <br>

## `값 타입과 불변 객체`

값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념입니다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 합니다.

<br>

### `값 타입 공유 참조`

임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험합니다. 공유하면 어떤 문제가 발생하는지 알아보겠습니다.

<br> <br>

### `예제 코드`

![스크린샷 2021-09-02 오후 10 29 06](https://user-images.githubusercontent.com/45676906/131852220-af56b47c-ae3e-4272-9f96-7506491b3637.png)

위와 같이 Member 엔티티에 `임베디드 타입 Period`와 `엔티티 Address`가 존재합니다.

<br>

![스크린샷 2021-09-02 오후 10 33 21](https://user-images.githubusercontent.com/45676906/131852905-747d35d9-7dc5-47b6-ae5a-0e8d2f73dec0.png)

Member 엔티티를 2개 생성하는데 둘 다 동일한 Address를 생성하여 setter로 값을 넣었습니다. 

<br>

![스크린샷 2021-09-02 오후 10 34 04](https://user-images.githubusercontent.com/45676906/131852965-6bf747a3-8fe9-423c-9fbe-22e2eba645cb.png)

그리고 실행을 해보면 Member 엔티티를 2개 생성했기에 당연히 INSERT 쿼리가 2번 실행되는 것을 확인할 수 있습니다. 

<br>

![스크린샷 2021-09-02 오후 10 34 48](https://user-images.githubusercontent.com/45676906/131853077-d060f4a6-35f7-48a6-ad16-c84b2f2f9d76.png)

그리고 테이블을 확인해보면 위와 같이 값이 잘 들어간 것도 볼 수 있는데요. 그런데 여기서 Member1의 CITY 값을 한번 수정해보겠습니다. 

<br>

![스크린샷 2021-09-02 오후 10 35 54](https://user-images.githubusercontent.com/45676906/131853396-1e71f92f-4dae-49da-9ead-4ce78628546e.png)

위와 같이 `new Gyunny1 City`로 바꿔보겠습니다. 

<br>

![스크린샷 2021-09-02 오후 10 38 39](https://user-images.githubusercontent.com/45676906/131853716-a5ead8a2-3d33-4e7e-8e86-54f17f5510b3.png)

그런데 바꾼 것은 Member1 하나인데 UPDATE 쿼리는 2번 실행된 것을 볼 수 있습니다. 일단 바로 DB Table을 확인해보겠습니다.

<br>

![스크린샷 2021-09-02 오후 10 39 19](https://user-images.githubusercontent.com/45676906/131853866-9101a4ac-6cda-4fe2-a42d-1fb17bdec1e5.png)

테이블을 보니 Member1 뿐만 아니라 `Member2의 CITY 값도 같이 바뀐 것을 볼 수 있습니다.` 이런 버그는 복잡한 도메인에서 찾기가 정말 쉽지 않은데요. 이렇게 둘 다 바뀐 이유가 무엇일까요?

<img width="1056" alt="스크린샷 2021-09-02 오후 10 25 46" src="https://user-images.githubusercontent.com/45676906/131851719-c59844d9-dcbb-4973-807b-cca0e74ee733.png">

위와 같이 Member Entity가 동일한 Address를 참조하고 있기 때문에 Address의 값을 바꾸면 Member 엔티티 둘 다 영향을 받게 됩니다.  

<br> <br>

## `값 타입의 복사`

값 타입의 실제 인스턴스인 값을 공유하는 것은 위험합니다. 그래서 위와 같은 부작용을 없애기 위해서는 `값을 복사해서 사용해야 합니다.`

<img width="1069" alt="스크린샷 2021-09-02 오후 10 42 19" src="https://user-images.githubusercontent.com/45676906/131854302-87c93b89-43ac-4337-a7a8-2a046be19289.png">

<br>

![스크린샷 2021-09-02 오후 10 43 52](https://user-images.githubusercontent.com/45676906/131854636-2cd71c37-0455-4b68-b03c-0399252d005c.png)

그래서 이번에는 새로운 Address를 하나 더 만들어서 Member 엔티티에 넣어주는 것을 볼 수 있습니다. 그리고 이번에도 Address의 값을 하나 바꿔보겠습니다.

<br>

<img width="428" alt="스크린샷 2021-09-02 오후 10 44 57" src="https://user-images.githubusercontent.com/45676906/131854744-3748571c-6850-4a69-b306-d34a413ffd21.png">

이번에는 Address를 복사해서 저장했기 때문에 둘 다 바뀌지 않고, Member1의 값 하나만 바뀐것을 볼 수 있습니다.

<br> <br>

## `객체 타입의 한계`

위의 예시처럼 값을 복사해서 사용하면 공유 참조로 발생하는 부작용을 피할 수 있습니다. 문제는 `임베디드 타입처럼 직접 정의한 값은 자바의 Primitive Type이 아니라 객체 타입`이라는 것입니다. 

```java
Address a = new Address("Old");
Address b = a;
b.setCity("New");
```

위와 같이 `Address b = a`에서 a가 참조하는 인스턴스의 참조 값을 b에 넘겨줍니다. 따라서 a와 b는 같은 인스턴스를 공유 참조합니다. 즉, 마지막 줄에서 b만 City 값을 바꾸려 했어도 a 까지 같이 바뀌게 되는 부작용이 발생하는 것입니다.

물론 객체를 복사해서 하면 공유 참조를 피할 수 있지만, `문제는 복사하지 않고 원본의 참조 값을 직접 넘기는 것을 막을 방법이 없다는 것`입니다. 즉, 객체의 공유 참조는 피할 수 없습니다. 따라서 근본적인 해결책이 필요한데 가장 단순한 방법은 객체의 값을 수정하지 못하게 막으면 됩니다.
(예를들면 setter를 삭제하기,,)

<br> <br>

## `불변 객체`

값 타입은 부작용 걱정 없이 사용할 수 있어야 합니다. 부작용이 일어나면 값 타입이라 할 수 없습니다. `객체를 불변하게 만들면 값을 수정할 수 없으므로 부작용을 원척 차단할 수 있습니다.` 따라서 값 타입은 될 수 있으면 `불변 객체`로 설계해야 합니다. 즉, 한 번 만들면 절대 변경할 수 없는 객체를 `불변 객체`라고 합니다. 자바가 제공하는 대표적인 불변 객체는 `String`, `Integer`가 있습니다. 

![스크린샷 2021-10-20 오후 12 46 10](https://user-images.githubusercontent.com/45676906/138025109-66068ff8-7c28-4c7c-ab06-8f7511612f66.png)

위와 같이 생성자로만 값을 설정하고 `setter`를 만들지 않으면 Address를 `불변 객체`로 만들 수 있습니다. 

<br> <br> 

## `값 타입의 비교`

값 타입은 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야 합니다. 

```java
public class ValueMain {
    public static void main(String[] args) {
        int a = 10;
        int b = 10;

        System.out.println(a == b);  // true

        Address address1 = new Address("City", "Street", "10000");
        Address address2 = new Address("City", "Street", "10000");
        System.out.println(address1 == address2); // false
    }
}
```

위의 코드를 보면 결과가 당연하다고 생각할 것입니다. 이유를 정리하면 아래와 같습니다. 

- `동일성(identity) 비교`: `인스턴스의 참조 값`을 비교, == 사용
- `동등성(equivalence) 비교`: `인스턴스의 값`을 비교, equals() 사용
- 값 타입은 a.equals(b)를 사용해서 동등성 비교를 해야 합니다.
- 값 타입의 equals() 메소드를 적절하게 재정의해야 합니다.
 
<br> <br>

## `값 타입 컬렉션`

![스크린샷 2021-09-03 오전 5 32 27](https://user-images.githubusercontent.com/45676906/131912126-3cd7b978-37a1-4ff0-8cff-27aab8c0b082.png)

자바에서는 객체에 컬렉션 타입으로 필드를 가질 수 있지만, 데이터베이스에서는 컬렉션을 저장할 수 없기 때문에 위와 같이 별도의 테이블로 만들어서 저장해야 합니다. 위의 그림처럼 테이블이 설계가 되도록 엔티티 설계를 해보겠습니다.

![스크린샷 2021-10-20 오후 1 06 01](https://user-images.githubusercontent.com/45676906/138026742-b7a3f39a-5d9e-41e5-827f-5ba64d824765.png)

값 타입을 하나 이상 저장하려면 컬렉션에 보관하고 `@ElementCollection, @CollectionTable` 두 개의 애노테이션을 사용하면 됩니다. 위와 같이 설정한 후에 바로 실행을 해보겠습니다.

<br>

![스크린샷 2021-10-20 오후 1 08 02](https://user-images.githubusercontent.com/45676906/138026874-5fa6c7f7-d3c2-4ef2-8452-fc1219f0e932.png)

먼저 `CollectionTable`을 통해서 테이블 이름을 `Address`로 지정한 것이 만들어진 것을 볼 수 있습니다. 

<br>

![스크린샷 2021-10-20 오후 1 09 55](https://user-images.githubusercontent.com/45676906/138027032-9768d8be-a00b-45df-b390-be23e681731c.png)

그리고 `FAVORITE_FOOD` 테이블도 `MEMBER_ID`로 지정한 외래키와 같이 테이블이 생성된 것도 확인할 수 있습니다.

<br>

![스크린샷 2021-10-20 오후 1 10 58](https://user-images.githubusercontent.com/45676906/138027122-7c9ce176-f49d-470b-bb2e-5e48151cfab2.png)

마지막으로 `Member` 테이블도 잘 생성이 된 것을 볼 수 있습니다.

<br>

![스크린샷 2021-10-20 오후 1 17 54](https://user-images.githubusercontent.com/45676906/138027667-f4df6623-0b97-43bf-b5aa-bd449b03527e.png)

이번에는 Member 엔티티를 생성한 후에 Member가 지금까지 거주했던 주소들, 좋아하는 음식들을 List, Set에 저장하는 코드입니다. 그리고 마지막에 `em.persist()`를 통해서 Member만 persist 한 것을 볼 수 있습니다.

마지막에 Member만 Persist 했는데 `INSERT` 쿼리는 몇번 실행이 될까요?

![스크린샷 2021-10-20 오후 1 24 36](https://user-images.githubusercontent.com/45676906/138028141-41c5e5d4-6d0c-4dbd-b88a-6eb0e74683cf.png)

`Member` 엔티티의 INSERT 쿼리가 실행되는 예측할 수 있는데요. 나머지 값 타입의 INSERT 쿼리는 어떻게 실행되었는지 궁금한데요.

<br>

![스크린샷 2021-10-20 오후 1 25 40](https://user-images.githubusercontent.com/45676906/138028286-b2404739-3348-49f3-b246-6fccd9ecca2e.png)

`Favorite_Food`와 `Address`는 persist 하지 않았는데 INSERT 쿼리가 실행된 것을 볼 수 있는데요. 이것이 가능한 이유는 `값 타입 컬렉션은 영속성 전이(cascade) + 고아 객체 제거(ORPHAN REMOVE) 기능을 필수로 가지기 때문입니다.`

<br> <br>

## `값 타입 조회`

![스크린샷 2021-10-20 오후 1 31 02](https://user-images.githubusercontent.com/45676906/138028775-63ded652-94dc-4577-91de-4edd219a7ed8.png)

그리고 위의 코드에서 추가로 Member 엔티티 하나 조회해보겠습니다. 

<br>

![스크린샷 2021-10-20 오후 1 34 00](https://user-images.githubusercontent.com/45676906/138028889-741522bb-cdb6-460f-aa65-b047e7d16824.png)

SELECT 쿼리가 나간 것을 보면 Member 엔티티만 조회해온 것을 볼 수 있는데요.(임베디드 타입은 제외) 이것을 보면 `값 타입 컬렉션은 조회할 때 페치 전략이 Default가 LAZY인 것을 볼 수 있습니다.`

<br> <br>

## `값 타입 수정`

```java
Member findMember = em.find(Member.class, member.getId());

// findMember.getHomeAddress().setCity("newCity");   값 타입 부작용 생길 수 있음 !!
findMember.setHomeAddress(new Address("newCity", "new Street", "new Zipcode"));
```

만약에 값 타입을 수정하고 싶다면 `새로운 값 타입을 만들어서 넣어야 합니다.` 위에서 말했던 것처럼 값 타입을 `불변 객체`로 사용하지 않으면 예상치 못한 버그가 발생할 수 있기 때문에 위와 같이 새로운 값 객체를 생성해서 수정해야 합니다. 

![스크린샷 2021-10-20 오후 1 39 57](https://user-images.githubusercontent.com/45676906/138029384-8768fcfb-5676-4ac5-9227-28084c1d780f.png)

UPDATE 쿼리도 원했던 대로 잘 실행된 것도 확인할 수 있습니다.

<br>

```java
findMember.getFavoriteFoods().remove("치킨");
findMember.getFavoriteFoods().add("짜장면");
```

Set에 저장된 값 타입을 바꿀 때에도 `remove`를 통해서 삭제하고, `add`를 통해서 다시 저장해야 합니다.  

![스크린샷 2021-10-20 오후 1 42 39](https://user-images.githubusercontent.com/45676906/138029547-19816702-612e-421f-a35a-0c96794d9138.png)

이렇게 컬렉션에 remove, add만 하더라도 `DELETE`, `INSERT` 쿼리가 나가는 것을 볼 수 있습니다.

<br> 

![스크린샷 2021-10-20 오후 2 09 12](https://user-images.githubusercontent.com/45676906/138031751-47e94c1f-0523-465b-9cef-fda3fbba26aa.png)

그리고 위에서 `Address`를 바꿀 때는 새로운 Address를 객체를 만들어서 넣어야 한다고 했는데요. 이번에 `old1`을 `new City`로 바꾸고 싶을 때 `old1`을 remove 하고 `new City`를 add 하는 예제를 실행해보겠습니다. (참고로 remove 는 equals 기반으로 실행되기 때문에 Address에 `equals`, `hashCode`를 재정의해야 합니다.)

![스크린샷 2021-10-20 오후 2 11 36](https://user-images.githubusercontent.com/45676906/138031937-3ff8deb2-b314-43c0-b00e-25ef1fe8f661.png)

위의 코드를 실행한 후에 출력되는 쿼리를 보면 `DELETE` 쿼리 1번, `INSERT` 쿼리 2번이 실행된 것을 볼 수 있습니다. 거기에다 DELETE 쿼리는 해당 멤버가 가진 주소 전체를 삭제하고 다시 하나씩 INSERT 하는 것을 볼 수 있는데요.(위의 예제는 2개라서 그렇지 100개라면 101번 쿼리가 실행될 것입니다.) 예상했던 쿼리는 DELETE 1번, INSERT 1번씩 실행되는 것이었는데.. 너무 예상과는 다르게 실행되는 거 같은데요. 이것이 `값 타입 컬렉션의 제약사항` 입니다.

<br>

![스크린샷 2021-10-20 오후 2 40 17](https://user-images.githubusercontent.com/45676906/13₩₩8034550-c3735166-9a44-4e67-b385-93b61641ebe4.png)

위의 코드는 `ArrayList`의 `remove()` 메소드 내부 코드입니다. 내부 코드를 보면 `equals`를 통해서찾는 것을 볼 수 있습니다.([Equals를 재정의하거든 HashCode도 재정의해야 하는 이유](https://devlog-wjdrbs96.tistory.com/255?category=925183)) 

<br> <br>

## `값 타입 컬렉션의 제약사항`

- 값 타입은 엔티티와 다르게 식별자 개념이 없습니다. 즉, 값을 변경하면 추적이 어렵습니다.

- 값 타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관됩니다. 따라서 여기에 보관된 값 타입의 값이 변경되면 데이터베이스에 있는 원본 데이터를 찾기 어렵다는 문제가 있습니다. 이런 문제로 인해 JPA 구현체들은 값 타입 컬렉션에 변경 사항이 발생하면, 값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데아터를 삭제하고, 현재 값 타입 컬렉션 객체에 있는 모든 값을 데이터베이스에 다시 저장합니다.

- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 합니다. (null 입력 X, 중복 저장 X)

<br> <br>

## `값 타입 컬렉션 대안`

- 실무에서는 상황에 따라 `값 타입 컬렉션을 사용하는 대신에 새로운 엔티티를 만들어서 일대다 관계로 설정하면 됩니다.`
- 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용합니다. 
- 영속성 전이 + 고아 객체 제거를 사용해서 값 타입 컬렉션처럼 사용합니다. 

<br>

![스크린샷 2021-10-20 오후 2 24 45](https://user-images.githubusercontent.com/45676906/138033144-4ee54b5e-c0d7-400f-bc95-a596fb22849e.png)

<br>

![스크린샷 2021-10-20 오후 2 23 49](https://user-images.githubusercontent.com/45676906/138033037-a8e33a98-2ef4-4d67-ada9-1185761ab50c.png)

`AddressEntity`를 하나 만든 후에 Member Entity와 `@OneToMany`, `Cascade.ALL`, `orphanRemoval = true`로 설정해서 하는 방법도 존재합니다.

<br> <br>

## `엔티티 타입의 특징`

- 식별자(@Id)가 있습니다.
  - 엔티티 타입은 식별자가 있고 식별자로 구별할 수 있습니다.
- 생명 주기가 있습니다.
  - 생성하고 영속화하고, 소멸하는 생명 주기가 있습니다.
  - em.persist(entity)로 영속화 합니다.
  - em.remove(entity)로 제거합니다.
  
- 공유할 수 있습니다.
  - 참조 값을 공유할 수 있고, 이것을 공유 참조라 합니다.
  - 예를 들어 회원 엔티티가 있다면 다른 엔티티에서 얼마든지 회원 엔티티를 참조할 수 있습니다.

<br> <br>

## `값 타입의 특징`

- 식별자가 없습니다. 
- 생명 주기를 엔티티에 의존합니다.
  - 스스로 생명 주기를 가지지 않고 엔티티에 의존합니다. 의존하는 엔티티를 제거하면 같이 제거됩니다.

- 공유하지 않는 것이 안전합니다.
  - 엔티티 타입과는 다르게 공유하지 않는 것이 안전합니다. 대신에 `값을 복사해서 사용`해야 합니다.
  - 오직 하나의 주인만이 관리해야 합니다.
  - `불변 객체`로 만드는 것이 안전합니다.