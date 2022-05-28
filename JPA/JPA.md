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

