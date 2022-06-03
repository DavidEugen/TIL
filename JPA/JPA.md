# JPA




 ## JPA 기본 사용 방법

1. EntityManagerFactory 생성 -> EntityManager 생성 ->  EntityTransaction 생성 -> 쿼리 사용

- EntityManagerFactory 는 애플리케이션 전체에서 하나 만들어서 공유
- EntityManager 는 쓰레드간 공유하면 안된다. 한번쓰고 버리기
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행 해야 한다. 



2. JPA 가 관리하는 객체 -> @Entity 을 통해 알려줌
   @Table 관 @Column을 통해서 객체와 테이블/컬럼 이 다르더라도 매핑가능

  

3. insert, update, find 사용법

  - java 컬렉션 다루듯이 작업 할 수 있다.



##  엔티티 매핑 - @Entity

@Entity
기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
interface, final class, enum, inner class 사용 불가
저장할 필드에 final 사용 불가

@Entity(name="")사용 할 수 있다. 기본은 클래스 이름 그대로,
가급적 기본값 사용



## 데이터 베이스 자동 생성

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

설정하면 어플리케이션 실행 시점에 자동으로 생성 (dialect에 따라 DB에 맞는 DDL 생성) => 되도록 개발 장비에서만.. 운영에서 사용시 다듬에서 사용 필요
create = drop create
create-drop = create, 종료때 drop
update = 변경 부분만 반영, alter colum 으로.. colum 삭제 없음

<- 운영사용 금지 

validate = 엔티티와 테이블 매핑 여부 확인

none = 아무것도 안함. 위의 값(create, create-drop, update, validate)과 다른모든 값들..



## 영속과 비영속

비영속 persist() 이전의 상태, 객체만 생성된 상태.
persist()를 하더라도 아직 DB 까지는 가지 않는다.

```shell
-----Before------
-----After------
Hibernate: 
    /* insert hello.jpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```



## Entity 생명 주기


1차 캐시 사용

find()를 2번 하더라도 1번만 조회 한다.
2번째는 EntityManager에 있던 1차 캐시에서 조회.

```
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
```

### 영속 엔티티의 동일성 보장

```java
Member findMember = em.find(Member.class, 101L);
Member findMember2 = em.find(Member.class, 101L);

System.out.println("result = " + (findMember == findMember2));
        
```

1차 캐시로 반복 가능한 읽기(Repeatable Read) 등급으 ㅣ트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

```shell
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
result = true
```



## 쓰기지연(transactional write-behind)

바로 쿼리 날리는 것이 아니라 트랜잭션을 지원하는 쓰기 지연을 통해 트랜잭션을 지원

버퍼처럼 사용 할 수 있다.


```java
Member memberA = new Member(111L, "A");
Member memberB = new Member(112L, "B");

em.persist(memberA);
em.persist(memberB);

System.out.println("=================");
```


```shell
=================
Hibernate: 
    /* insert hello.jpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert hello.jpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```



persistence.xml 에서 buffer 사이즈를 옵션으로 조정 할 수도 있다.

```xml
<property name="hibernate.jdbc.batch_size" value="10"/>
```



## 변경 감지(Dirty Checking)

업데이트 시
EntityManager.persist(member); 나
EntityManager.update(member); 
있어야 할 것 같은데, 이런거 없이 update가 바로 되는 이유는


변경 감지 때문

0. 처음 영속 컨텍스트에 올라올때 최초 상태를 1차 캐시에 스냅샷으로 저장
1. commit()을 하게 되면 내부적으로 flush()호출
2. 이때 엔티티와 스냅샷을 비교하여 
3. 다르면 update SQL 을 생성하여 쓰기 지연 저장소에 저장했다가
   나중에 일괄적으로 처리 

```java
 Member member = em.find(Member.class, 112L);
 member.setName("changeB");

//em.persist(member); // 요론것들이 있어야 할 것 같은데...
//em.update(member); //

System.out.println("============");
```



## Flush

영속성 컨텍스트의 변경내용을  데이터베이스에 반영, 동기화 (맞추는) 작업.

- 변경 감지 하고
- 수정된 엔티티의 쓰기 지연 SQL 저장소에 등록하고
- 쓰기 지연 SQL 저장소의 쿼리를 DB로 전송

EntityManager.flush() 로 직접 호출 할 수 도 있다.

보통은 커밋에서 (자동으로 설정되어 있으면)



캐쉬 지우나? => 지우지 않는다.





EntityManager.setFlushMode(FlushModeType.AUTO/FlushModeType.COMMIT) 으로 설정가능

FlushModeType.AUTO => 커밋이나 쿼리 실행 시 플러시 (default)

FlushModeType.COMMIT => 커밋할때만 플러시



영속성 컨텍스트를 비우는게 아니라, 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화

트랜잭션이라는 작업 단위가 중요

커밋 직전에만 동기화 하면 됨





## 양방향 연관관계

단방향 연관관계가 2번 있는것, 이미 단방향 맵핑만으로도 이미 연관관계 매핑 완료

양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐



#### 주인 정하기

비지니스 로직상으로 연관관계의 주인을 결정하지 말고,  FK 있는 객체(FK 등록 변경 가능한 객체) 으로 정한다.

단방향 매핑으로 설계 ( FK 있는 곳이 주인으로 만들고 ) , 탐색 할 일이 있으면 역방향을 맺어준다.

탐색을 위한 역방향 연관관계는 mappedBy로 설정

mappedBy 있는 곳은 읽기 전용

mappedBy = "주인필드"



양방향 매핑시 주의사항 - 무한루프

ex) toString(), lombok, JSON 생성 라이브러리 



양방향 연관관계를 넣는 이유는 개발의 편리성을 위해서이다.

연관관계의 주인이 아닌것은 조회만 가능하도록 한다.



## 상속관계 맵핑

RDB에서는 상속관계 없음. 다만 수퍼타입-서브타입 모델링 기법이 상속과 유사

@Inheritance(strategy=InheritanceType.XXX) 

- JOINED: 조인 전략
- SINGLE_TABLE: 단일 테이블 전략
- TABLE_PER_CLASS: 구현 클래스마다 테이블 전략

@DiscriminatorColumn(name=“DTYPE”) 구분자 이름을 무엇으로 할 것인가
@DiscriminatorValue(“XXX”)


JOINED
장점 : 테이블 정규화 , FK 참조 무결성 제약조건 활용 가능, 저장공간 효율화
단점 : 조회시 조인 많이 사용 성능 저하, 조회 쿼리 복잡, 데이터 저장시 INSERT 2번씩 호출

SINGLE_TABLE
장점 : 조인이 필요 없으므로 일반적으로 조회 성능이 빠름, 조회 쿼리 단숨
단점 : 자식 엔티티가 매핑한 컬럼은 모두 null 허용 필요. 테이블이커질 수 있음 => 상황에 따라서 조회 성능이 오히려 느려질 수 있음.

TABLE_PER_CLASS
되도록 쓰지 말것
장점 : 서브타입 명확하게 구분해서 처리 할 때 효과적, Not Null 제약조건 사용 가능
단점 : UNION 걸림, 자식 테이블 통합 쿼리 어려움.



### @MappedSuperclass

테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할.
주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통 으로 적용하는 정보를 모을 때 사용.
엔티티X, 테이블과 매핑X

직접 생성해서 사용할 일이 없으므로 추상 클래스 권장

@Entity 클래스는 엔티티나 @MappedSuperclass로 지 정한 클래스만 상속 가능



## 지연로딩



보통 지연 로딩만 쓰도록 한다.

즉시 로딩시 관련 테이블들 다 불러 올 수 있다



즉시 로딩시 N+1 나오는 이유

find()의 경우 PK 를 찍어서 가져오지만

createQuery() 인 JPQL 은 sql로 번역을 한다. 

만약 Member 조회 하면 Member 전체를 가지고 온다. 그 안에서 다 즉시 로딩이 걸린 Entity가 있으면 그 값을 채워 넣어야 하기때문에 Entity를 조회해 온다.



### FetchType default

@ManyToOne 의 경우 : EAGER

=> FetchType.LAXY로 선언해 줘야 한다.

@OneToMany의 경우 : LAZY



이론 : 자주 사용되는 짝은 즉시 로딩을

가끔 사용되는 짝은 지연 로딩을

**실무 : 모두 지연로딩으로 써라!!**

JPQL fetch 조인이나, 엔티티 그래프 기능을 활용하라!!



## CASCADE

부모 Entity를 저장하면서 자속 Entity를 모두 저장하고 싶을때 사용한다.

연관관계 매핑하는 것과는 상관 없다.  



CascadeType.ALL

하나의 Parent가 Child 를 관리 할때 , 즉 Parent와 Child 의 라이프사이클이 동일할때 

(다른 Entity와 Child가 관계 있으면 쓰면 안된다.)

CascadeType.ALL 쓰면 된다.



CascadeType.Persist는 단일 관계 아닐때 .저장만 한꺼번에 해 줬으면 좋을때



CascadeType.Remove는 삭제 조심해야 할때



orpahnRemoval = true 는

참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제 가능

참조하는 곳이 하나일때 사용해야 한다 (특정 엔티티가 개인 소유할 때 사용가능)

@OneToOne, @OneToMany에서만 사용 가능

개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다.

CascadeType.REMOVE처럼 동작한다. 



CascadeType.ALL + orpahnRemoval = true 로 

persist()로 영속화, remove()로 제거

두 옵션을 모두 활성하 하면 부모 엔티티를 통해서 자식의 생명 주기를 관리 할 수 있다.

DDD 의 Aggregate Root 개념을 구현할때 유용



## 값 타입

### JPA의 데이터 타입 분류

**엔티티 타입**

@Entity로 정의하는 객체

데이터가 변해도 식별자로 지속해서 추적 가능

**값 타입**

int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체

식별자가 없고 값만 있으므로 변경시 추적 불가

- 기본 값 타입
  - java primitive type ( int, double )
  - 래퍼 클래스(Integer, Long)
  - String
- 임베디드 타입(embedded type, 복합 값 타입)
- 컬렉션 값 타입 (Collection value type)



!! 기본타입은항상값을복사함



### Embedded Type

주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함

정의는 @Embeddable
사용하는 필드에서는 @Embedded

**기본 생성자 필수로 만들어 줘야 함**

```sehll
No default (no-argument) constructor for class: jpabook.jpashop.domain.Member (class must be instantiated by Interceptor)

```

> 자바 리플렉션은 런타임에 해당 클래스의 정보를 알아내는 역할을 하는데 구체적인 클래스 타입을 알지 못해도 해당 클래스의 정보를 알 수 있다. 그런데 리플렉션은 기본생성자를 통해서 클래스 정보를 가져오므로 파라미터가 포함된 생성자 만으로는 정보를 가져오지 못한다.

[기본 생성자가 갖는 의미](https://velog.io/@jakeseo_me/%EA%B0%84%EB%8B%A8%EC%A0%95%EB%A6%AC-%EC%9E%90%EB%B0%94%EC%97%90%EC%84%9C-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EC%9D%98-%EC%9D%98%EB%AF%B8-feat.-Java-Reflection-Jackson-JPA)

https://hyeonic.tistory.com/191

https://velog.io/@yyy96/JPA-%EA%B8%B0%EB%B3%B8%EC%83%9D%EC%84%B1%EC%9E%90

https://wbluke.tistory.com/6