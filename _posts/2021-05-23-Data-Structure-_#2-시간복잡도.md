---
title: "Data Structure & Algorithm _#2 시간복잡도"
categories:
  - Data Structure
  - Python
  - Algorithm
tags:
  - Data Structure
  - Python
  - Algorithm
toc: true  
toc_sticky: true 
---
## 수행시간

* 최악의 경우의 입력에 대한 기본 연산 횟수


## 시간 복잡도(time complexity)

* 측정방법:

  * 모든 입력에 대해 기본연산횟수 더한 후 평균 ->불가능에 가까움..

  * **가장 안 좋은 입력(worst case input)에 대한 기본 연산 횟수를 측정**

    ![image](https://user-images.githubusercontent.com/79195793/119261151-0f2ceb00-bc11-11eb-8cad-b4f6b2058955.png)

## Big-O

-> **최고차항**으로만 간단하게 **증가율 표기**

 **-방법**

​	**1)**  **함수의 최고차항만 남긴다**

​	**2)**  **최고차항 계수(상수) 생략**

​	**3)**  **Big-O(최고차항) 형태로**

Ex) Algorithm1: **T1(n)=2n-1**   Algortihm2: **T2(n)=4n+1**   Algorithm3: **T3(n)=3/2n^2-3/2n+1**

  ![image](https://user-images.githubusercontent.com/79195793/119261249-85315200-bc11-11eb-80ff-7ea0736839f3.png)


 * **Big-O표시**

   -> T1(n)=O(n)     T2(n)=O(n)    T3(n)=O(n^2)

Q. 

​	1)  Algorithm2가 Algorithm1보다 2배 느리다 : O (계수로 확인 가능)

​	2)  Algorithm3는   n<5/3면 Algorithm2보다 빠르다  : O

​       						     모든 n에 대해 Algorithm 1보다 느리다 : O

​	3) Algorithm3는 n>5/3면 항상 Algorithm2보다 느리다    : O



**-집합으로 이해해보기**

![image](https://user-images.githubusercontent.com/79195793/119261331-e1947180-bc11-11eb-8a3d-cc4d4f0489d2.png)



**-연습문제**

![image](https://user-images.githubusercontent.com/79195793/119261347-f07b2400-bc11-11eb-930a-4336cdca9786.png)

​		

​					A. **T(n): 3n/2**

​						**Big-O: O(n)**

![image](https://user-images.githubusercontent.com/79195793/119261352-f53fd800-bc11-11eb-9d41-a2ca72657193.png)

![image](https://user-images.githubusercontent.com/79195793/119261380-16082d80-bc12-11eb-9e03-96604d9f421d.png)

​					**A. T(n): 1 + n^2(4log n)**

​						 **Big-O: O(n^2log n)**

 

![image](https://user-images.githubusercontent.com/79195793/119261376-10124c80-bc12-11eb-8b7f-7b6e153affb1.png)

​					**A. T(n): 2 + 4n^0.5**

​						**Big-O: O(n^(0.5)) = O(sqrt(n))**



![image](https://user-images.githubusercontent.com/79195793/119261372-0b4d9880-bc12-11eb-8c61-d78db809cd33.png)

​					**A. T(n): 2 + 4log n**

​						**Big-O: O(log n)**



<img src="https://user-images.githubusercontent.com/79195793/119261410-451e9f00-bc12-11eb-8c79-d941bf4cf727.png" style="zoom:67%;" />

![image](https://user-images.githubusercontent.com/79195793/119261449-739c7a00-bc12-11eb-9d9a-d8310230ebd2.png)

![image](https://user-images.githubusercontent.com/79195793/119261462-7eefa580-bc12-11eb-90db-2e4a41dfdd78.png)

![image](https://user-images.githubusercontent.com/79195793/119261479-92027580-bc12-11eb-9fd0-b4ee33717451.png)
