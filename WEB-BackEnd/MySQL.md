# 관계형 데이터베이스 MySQL

<font size="5"> **DATABASE SERVER >> DATABASE >> TABLE** </font> </br>
MySQL은 DATABASE SERVER 속에 DATABASE를 생성하고 그 속에 TABLE들을 저장해 CRUD를 구현하는 구조이다.
(DB는 스키마라고 부르기도 한다.)

**C(CREATE)**, **R(READ)**, **U(UPDATE)**, **D(DELETE)**

데이터를 생성하고 읽고, 수정과 삭제를 하는 데이터베이스의 본질이 CRUD이다.

GIT BASH를 이용해 MySQL서버에 접속했다.  
`mysql -uroot -p` -uroot (최상위 권한)과 -p(password)를 통해 서버에 접속한다.

---

**아래 내용은 구글링을 통해 확인하자 외우지말고 !(기본적인 내용이기 때문에 요약)**

`SHOW DATABASES;` 를 통해 현재 DATABASE를 확인한 후에 생성하도록하자.

DB 생성 : `CREATE DATABSE name`  
DB 삭제 : `DROP DATABASE name `  
DB 선언 : `USE name`
