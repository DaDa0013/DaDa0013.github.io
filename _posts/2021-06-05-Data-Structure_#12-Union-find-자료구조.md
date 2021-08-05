---
title: "Data Structure_#12 Union-find 자료구조"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
---

## Union-find 자료구조

![image](https://user-images.githubusercontent.com/79195793/120885921-67f37f00-c626-11eb-8074-324bc72c1161.png)
## 연산

1. make-set(x): {x}  				=> O(n)

2. find(x): x가 속한 집합의 대표값 리턴  =>O(log n)

3. union(x, y): x, y의 합집합  =>O(logn )

## 구현 방법

​	1. 원형 양방향 연결리스트  => 0(n)이 될 수 있기 때문에 비효율적

![image](https://user-images.githubusercontent.com/79195793/120885924-6cb83300-c626-11eb-97ab-32bbd7f02190.png)
​	2. 트리  => O(h)

​		: **부모링크만 가지고 있고**, **root는 자기 자신을 가리키게**

![image](https://user-images.githubusercontent.com/79195793/120885933-76419b00-c626-11eb-9852-ddfd17e62b48.png)

​      Height가 작아지도록 height가 큰쪽에서 작은쪽으로 union

![image](https://user-images.githubusercontent.com/79195793/120885947-82c5f380-c626-11eb-9ea5-9ad7c01a5573.png)
![image](https://user-images.githubusercontent.com/79195793/120885956-89ed0180-c626-11eb-9776-88b3a5e57a5d.png)
