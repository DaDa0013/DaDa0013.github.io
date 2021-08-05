---
title: "Typescript_#2"
categories:
  - Typescript
  - Blockchain
tags:
  - Typescript
  - Blockchain
---

#  Types

* 타입 지정하기

  --> **파라미터: 타입**

  ![image](https://user-images.githubusercontent.com/79195793/119695338-08a8a880-be89-11eb-97df-e253b99df250.png)

  =>많은 인수들을 넣어야할 때 타입을 지정해줘서 쉽게 알 수 있게

* 함수 리턴 지정

  => 함수: 리턴 유형

  ex)

  ![image](https://user-images.githubusercontent.com/79195793/119695286-fc245000-be88-11eb-8379-d18b7a5b0437.png)

### TSC watch

1.  tsc watch 패키지 다운

   ​	npm add tsc-watch --dev

   ​		**=> watch 모드에서 컴파일을 시작**

   ​	->src에서 무언가 바꿀때마다 dist가 바뀌도록 하고 저장하면 node 작업이 진행됨

2. src파일과 dist파일 만들기

   ->src에서 모든 것을 컴파일 할 것이기 때문에, src로 index.ts 이동

3. package.json

   start 바꿔주기

   ![image](https://user-images.githubusercontent.com/79195793/119695241-f3337e80-be88-11eb-9d19-7725e4131104.png)

4. tsconfig.json

   include부분 바꿔주기 & outDir 설정

   ![image](https://user-images.githubusercontent.com/79195793/119695196-e878e980-be88-11eb-9257-09ce3073b25e.png)
5. gitignore

   -> dist 추가

   **=> 모든 typescript는 src로 들어가고 컴파일 된 것들은 dist로 들어감**



**->src에서 무언가 바꿀때마다 dist가 바뀌도록 하고 저장하면 node 작업이 진행됨**



# Interfaces on Typescript

* function에 object 전달하는 법

  **-> 인터페이스 생성하기(js에서 작동 안함)**

  ​![image](https://user-images.githubusercontent.com/79195793/119695135-dac36400-be88-11eb-83c9-734eb0c52b3f.png)
  **그 다음 설정한 인터페이스를 타입으로 하는 매개변수를 넣음! **(js에서 작동 안함)

  **그리고 interface 속성을 '.'으로 접근**

 ![image](https://user-images.githubusercontent.com/79195793/119695071-c97a5780-be88-11eb-8acc-19884a9941a2.png)
