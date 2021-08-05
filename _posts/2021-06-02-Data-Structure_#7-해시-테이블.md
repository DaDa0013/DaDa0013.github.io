---
title: "Data Structure_#7 해시 테이블(Hash table)"
categories:
  - Data Structure	
  - Python
tags:
  - Data Structure
  - Python
---

# 해시 테이블(Hash Table)

: 일종의 **사전(dictionary)**으로, **매우 빠른 삽입, 삭제, 탐색 연산 제공**

-> item: key - value 을 

ex)

​	D = {} # 사전

​	D[key] = value



* **해시 테이블 성능에 중요한 것?**
  1. Table(list 저장 서랍장): list
  2. hash function( 해시 함수 )
  3. collision resolution method( 충돌 회피 )

## 해시함수 (hash function)

: **정보(key)를 저장할 슬롯의 번호를 계산하는 함수**

**-> key를 mapping해서 table의 index로 전환하는 역할**



### 종류

1. division hash function: 소수로 나눈 나머지를 인덱스로 -> 충돌 많음
2. perfect hash function: key 값들이 slot에 일대일로 -> 이상적 해시함수**(비현실적)**
3. **c-universal hash fuction** :두 개의 hash fuction이 같은 값, 즉 **충돌이 일어날 확률이 c/m(c=상수값)** -> 서로 다른 임의의 두 key값 x, y에 대해 **prob(f(x) == f(y)) =c/size(H)** 가 성립

![image](https://user-images.githubusercontent.com/79195793/120478740-22e20980-c3e8-11eb-9765-1e9eeac7d17e.png)

그 외 교재 p.53-54 참고



### 좋은 hash function 기준

1. less collision(최소 충돌)
2. fast compution (빨리 계산)



## 충돌 해결방법(collision resolution methods)

### **1. open addressing**

​	**(1) linear probing**

​			: 충돌 일어나면 **선형적으로 밑으로 내려가면서 빈칸 찾음**

​			- cluster(충돌): key값들이 연속된 슬롯에 저장되어 있는 것 (cluster는 되도록 적어야 함)

​	**(2) quadratic probing** : **클러스터 사이즈 줄이기**

​			: 충돌 시 **k+n^2식으로 내려 감!**

​			Ex) k -> k+1^2 -> k+2^2 -> k+3^3

​			->**cluster 사이즈가 늦게 증가함! But table이 복잡해짐**

​			->**hash 함수 2개 마련해야 함& 2번 계산**

​	**(3) double hashing**: **클러스터 사이즈 줄이기**

​			: 충돌 시 **f(key)+ n*g(key)** 식으로!

​			Ex) f(key) -> f(key)+g(key) -> f(key)+2g(key) -> f(ley)+ 3g(key)

​			->**hash 함수 2개 마련해야함& 2번 계산**



​	**[연산]**

​	**(1) Set& Search**

​		->Find-slot(key): key 값 있으면 slot 번호 리턴, 없으면 key값이 삽입될 번호 리턴
    -> set(key,value): key를 갖는 item이 테이블에 있으면 해당 아이템의 value를 수정, 없으면 (key, value)아이템삽입
```python
def find-slot(key):
    i = f(key)
    start = i
    while (H[i]== occupied) and (H[i].key != key):
    	i = (i+1) % m
    	if i == start: return "Full"
   	return i
```

```python
def set(self, key, value=None):
    i = self.find_slot(key)
    v = self.H[i].search(key)
    if v == None:
        self.H[i].pushFront(key,value)
    else:
        v.value = value
```

**(2) remove**

​      **옮겨야 하는 경우 3가지** -> k<i<=j, i<j<k, j<k<i
    ->**시간: cluster 길이에 비례해서**

```pseudocode
def remove(key):
    i = find-slot(key)
    if H(i) is unoccupied:
        return None
    j = i
    while True:
        H[i]=None
        while True:
            j = (j+1)%m
            if H[j] is unoccupied: return key
            k = f(H[j].key)
            if (k<i<=j): break
        H[i] = H[j]
        i = j
```

![image](https://user-images.githubusercontent.com/79195793/120478827-3a20f700-c3e8-11eb-881a-2f5325037275.png)

### 2. chaining

: **충돌 시 한방향 연결리스트로!** 

​		**[연산]**

* **Set(key, value=None):**

1. key값이 H에 있으면 value을 update

2. key 값이 H에 없으면 (key, value)을 insert

   -> slot마다 연결리스트 연결

   ->한방향 연결리스트 사용 

* **search**

* **remove**

  

​		**[연산 수행시간]**

* **set: O(1)**

* **search& remove: O(충돌 key의 평균 개수= 연결리스트 길이)**

 

-> **연결리스트 길이가 상수, 즉 O(충돌 key의 평균 개수)= O(1)이 되도록 c-universal쓰기!**

-> **0(1) 내에 set, search, remove 수행가능** (p.64)



![image](https://user-images.githubusercontent.com/79195793/120478876-47d67c80-c3e8-11eb-8172-304b9101e07e.png)

**But if cluster size가 평균적으로 m>=2n(n, m은 load factor), 즉 최소 50% 이상 빈 슬롯일때**

​	**=>cluster 평균사이즈 = O(1)**

## dict 자료구조

* **dict** **자료구조가 python 내부에서 어떻게 구현?**

  ​	**1. hash function**

  ​     	->**index를 size로 나눈 나머지 값**

  ​	**2. collision resolution**

  ​     	->**open addressing 방법 사용**

​                   **-빈 곳 찾는 코드(size가 2의 n승 형태일 때 슬롯 모두 돎)**

​													 ![image](https://user-images.githubusercontent.com/79195793/120478924-545ad500-c3e8-11eb-8b3a-fc3e3ecd7f58.png)

​    					 ->**perturb로 랜덤하게 (더 적은 수로) 돌 수 있도록**

​		**3. resize**

​    				 **: table이 다 찼을 때 다시 크기 정의**

​     					-> **2/3만큼 찼을 때 2배로 resize함! (amortized time analyois)**

​     					->**set, remove, search:** **평균 O(1) (hash table의 강점)**
