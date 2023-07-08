Spring Data JPA
=

### JPARepository<>를 상속받으면 구현체는 spring이 직접 인젝션해준다.

### 공통적인 CRUD는 기본적으로 제공된다.

(상상 가능한 모든 공통적인 메서드!)

### 메서드명으로 메서드시그니처만 작성해주면 알아서 구현해준다.(물론 다 JPA를 이용해서)

자세한 쿼리메서드 작성기준은 아래를 참조하자. 다나온다.

https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation

### 메서드 작성 시 주의점

- 너무 길어지면 가독성이 떨어진다. 안되겠다싶으면 @Query로 직접 jpql 작성하자

- like 와 containing 은 나가는 쿼리가 다르다. 검색엔 주로 containing 으로하자

---

### 엔티티에 직접 NameQuery가 있지만 사용하지 말자.

더 좋은게 있다.

---

### 더 좋은건 @Query 로 메서드 시그니처에 직접 jpql작성하는 방식이다.

```java
@Query("select m from Member m where m.username= :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int
    age);
```

@Param 으로 변수를 바인딩해서 사용하면 된다.

```java
@Query("select m.username from Member m")
    List<String> findUsernameList();
```
이렇게 단순히 값 하나를 조회할 수도있다.

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
          "from Member m join m.team t")
  List<MemberDto> findMemberDto();
```

DTO를 직접 조회하는건 좀 번거롭다. new연산자를 사용해서 생성자와 형식을 맞춰야한다.

(QueryDSL을 사용하면 간편해진다.)

```java
@Query("select m from Member m where m.username in :names")
    List<Member> findByNames(@Param("names") List<String> names);
```

in 절로 사용하기위해 컬렉션을 바인딩해서 사용할 수 있다.

물론 Query Creation을 이용해도 할 수 있다.

---

## 기가막힌 페이징을 지원한다.

Page 와 Slice를 사용해서 매우 단순하게 페이징이 해결된다.

우선 Page는 흔히 게시판에서 번호를 눌러 페이징하는 경우에 사용하고 Slice는 모바일에서 스크롤링하는 방식에서 사용한다.

굳이 둘로 나누는 이유는 성능상의 문제인데, Page는 select쿼리이후에 count를 위한 쿼리가 따로 날아간다. 

하지만 Slice는 count쿼리가 날아가지않는다.

데이터가 많이 없다면 차이는 미미하겠지만 데이터가 많아질 수록 count쿼리에대한 부담이 생길 수 있다.

```java
Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안 함
```

Pageable을 파라미터로 받게되어있다.

제네릭에 있는 데이터이외에 프론트에서 사용해야할 페이징처리 정보가 다 JSON으로 넘어간다.

또 한 엔티티는 직접 내리는것은 금지다. map메서드로 매핑해서 보내자.

---

### 수정 삭제 쿼리엔 @Modifying 붙여야한다.

안그러면 내부에서 jpa가 .getxxx()로 예외터진다.

@Modifying은 jpa 내부적으로 excute한다.

```java
@Query("select m from Member m left join fetch m.team")
  List<Member> findMemberFetchJoin();
```

N+1문제를 해결하는 첫 번째 방법은 fetch join이다.

기본을 LAZY로 깔아놓음면 한 번 쿼리에 단건쿼리가 계속 날아간다.

한 번 쿼리에 10개의 행이 있다면 연관관계 엮인 객체 찾겠다고 그 객체 수 만큼 10개씩 추가적으로 나간다.

```java
@Override
@EntityGraph(attributePaths = {"team"}) List<Member> findAll();
```

두 번째 방법은 엔티티 그래프 참조인데, 위처럼 참조하는 엔티티 이름을 넘기자.

하지만!!!! 간단할때만 사용하고 복잡할땐 직접 쿼리쓰자.

---

### 페이징 자세히

```java
@GetMapping("/members")
  public Page<MemberDto> list(Pageable pageable) {
      Page<Member> page = memberRepository.findAll(pageable);
      Page<MemberDto> pageDto = page.map(MemberDto::new);
      return pageDto;
}
```

파라미터로 Pageable을 받는데 여기엔 파라미터가 아주 많이 들어간다.

정렬 조건, page 번호, size 등등...

---

쿼리 방식 선택 권장 순서
1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화 한다. 대부분의 성능 이슈가 해결된다.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사
용한다.

---

일대다 즉 리스트를 땡겨올때 !!

jpql에 select distinct를 넣자 !!

이건 sql의 distinct (중복제거) + 객체 중복제거!!!

근데 페이징이 안된다.

DB row 와 객체매핑간에 차이가 있다...

객체는 중복제거가 가능하지만 db는 여전히 중복이있기때문에 페이징할때 자를 데이터를 선택할 수가 없다.

하이버네이트가 메모리에 다 퍼올려서 해주기는 하지만 로컬메모리에 올리기때문에 사용하지말자.

방법이 있긴하다. 있긴해도 일대다가 한번만 나오는 경우에 사용하자.

 
