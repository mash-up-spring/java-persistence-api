# `프록시란?`

<img width="1273" alt="스크린샷 2021-08-27 오후 5 21 08" src="https://user-images.githubusercontent.com/45676906/131096558-cf070efd-d13c-4be2-a0a3-da8f61d4d5d3.png">

회원 엔티티를 조회할 때 연관된 팀 엔티티는 비즈니스 로직에 따라 사용될 수도 있지만, 사용되지 않을 수도 있습니다. 연관관계가 걸려있다 해서 Member를 조회할 때 Team 까지 같이 가져오는 것은 비효율적일 것입니다. 
어느 경우에는 Member만 가져오고 어느 경우에는 Member, Team을 같이 가져오는 것이 효율적일 것입니다. 

JPA는 이런 문제를 해결하려고 엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 방법을 제공하는 이것을 `지연 로딩`이라고 합니다.
`지연 로딩 기능을 사용하려면 실제 엔티티 객체 대신에 데이터베이스 조회를 지연할 수 있는 가짜 객체가 필요한데 이것을 프록시 객체라고 합니다.`

<br> <br>

## `프록시 기초`

지금까지는 `em.find()`를 통해서 객체를 조회했었는데요. em.find()를 통해서 조회하면 실제 엔티티 객체를 조회하는 것입니다. 그와 비슷하게 `em.getReference()` 라는 것이 존재합니다. 
이건 `데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체를 조회하는 것`입니다. 

<img width="873" alt="스크린샷 2021-08-27 오후 5 29 01" src="https://user-images.githubusercontent.com/45676906/131097591-caf46949-ff68-4aca-8446-41045c018f80.png">

즉, 다시 정리하면 엔티티를 실제 사용하는 시점까지 데이터베이스 조회를 미루고 싶을 때 `em.getReference()`를 사용하면 됩니다. 이 메소를 호출할 때 JPA는 데이터베이스를 조회하지 않고 실제 엔티티 객체도 생성하지 않습니다. 
대신에 데이터베이스 접근을 위임한 `프록시 객체가 반환됩니다.`

<br>

![스크린샷 2021-08-27 오후 5 35 11](https://user-images.githubusercontent.com/45676906/131098398-f6002d11-7ba1-4ead-8d16-808f57267033.png)

위의 코드를 보면 Member를 저장한 후에 영속성 컨텍스트를 비우고 다시 `em.find()`를 통해서 Member를 SELECT 하고 있습니다. 이 때 JPA는 어떠한 쿼리를 생성할까요?

![스크린샷 2021-08-27 오후 5 36 37](https://user-images.githubusercontent.com/45676906/131098622-1efb18bb-648c-4ffb-a5ab-54b9b00906ae.png)

쿼리를 보면 Member만 가져오도록 했지만 연관관계가 있는 Team 까지 JOIN해서 같이 가져오는 것을 볼 수 있습니다. 

<br>

![스크린샷 2021-08-27 오후 5 39 14](https://user-images.githubusercontent.com/45676906/131098949-8ccadcf2-0e64-48d5-adae-d098d6a99ae2.png)

이번에는 `em.getReference()`를 사용해서 Member를 조회한 후에 JPA가 쿼리를 언제 어떻게 생성하는지 보겠습니다. 

![스크린샷 2021-08-27 오후 5 40 12](https://user-images.githubusercontent.com/45676906/131099117-03fb3d34-39d8-4b0d-8aa6-e9f506bb103d.png)

이번에도 생성한 쿼리는 똑같은데 보면 `em.getReference()` 했을 때 바로 쿼리가 실행되는 것이 아니라 Member의 필드를 접근할 때 쿼리가 실행되는 것을 볼 수 있습니다.
그런데 id에 접근할 때는 쿼리가 나가지 않고 name에 접근하려고 하니까 쿼리가 실행된 것도 볼 수 있는데요. id는 이미 `em.getReference()`에서 매개변수로 넘겨줬기 때문에 알고 있는 값이라서 쿼리가 실행되지 않았던 것입니다. 

<br>

![스크린샷 2021-08-27 오후 5 43 47](https://user-images.githubusercontent.com/45676906/131099589-9b96afe0-963c-4cbf-8b69-fc5a1a47c9e9.png)

그리고 이번에는 `em.getReference()`로 찾아온 Member의 getClass를 호출해보면 아래와 같습니다. 

![스크린샷 2021-08-27 오후 5 44 56](https://user-images.githubusercontent.com/45676906/131099725-0f94ffcd-ff90-4cdd-afac-6d8e37529c19.png)

결과를 보면 `Proxy`라고 출력되는 것을 볼 수 있습니다. 즉, Hibernate가 만들어내는 가짜 클래스라는 것을 알 수 있습니다. 

<br> <br>

## `프록시 특징`

<img width="453" alt="스크린샷 2021-08-27 오후 5 46 34" src="https://user-images.githubusercontent.com/45676906/131099934-c0e5de57-fa62-4841-abf5-84adf53d9f47.png">

프록시는 실제 엔티티를 상속 받아서 만들어집니다. 그래서 실제 클래스와 겉모양이 같습니다. 따라서 사용하는 입장에서는 이것이 진짜 객체인지 프록시 객체인지지 구분하지 않고 사용하면 됩니다. 

<br>

<img width="1072" alt="스크린샷 2021-08-27 오후 5 48 19" src="https://user-images.githubusercontent.com/45676906/131100170-5cc6fd19-7bb1-45fb-b81b-e25ecf724992.png">

프록시 객체는 실제 객체의 참조를 보관하고 있습니다. 그리고 프록시 객체의 메소드를 호출하면 프록시 객체는 실제 객체의 메소드를 호출합니다. 

<br> <br>

## `프록시 객체의 초기화`

프록시 객체는 위의 코드 예시에서 본 거 처럼 `findMember.getUserName()` 처럼 실제 사용될 때 데이터베이스를 조회해서 실제 엔티티 객체를 생성하는데 이것을 `프록시 객체의 초기화`라고 합니다. 

<img width="1158" alt="스크린샷 2021-08-27 오후 5 50 39" src="https://user-images.githubusercontent.com/45676906/131100460-9d9e93ba-6615-455b-a701-5e37150863a7.png">
 
1. 프록시 객체에 member.getName()을 호출해서 실제 데이터를 조회합니다.
2. 프록시 객체는 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티 생성을 요청하는데 이것을 `초기화`라고 합니다.
3. 영속성 컨텍스트는 데이터베이스를 조회해서 실제 엔티티 객체를 생성합니다.
4. 프록시 객체는 생성된 실제 엔티티 객체의 참조를 Member Target 멤버 변수에 보관합니다.
5. 프록시 객체는 실제 엔티티 객체의 getName()을 호출해서 결과를 반환합니다.

<br> <br>

## `프록시의 특징`

- 프록시 객체는 처음 사용할 때 한 번만 초기화됩니다.
- 프록시 객체를 초기화될 때 `프록시 객체가 실제 엔티티로 바뀌는 것이 아닙니다. 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근 가능한 것입니다.`

![스크린샷 2021-08-27 오후 5 57 12](https://user-images.githubusercontent.com/45676906/131101397-e1ac6222-7501-4a85-9bac-b89b1bd2f116.png)

<br>

![스크린샷 2021-08-27 오후 5 57 52](https://user-images.githubusercontent.com/45676906/131101520-3e53934e-bbb3-4f83-8b10-cfa972e13103.png)

위의 코드와 결과를 보면 SELECT 쿼리도 한번만 실행되는 것을 볼 수 있고 여러 번 조회해도 둘 다 `Proxy 객체`인 것을 확인할 수 있습니다.

- 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시에 주의해야 사용해야 합니다. 
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 `em.getReference()`를 호출해도 실제 엔티티를 반환합니다. 

<br>

![스크린샷 2021-08-27 오후 6 42 55](https://user-images.githubusercontent.com/45676906/131107754-fa1597e1-e016-474a-b643-157d4c2ac2b8.png)

<br>

![스크린샷 2021-08-27 오후 6 44 03](https://user-images.githubusercontent.com/45676906/131107823-41eee4ee-f8a9-4e12-a132-f46e85fdbabc.png)

위와 같이 위에서 `em.find()`를 통해서 Member 클래스를 영속성 컨텍스트의 1차 캐시에 올려놓았으면 `em.getReference()`로 객체를 조회해도 Proxy가 반환되는 것이 아니라 1차 캐시에 있는 진짜 Member 객체가 반환이 되는 것을 볼 수 있습니다.

- `영속성 컨텍스트의 도움을 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생합니다. (실제 개발하다 보면 반드시 만날 수 있는 에러!!)`

<br>

![스크린샷 2021-08-27 오후 6 56 03](https://user-images.githubusercontent.com/45676906/131109550-c2c2e2a8-e3f6-488b-9eee-033b14cf2564.png)

<br>

![스크린샷 2021-08-27 오후 7 00 34](https://user-images.githubusercontent.com/45676906/131110103-c6a67e5f-e090-47ee-8093-3cca3ca13853.png)

코드에서 프록시 객체를 영속성 컨텍스트에서 제거한 후에 초기화를 시도했기 때문에 위와 같이 에러가 발생하는 것을 볼 수 있습니다. 즉, 위에서 보았듯이 프록시 객체를 초기화 하려면 영속성 컨텍스트가 필요한데 `detach`를 영속성 컨텍스트와 분리시켰기 때문에 에러가 발생하는 것입니다. 

<br> <br>

## `즉시로딩과 지연로딩`

### `지연로딩(LAZY)`

![스크린샷 2021-08-27 오후 7 06 55](https://user-images.githubusercontent.com/45676906/131110925-d3fcc70f-7074-4770-a6b3-57aaf432c5eb.png)

위와 같이 연관관계에서 `FetchType을 Lazy`로 주면 Team을 Proxy 객체로 가져오게 됩니다. 

<br>

![스크린샷 2021-08-27 오후 7 08 11](https://user-images.githubusercontent.com/45676906/131111075-32d6f799-e09a-45cd-87c6-c4336610dc1e.png)

위와 같이 `find`를 통해서 Member를 가져오면 JPA는 어떤 쿼리를 만들어서 실행시키는지 보겠습니다. 

![스크린샷 2021-08-27 오후 7 09 07](https://user-images.githubusercontent.com/45676906/131111159-d1ce2878-4d45-4fd5-9335-a42f0ea32b1a.png)

Member가 가져오는 쿼리를 만들어내는 것을 볼 수 있습니다. 

<br>

![스크린샷 2021-08-27 오후 7 12 41](https://user-images.githubusercontent.com/45676906/131111710-abfadd8a-e4a7-4628-98da-6459fa7eaf6d.png)

이번에는 Team 객체도 영속성 컨텍스트에 올린 후에 Team 객체의 정보를 출력해보겠습니다. 

![스크린샷 2021-08-27 오후 7 14 02](https://user-images.githubusercontent.com/45676906/131111883-6685f697-8c2d-47db-bc0c-52808212a7f8.png) 

그러면 위와 같이 `Team` 객체는 `Proxy`로 출력되는 것을 볼 수 있고, Team 객체의 name을 출력해오는 시점에 SELECT 쿼리가 출력되는 것도 볼 수 있습니다. 이렇게 엔티티를 실제 사용하는 시점에 JPA가 SQL을 호출하는 것을 `지연로딩`이라고 합니다.

<br> <br>

### `즉시 로딩(EAGER)`

![스크린샷 2021-08-27 오후 7 17 31](https://user-images.githubusercontent.com/45676906/131112303-1b61eead-2b5e-453a-adda-d773bc40dc48.png)

이번에는 `EAGER`라고 설정을 한 후에 다시 예제 코드를 실행해보겠습니다. 

<br>

![스크린샷 2021-08-27 오후 7 21 31](https://user-images.githubusercontent.com/45676906/131112766-17348357-7457-4f3b-b90c-6814eb24c8dc.png)

이번에는 Member를 조회하는 시점에 `Left JOIN`을 통해서 Team 까지 같이 가져오는 것을 볼 수 있습니다. 그리고 Team 클래스를 출력하는 부분을 보면 Proxy가 아니라 Team 객체 자체가 출력된 것도 확인할 수 있습니다.

<br> <br>

## `즉시 로딩, 지연 로딩 정리`

처음부터 연관된 엔티티를 모두 영속성 컨텍스트에 올려두는 것은 현실적이지 않고, 필요할 때마다 SQL을 실행해서 연관된 엔티티를 지연 로딩하는 것도 최적화 관점에서 보면 꼭 좋은 것만은 아닙니다. 
개발하고 있는 비즈니스에서 Member를 조회할 때 Team 까지 같이 사용하는 경우가 많다면 `즉시 로딩`을 사용하는 것이 좋고, Member를 조회할 때 Member만 필요한 경우가 많다면 `지연 로딩`을 사용하는 것이 좋습니다.

하지만 실무에서는 `지연 로딩`만을 사용하는 것이 좋습니다. 이유는 아래와 같은데요.

- 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생합니다.
- 즉시 로딩은 `JPQL에서 N + 1 문제를 일으킵니다.`
- `@ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정!`
- `@OneToMany, @ManyToOne 기본이 지연 로딩입니다.`

<br>

![스크린샷 2021-08-28 오전 1 08 51](https://user-images.githubusercontent.com/45676906/131156847-261bbd03-ccda-4755-a744-3675a09562c4.png)

JPQL에서 N+1 쿼리 문제의 예시를 들어보겠습니다. 위의 코드에서 JPPL로 Member를 조회하는 쿼리를 작성했는데 JPA에서는 어떤 쿼리를 만들어주게 될까요?

<br>

![스크린샷 2021-08-28 오전 1 11 10](https://user-images.githubusercontent.com/45676906/131157129-128068f1-1438-471a-b849-a0b0c1893389.png)

참고로 `즉시 로딩`을 사용하고 있습니다.

<br>

![스크린샷 2021-08-28 오전 1 12 28](https://user-images.githubusercontent.com/45676906/131157246-b03c217a-eec6-4e83-9ebd-742e66549a87.png)

쿼리를 보니 위와 같이 Member를 조회해오는 SELECT 쿼리문 하나, Team을 조회해오는 SELECT 쿼리문 하나를 만들게 됩니다. 영속성 컨텍스트를 사용해서 Member를 조회하면 내부적으로 최적화가 되어 있어서 한번에 JOIN을 해서 가져오지만 JPQL을 사용하면 위와 같이 쿼리가 2번 나가게 되는 문제가 발생합니다.

그런데 만약에 연관된 관계가 여러 개라면 여러 번의 쿼리가 발생하기 때문에 문제가 클 것입니다. 그래서 여기서 `EAGER`가 아니라 `LAZY`로 타입을 바꾸면 쿼리가 1번만 나가서 이러한 문제를 해결할 수 있습니다.

<br> <br>

## `지연 로딩 활용 - 실무`

- `모든 연관관계에 지연로딩을 사용해야 합니다.`
- `실무에서 즉시 로딩을 사용하면 안됩니다.`
- JPQL fetch 조인이나, 엔티티 그래프 기능을 사용하면 좋습니다.
- 즉시 로딩은 상상하지 못한 쿼리가 나갈 수도 있습니다.

<br> <br>

## `영속성 전이: CASCADE`

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 `영속성 전이` 기능을 사용하면 됩니다. 쉽게 말해서 영속성 전이를 사용하면 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장할 수 있습니다.

![스크린샷 2021-08-28 오후 8 36 43](https://user-images.githubusercontent.com/45676906/131216632-11d23f37-7fff-4143-9bcb-b243b0b2a19d.png)

<br>

![스크린샷 2021-10-04 오후 2 36 31](https://user-images.githubusercontent.com/45676906/135799310-85da1b0c-128b-45df-82a7-21d220b33c13.png)

예를들어, 위와 같이 Parent 엔티티가 Child와 `OneToMany` 관계로 존재한다고 가정하겠습니다. 

<br>

![스크린샷 2021-10-04 오후 2 37 37](https://user-images.githubusercontent.com/45676906/135799417-7339a9a9-7a94-4ab0-83c4-9b80ca428214.png)

Child 엔티티는 위와 같이 `Parent`와 `ManyToOne`의 관계를 가지고 있습니다. 

<br>

![스크린샷 2021-10-04 오후 2 38 54](https://user-images.githubusercontent.com/45676906/135799568-1113576a-09ad-4dd9-9917-fe014b1e50db.png)

위와 같이 하나의 Parent에 두명의 Child 엔티티를 만들었습니다. 이 때 엔티티를 영속성 persist 하기 위해서는 엔티티 마다 persist를 해야 하기 때문에 위와 같이 세 줄의 코드가 필요한데요. 이 때 Parent와 연관된 것은 같이 영속 상태가 되면 좋지 않을까? 해서 나온 것이 바로 `영속성 전이 CASCADE` 입니다.

<br>

![스크린샷 2021-10-04 오후 2 41 25](https://user-images.githubusercontent.com/45676906/135799739-7dbc8dd5-3601-412f-ae5c-9701b0200495.png)

Parent Entity에 위와 같이 `cacade = CascadeType.ALL` 이라는 속성을 주겠습니다. 

<br>

![스크린샷 2021-10-04 오후 2 42 29](https://user-images.githubusercontent.com/45676906/135799813-17d597ce-be55-4466-b956-3d84c37f3bf5.png)

그리고 이번에는 Parent만 persist를 한 후에 실행해보겠습니다. 

<br>

![스크린샷 2021-10-04 오후 2 43 25](https://user-images.githubusercontent.com/45676906/135799871-1e7b90d4-5699-43f3-9e78-c4543c1be3ab.png)

그러면 INSERT 쿼리도 3번 다 잘 실행된 것을 확인할 수 있습니다. 

<br>

<img width="166" alt="스크린샷 2021-10-04 오후 2 44 03" src="https://user-images.githubusercontent.com/45676906/135799920-91686564-1c5a-4a48-8a31-14a5adba7d0a.png">

그리고 데이터베이스에도 값이 잘 들어간 것을 확인할 수 있습니다. (name은 넣지 않았기 때문에 null 입니다.)

<br> <br>

## `영속성 전이: CASCADE 주의`

- 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없습니다.
- 엔티티를 영속화할 뿐 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐입니다. 

<br> <br>

## `CASCADE의 종류`

- `ALL: 모두 적용`
- `PERSIST: 영속`
- REMOVE: 삭제
- MERGE: 병합
- REFRESH: REFRESH
- DETACH: DETACH

<br>

보통 `ALL`, `PERSIST`를 많이 사용합니다. 저 또한 데이터베이스에서 유저가 쓴 게시글처럼 연관되어 있을 때 유저를 삭제할 때 CASCADE 속성을 지정해서 유저가 작성한 글 까지 한번에 다 삭제되도록 데이터베이스에서 적용한 경험이 있는 것이 생각나네요,,

<br> <br>

## `고아 객체`

JPA는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제 공하는데 이것을 `고아 객체` 제거 라고 합니다.

![스크린샷 2021-10-04 오후 2 51 03](https://user-images.githubusercontent.com/45676906/135800547-af26ad0a-0a2e-4e2f-8ec3-6132c1009f32.png)

위와 같이 `orphanRemoval = true` 옵션을 준 후에 Parent 중에 Child 하나의 연관관계를 끊어보겠습니다. 

<br>

![스크린샷 2021-10-04 오후 2 52 14](https://user-images.githubusercontent.com/45676906/135800657-7e8380e0-a5de-4f15-ae2c-1d4191902956.png)

<br>

![스크린샷 2021-10-04 오후 2 52 58](https://user-images.githubusercontent.com/45676906/135800704-6e159eca-bd53-40c0-b1fb-93000b114e4e.png)

그러면 위와 같이 remove를 통해서 Parent의 Child를 삭제하니 `DELETE` 쿼리가 실행된 것을 확인할 수 있습니다. 

<br> <br>

## `고아 객체 주의`

- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능입니다.
- `참조하는 곳이 하나일 때 사용해야 합니다.`
- `특정 엔티티가 개인 소유할 때 사용해야 합니다.`
- `@OneToOne, @OneToMany`만 사용 가능합니다.
- 참고: 개념적으로 부모를 제거하면 자식은 고아가 됩니다. 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거됩니다. `CASCADEType.REMOVE` 처럼 동작합니다.

<br> <br>

## `영속성 전이 + 고아 객체, 생명주기`

`CASCADEType.ALL + orphanRemoval = true`를 동시에 사용하면 `부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있습니다.` 즉, 위에서 본 예제처럼 Parent만 persist 해도 연관된 Child도 같이 Persist 되고, Parent를 Remove 해도 연관된 자식 Child도 Remove 되는 것을 말합니다. 