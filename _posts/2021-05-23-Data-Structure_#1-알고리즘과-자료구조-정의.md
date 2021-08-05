---
title: "Data Structure_#1 알고리즘과 자료구조 정의"
categories: 
  - Data Structure
tags:
  - Data Structure
  - Python
---

## 자료구조 & 알고리즘 개념

## 자료구조

-> 저장공간 + 읽기, 쓰기, 삽입, 삭제, 탐색 (연산)

ex) 

 * 배열/리스트 -> 각 원소의 인덱스로 접근(읽기, 쓰기, 삽입, 삭제)
 * 변수 -> 변수의 이름으로 접근(읽기, 쓰기 연산)

## 알고리즘

: 자료구조에 담긴 데이터를 가공(연산)하여 **원하는 값 출력**

ex) GCD(최대공약수)

1. gcd_sub(a,b): a-b (큰 수에서 작은 수를 반복적으로 빼가면서 gcd 계산)

```python
   def gcd_sub(a, b):
   		while (a*b!=0):
   			if(a>b):
   				a=a-b
   			else:
   				b=b-a
   		return a+b 
```

2. gcd_mod(a, b): a-b (큰 수를 작은 수로 나눈 나머지를 이용하여 gcd 계산)

   ```python
   def gcd_mod(a, b):
   		while (a*b!=0):
   			if(a>b):
   				a=a%b
   			else:
   				b=b%a
   		return a+b
   ```

   

3. gcd_rec(a, b): gcd_sub의 재귀함수 버전

   ```
   def gcd_rec(a, b):
   		if (a*b!=0):
   			return gcd_rec(b,a%b)
   		else:
   			return a
   ```

   

# 가상컴퓨터 & 가상언어 & 가상코드

**자료구조와 알고리즘의 성능을 측정하기 쉽도록 ** **가상언어로 작성된 가상코드를 가상컴퓨터에서 시뮬레이션하여 HW/SW에 독립적인 계산환경을 만듦**

## 가상컴퓨터(virtual Machine)

: Turing machine(컴퓨터 모델)에 기초한 **RAM(Random Access Machine)**모델 사용

 * **RAM**: CPU(계산 수행 유닛) + Memory + **기본연산**(단위시간에 수행되는 연산 모음)

 * **기본연산**

   - 배정, 대입, 복사 연산

   - 산술 연산: +, -, *, /  

     ​		*(%, 버림연산, 올림연산, 반올림은  RAM모델에서 기본연산에 속하지 않음)*

   - 비교 연산: >, >=, <=, ==, !=

   - 논리 연산: AND, OR, NOT

   - 비트 연산: bit-AND, OR, NOT

## 가상 언어(Pseudo/virtual languages)

: 가상 컴퓨터에 전달하는 언어로, 기본 연산들을 표현하고 여러 명령어들을 제공하는 역할

* **기본 명령어**
  * 기본 연산
  * 비교문: if, if else, elif, else
  * 반복문: for, while
  * 함수 정의 & 호출
  * return 문

## 가상 코드(Pseudo Code)

: 가상 언어로 작성된 코드

ex)

```python
algorithm Array Max(A,n):
	Input: n개의 정수를 갖는 배열 A   
    output: A 중에서 최대값 리턴
    currentMax=A[0]           

    for i=1 to n-1 do
		if currentMax<A[i]:
			currentMax=A[i]
	return currentMax
```





