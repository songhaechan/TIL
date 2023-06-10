DBCP(DataBase Connection Pool)
=
중고차 판매시스템을 구현하던중에 아래와 같은 코드를 작성했었다.

```java
private static class UserDaoCreation{
        private static final UserDaoImpl INSTANCE = new UserDaoImpl();
    }
    public static UserDaoImpl getInstance(){
        return UserDaoCreation.INSTANCE;
    }
```
Dao객체를 싱글톤으로 생성했고 당시의 의도는 DBConnection은 매번 연결하는 작업은 무겁기때문에 싱글톤으로 구현했다.

하지만 어차피 Connection을 열고 마지막엔 닫아주기때문에 Dao를 싱글톤으로 구현한다고해서 Connection을 유지하지않는다.

## DB의 Connection을 유지하려면?

DBCP는 WAS(Web Application Server)에서 사용하는 기술이다.

Connection 객체의 개수를 WAS에 설정하면 그 수만큼 미리 Connection을 만들고 Connection Pool에 저장해놓는다.

웹서비스의 경우 (굳이 웹이 아니더라도) 사용자가 많아지면 매번 Connection을 생성하는 양이 매우 많아질 것이고 서버의 부하가 걸릴것이다.

이를 대비해 미리 Connection을 생성해놓고있다가 요청이 들어오면 Connection을 반환해주고 다 사용하면 다시 반환받아 Pool에 저장해놓는다.

이렇게 Connection의 생성비용을 낮추는 기술이 DBCP이다.