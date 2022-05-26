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



 