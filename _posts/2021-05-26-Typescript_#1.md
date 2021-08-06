---
title: "Typescript_#1"
categories:
  - Typescript
  - Blockchain
tags:
  - Typescript
  - Blockchain
toc: true  
toc_sticky: true 
---

# typescript란

​	:자바스크립트 처럼 생긴 것으로  컴파일시 **자바스크립트로 컴파일됨**

* **사용이유**:	
   * 자바스크립트를 보완해줌
   * 언어가 예측이 가능하고 읽기 쉼고  Js 사용을 더 편리하게 함

* **설정**:

  ​	repo만들어서 vscode에 yarn init(난 npm init -y로 함)

​		**<tsconfig.json>**	

1. compillerOptions :

   ​	(1)"module": "commonjs" 

   ​			-> node.js를 평범하게 사용하고 다양한 걸 import/export할 수 있게 만들기

   ​	(2)"target": "ES2015"

   ​			-> js의 버전 결정

   ​	(3)"sourceMap":true

   ​			-> sourceMap 처리 유무

   

2. include: 어떤 파일을 포함할지

   -> 여기선 index.ts 생성하고 추가함

   

3. exclude: 어떤 파일을 제외할지

   ->node_modules(디폴트로 해놓는 게 좋음)

   

   **index.ts**

   ​		=>**typescript 작성하는 곳**

   

   이때 **npx tsc로 ts파일을 컴파일해서 js 생성**

   ​		->**Node.js는 Typescript를 이해하지 못하므로** 일반적인 js코드로 컴파일하는 작업이 필요 (js파일로 바꾸기)

   ​	ex) index.ts -> index.js & index.js.map가 생성

   

   **package.json**

   - "scripts"부분에 start는 npm run을 실행시 실행할 것을 넣기(여기선 node index.js을 실행)

   - prestart는 먼저 실행할것(tsc로 파일을 js로 만들어야하기 때문에 )

   

   ![image](https://user-images.githubusercontent.com/79195793/119694599-52dd5a00-be88-11eb-9f60-8ca1ac8948eb.png)

   # 블록 체인 만들기

   * typescript -> 변수와 데이터의 종류 설정해야함

   ​	-->코드를 읽을 떄 무슨일이 일어날지 쉽게 알 수 있음

   

   * index.ts에서 name 만들떄 **export{}**해서 name파일이 모듈이 된다는 것을 알리도록(버그발생시)

  ![image](https://user-images.githubusercontent.com/79195793/119694638-5cff5880-be88-11eb-8d74-222e5cb3721f.png)
   
   
   
   
   
   ★**파라미터 옆에 ?를 붙이면 선택적인 파라미터로 변함**( 파라미터를 적지 않아도 괜찮음) -> undefine으로 나옴(결과는 ? 안붙여도 똑같음)

