논 클러스터 인덱스 vs 클러스터 인덱스
=
인덱스에대해 DB책에서 자세히 다루어서인지 두 개념의 차이를 명확하게 공부할 수 있었다.

[MySQL Official Document](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)

## 우선 클러스터 인덱스의 특징을 알아보자.

>Typically, the clustered index is synonymous with the primary key.

    보편적으로, 클러스터인덱스는 기본키와 동의어이다.

>When you define a PRIMARY KEY on a table, InnoDB uses it as the clustered index. A primary key should be defined for each table. If there is no logical unique and non-null column or set of columns to use a the primary key, add an auto-increment column. Auto-increment column values are unique and are added automatically as new rows are inserted.

    PK를 지정하면 InnoDB는 PK를 클러스터인덱스로 지정한다. PK는 테이블마다 정의되어있어야만한다. 논리적고유성 그리고 널이아닌값이 없거나 기본키를 지정하지않았다면 auto-increment컬럼을 추가한다.(클러스터인덱스)


- 물리적으로 행을 재배열한다.

- 클러스터를 지정하지않으면 PK가 클러스터인덱스가된다.

- 데이터 수정 삽입시 항상 정렬상태를 유지한다.

    그만큼 시간이 오래걸리게된다. 조회와 트레이드오프관계에있다.

### 어떤 경우에 사용하면 좋을까?

클러스터 인덱스의 가장 큰 특징은 "이미 정렬된 상태"이다.(정렬된 상태에 도달하기까지의 시간을 제외)

항상 정렬된 상태를 유지하기때문에 탐색에 매우 빠르다.

즉 이 장점을 이용해야한다.

DB에서는 집약함수(count/sum/avg/max), GROUP BY를 이용하면 정렬을 발생하게한다.

하지만 인덱스가 존재하면 정렬을 건너뛰기때문에 고속화가 가능하다.

즉 위와같은 조회를 많이 사용한다면 클러스터 인덱스를 사용하면 좋다.

하지만 단점도 있다.

정렬이라는 장점이 단점이된다. 정렬을 한다는것 자체가 부담이 가는 작업이고 조회엔 빠르지만 데이터변경에는 성능이 떨어진다.

## 이제 논클러스터 인덱스를 알아보자.

- 테이블당 64개의 논클러스터 인덱스를 생성할 수 있다.

- 입력 수정 삭제는 클러스터보다 빠르다. 하지만 조회는 느리다.

- 논클러스터 인덱스 페이지를 따로 생성한다. 기존의 테이블을 정렬하는 것이 아닌 새로 생성한 페이지에 정렬된 정보를 저장한다.(키값과 데이터의 위치를 저장)

- 인덱스 페이지를 따로 저장하기때문에 용량측면에서 클러스터보다 좋지않다.

기존의 테이블이 정렬된 상태가아니기때문에 인덱스를 이용한 탐색을 하기위해 따로 생성한 인덱스페이지를 경유해 탐색하기때문에 클러스터인덱스방식보단 느리다.

