@Modifying
=

[Ref]https://www.baeldung.com/spring-data-jpa-modifying-annotation

## Spring Data JPA가 제공하는 데이터베이스에 질의하는 3가지 방법

1. Query Methods
   메서드의 이름으로 JPQL을 생성
2. @Query Annotation
   어노테이션으로 상세한 JPQL을 직접 작성
3. Custom repository implementation


## @Modifying

@Query 를 이용해 JPQL을 작성할 수 있다.(아래처럼)

```java
@Query("select u from User u where u.email like '%@gmail.com'")
List<User> findUsersWithGmailAddress();
```

**하지만 !** 업데이트쿼리(insert, update, delete)는 꼭 Modifying 어노테이션이 붙어야한다.

## findByUserName()

부제와 같이 쿼리 메서드로 생성한 커스텀 메서드를 두 번 호출한다면 어떨까?

물론 같은 트랜잭션 내부에서 영속성컨텍스트도 초기화된 상태에서 말이다.

findByUserName이 처음 호출되면 캐싱돼 다음 호출땐 쿼리가 안나가고 총 1번의 쿼리가 나갈 것 같다.

하지만 실제로는 총 2번의 쿼리가 나간다.

영속성컨텍스트는 key는 PK value는 Entity를 가진다.

findByUserName은 이름을 기준으로 찾기때문에 key값을 알 수 없다.







