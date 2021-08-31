# 엔티티 매핑
- 객체와 테이블 매핑 @Entity, @Table
- 기본키 매핑 @Id
- 필드와 컬럼 매핑 @Column
- 연관관계 매핑 @ManyToOne, @JoinColumn

매핑정보는 xml 이나 어노테이션 중에 선택해서 기술하면 되는데 책에서는 어노테이션만 사용.
어노테이션이 더 쉽고 직관적이다. 

## @Entity
@Entity 가 붙은 클래스는 JPA 가 관리
### 속성
- name : 기본값은 클래스 이름. 다른 패키지에 동일한 이름의 엔티티가 있으면 충돌하지 않게 해아함
### 주의사항
- 기본 생성자는 필수다 (public, protected)
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없다
- 저장할 필드에 final 을 사용하면 안된다

## @Table
- 엔티티와 매핑할 테이블을 지정
- 생략하면 매핑한 엔티티 이름을 테이블 이름으로 지정
### 속성
- name : 테이블 이름. 기본값은 엔티티 이름
- catalog : catalog 매핑
- schema : schema 매핑
- uniqueConstraints : ddl 을 만들 때만 사용

## 다양한 매핑 사용
```java
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)    // enum 을 사용하려면 @Enumerated 매핑
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)   // 날짜 타입은 @Temporal 을 사용해서 매핑
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;     // @Lob 을 사용하면 CLOB, BLOB 타입 매핑
    ...
}
```
## 데이터베이스 스키마 자동 생성
### feature/h2
```log
Hibernate: 
    drop table MEMBER if exists
Hibernate: 
    create table MEMBER (
        ID varchar(255) not null,
        age integer,
        createdDate timestamp,
        description clob,
        lastModifiedDate timestamp,
        roleType varchar(255),
        NAME varchar(10) not null,
        primary key (ID)
    )
```

### feature/mysql
```log
Hibernate: 
    drop table if exists MEMBER
Hibernate: 
    create table MEMBER (
        ID varchar(255) not null,
        age integer,
        createdDate datetime,   # datetime
        description longtext,   # longtext
        lastModifiedDate datetime,
        roleType varchar(255),
        NAME varchar(10) not null,
        primary key (ID)
    )
```
### hibernate.hbm2ddl.auto
- create
- create-drop
- update
- validate
- none: *none 은 유효하지 않은 옵션 값이다. 

### 이름 매핑 전략
ImproveNamingStrategy: 테이블 명이나 컬럼 명이 생략되면 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑

https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#naming

## DDL 생성 기능
### @Column
- nullable: 자동 생성되는 ddl 에 notnull 제약조건을 추가할 수 있음
- length: 문자의 크기 지정할 수 있음
### @Table
- uniqueConstaint: 유니크 제약조건 추가

## 기본 키 매핑
### 직접 할당
- @Id 어노테이션 사용해서 기본키를 애플리케이션에서 할당
### 자동 생성
- IDENTITY: 기본 키 생성을 데이터베이스에 위임
- SEQUENCE: 데이터베이스 시퀀스를 사용해서 기본 키를 할당
- TABLE: 키 생성 테이블을 사용

SEQUENCE 나 IDENTITY 전략은 사용하는 데이터베이스에 의존함
- oracle 은 시퀀스를 제공하지만 mysql 은 제공하지 않음
- mysql 은 AUTO_INCREMENT 기능 제공

### hibernate.id.new_generator_mappings 
책에는 기본값 false, 현재는 true
https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#configurations-mapping

하이버네이트 5부터 기본값이 true 임
```
By the way, starting with Hibernate 5 (which is in WildFly 10.0.0.Final), hibernate.id.new_generator_mappings defaults to true.  I like the idea of being able to specify default persistence unit properties to be used, although I think that is a separate concern than the one raised by your post.  Currently, you have to update all of your applications persistence.xml. 
```
https://developer.jboss.org/thread/267613

### 기본키 직접 할당 전략
em.persist() 로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방식
- 자바 기본형 (Primitive)
- 자바 wrapper 형
- String
- java.util.Date
- java.sql.Date (https://docs.oracle.com/javase/7/docs/api/java/sql/Date.html)
- java.math.BigDecimal
- java.math.BigInteger
- java.util.UUID
    An interesting feature introduced in Hibernate 5 is the UUIDGenerator. To use this, all we need to do is declare an id of type UUID with @GeneratedValue annotation:
    ```
    @Entity
    public class Course {

        @Id
        @GeneratedValue
        private UUID courseId;

        // ...
    }
    ```
    https://www.baeldung.com/hibernate-identifiers

### IDENTITY 전략
- 기본키 생성을 데이터베이스에 위임. 
- MySQL, PostgreSQL, SQL Server, DB2 에서 주로 사용

#### mysql auto_increment
```sql
CREATE TABEL BOARD (
    ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    DATA VARCHAR(255)
);
INSERT INTO BOARD(DATE) VALUES('A');
INSERT INTO BOARD(DATE) VALUES('B');
```
데이터베이스에 값을 저장할 때 ID 컬럼을 비워두면 데이터베이스가 순서대로 값을 채워준다

#### 최적화
Statement.getGeneratedKeys() 를 사용하면 데이터를 저장하면서 동시에 생성된 기본 키 값도 얻어올 수 있다. 
하이버네이트는 이 메소드를 사용해서 데이터베이스와 한 번만 통신한다

#### 쓰기 지연
IDENTITY 식별자 생성 전략은 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있으므로 em.persist() 를 호출하는 즉시 INSERT SQL 이 데이터베이스에 저장된다. 따라서 IDENTITY 전략에서는 트랜잭션을 지원하는 쓰기 지연이 동작하지 않는다. 

### SEQUENCE 전략
데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트다. 

이 전략은 시퀀스를 지원하는 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용할 수 있다. 


코드는 IDENTITY 전략과 같지만 내부 동작 방식은 다르다. 

SEQUENCE 전략은 em.persist() 를 호출할 때 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회한다. 
그리고 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다. 
이후 트랜잭션을 커밋해서 플러시가 일어나면 엔티티를 데이터베이스에 저장한다. 
반대로 이전에 설명했던 IDENTITY 전략은 먼저 엔티티를 데이터베이스에 저장한 후에 식별자를 조회해서 엔티티의 식별자에 할당한다. 

#### @SequenceGenerator 
- name
- sequenceName
- initialValue
- allocationSize
    - SequenceGenerator.allocationSize 의 기본값이 50인것에 주의.
    - 데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값을 반드시 1로 설정해야 한다.
- catalog, schema


#### mysql 에서 sequence 사용
```log
Hibernate: 
    drop table if exists MEMBER
Hibernate: 
    drop table if exists hibernate_sequence
Hibernate: 
    create table MEMBER (
        ID bigint not null,
        age integer,
        createdDate datetime,
        description longtext,
        lastModifiedDate datetime,
        roleType varchar(255),
        NAME varchar(10) not null,
        primary key (ID)
    )
Hibernate: 
    alter table MEMBER 
        add constraint NAME_AGE_UNIQUE  unique (NAME, age)
Hibernate: 
    create table hibernate_sequence (
         next_val bigint 
    )
Hibernate: 
    insert into hibernate_sequence values ( 1 )
...

Hibernate: 
    select
        next_val as id_val 
    from
        hibernate_sequence for update
            
Hibernate: 
    update
        hibernate_sequence 
    set
        next_val= ? 
    where
        next_val=?

Hibernate: 
    /* insert jpabook.start.Member
        */ insert 
        into
            MEMBER
            (age, createdDate, description, lastModifiedDate, roleType, NAME, ID) 
        values
            (?, ?, ?, ?, ?, ?, ?)
...
```

### TABLE 전략
키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다. 
시퀀스 대신에 테이블을 사용한다는 것만 제외하면 SEQUENCE 전략과 내부 동작방식이 같다. 

```sql
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name )
)
```

#### @TableGenerator
- name
- table
- pkColumnName
- valueColumnName
- initialValue
- allocationSize
- catalog, schema
- uniqueConstraints(DDL)


### AUTO
데이터베이스 방언에 따라 전략 중 하나를 자동으로 선택한다. 
예를 들어 오라클은 SEQUENCE, MySQL 은 IDENTITY 를 사용한다. 


## 필드와 컬럼 매핑: 레퍼런스
필요할 때 코드, 주석, 문서를 보자.

## 정리
생략

## 실전 예제 1. 요구사항 분석과 기본 매핑


## 질문
- 왜 기본생성자는 필수이고, public, protected 만 가능한지
    - 상속을 쓰나?
- 저장할 필드에 왜 final 을 사용하면 안되는지
    - kotlin 은 val 써도되던데..
- mysql auto_increment 어떻게 동작하는지
- kotlin `@Id @GeneratedValue val id: Long = 0L` 으로 지정하면 query 어떻게 만들어지는지