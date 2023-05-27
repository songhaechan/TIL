Mybatis vs Hibernate
=
우선은 Persistence Layer에 대해 알아보자.

영속계층이라 불리는 이 계층은 DB에 직접 접근하는 계층이다. 지금 프로젝트에서 사용하는 Dao객체가 Persistence Layer에 속한다.

현재 프로젝트에서 Dao객체를 이용해 순수한 JDBC를 이용해 코드를 작성하고있지만 코드의 양이 상당히 많고 중복도 많이 발생한다.

그래서 탄생한 것이 Persistence Framework이다.

## Persistence Framework 에는 SQL Mapper와 ORM 두 가지가 있다.

ORM은 그 뜻대로 자바의 객체와 데이터베이스 테이블을 매핑하는데 그 중점은 관계에 있다. SQL Mapper는 중점은 관계가 아닌 필드와 테이블사이의 단순한 맵핑에 있다.

SQL Mapper에는 오늘 알아보고자하는 MyBatis, JDBC templetes(spring)이 있다.

또 ORM은 SQL을 자동 생성하지만 SQL Mapper는 SQL을 직접 명시해서 데이터에 접근해야한다.

ORM에는 JPA, Hibernate가 있다. 엄밀히말하면 JPA는 인터페이스이고 Hibernate는 JPA의 구현체이다. 하지만 둘 다 ORM이라는 것은 동일하다.

## MyBatis

Plain JDBC를 프로젝트에 사용중이지만 여러가지 설정이 필요하다. 

Dao객체를 이제 하나 만들었지만 그 설정은 매 객체마다 해주어야한다.(질문:Templete패턴을 사용)

그 설정을 나열해보자면... 드라이버정보, 커넥션, PreparedStatement, excute SQL, 리소스 순차로 닫기 등등 많은 일을 개발자가 직접해주어야한다. try-catch로 예외처리는 덤이다.

하지만 MyBatis는 이 설정들을 해주기때문에 개발자는 SQL쿼리만 작성하면된다.

## Hibernate

당연한 말이지만 Hibernate도 Plain한 JDBC의 설정들을 하지않아도 된다. 추가적으로 SQL쿼리조차 작성하지않아도 된다.

SQL문을 개발자가 작성하지않는다고해서 Hibernate가 마법을 부리는것은 아니다. 내부에서는 JDBC API가 작동하고있다.