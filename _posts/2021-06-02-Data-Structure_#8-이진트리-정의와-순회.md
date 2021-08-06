---
title: "Data-Structure_#8 트리(이진, 힙)"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
toc: true
toc_sticky: true
---

# 트리(Tree)

: 부모- 자식 관계를 계층적으로 표현한 자료구조

**[용어]**

* 루트(root): 최고 조상
* 에지(edge): 연결선, 링크(link)
* 부모 - 자식- 형제
* 리프(leaf): 자식이 없는 노드
* 레벨(level): 루트에서 0으로 시작하며, 한 세대씩 1 증가
* 경로(path): 출발노드 + 경유노드 + 도착노드
* 높이(height): 자식노드까지의 가장 큰 깊이
* 깊이(depth): 루트에서 해당 노드까지의 에지갯수
* 트리높이(tree height): 루트의 높이
* 분지수(degree): 자신의 자식 수로, 트리의 분지수는 가장 큰 분지수로 정의!
* 부트리(subtree): 어떤 노드와 그 노드의 자손노드들로 구성된 부분 트리

# 이진트리

:자식이 두개 이하인 트리

**[표현법]**

1. **리스트(다 차있다고 생각)**

* **레벨순으로! ** Ex))level 0 -> level 1 -> level2 ->…

* **같은 레벨일 경우 왼쪽꺼부터** 

 ![image](https://user-images.githubusercontent.com/79195793/120484324-ec0ef200-c3ed-11eb-8a75-5dcc3f1c5d19.png)

  ex) [a, b, c, None, d, e, f, None, None, h, i ,g]

 

2. **리스트 중복**

* 부모와 자식으로!(재귀적으로)
=>[루트, 왼쪽 부트리, 오른쪽부트리]
-> 부트리시작(None이 아니면)할 때 []붙이기
![image](https://user-images.githubusercontent.com/79195793/120484391-fc26d180-c3ed-11eb-98fb-737c3382fed9.png)

  ex) [a, [b, [ ] ],[d, [ ] ,[ ] ,[ c, [e, [ ], [ ] ], [ f, [ ], [ ] ]]] ]



3. **노드& 링크 직접 class로 정의**

   * 직접 트리 노드 정의(key/value+ left link+ right link + parent)

     ![image](https://user-images.githubusercontent.com/79195793/120484428-08129380-c3ee-11eb-94fe-ac22d750a555.png)

## 힙(heap)

: 힙 성질(heap property)를 만족하는 이진트리 (이진트리를 배열, 리스트로 표현)

* **Heap 성질 : 모든 부모노드의 key값은 자식노드의 key 값보다 작지 않다**

1. **모양 : level 별로 다 채워져있고 같은 레벨은 왼쪽부터 **

2. **루트노드엔 가장 큰 값이 들어감! (max_heap) <--> 루트에 가장 작은 값(min_heap)**

3. **제공연산 :**

   **A.**   **Insert**   **->O(log n)**

   **B.**   **Find_max : return A[0]   ->O(1)**

   **C.**   **Delete_max**  **-> O(log n)**

ex)H=[a, b, c, None, d, e, f]

![image](https://user-images.githubusercontent.com/79195793/120484481-152f8280-c3ee-11eb-8e02-e08bff1c1990.png)



## 연산

**1.**  **Make-heap : heapify-down**

​	: **리스트 값들이 힙 성질 만족하도록 재배열하는 함수**

​	->맨 마지막 노드부터 힙 성질(부모>자식) 만족하도록 바꾸기

​    -> 부모 < 자식이면 부모와 자식의 리스트 자리를 바꿈

```python
def make_heap(self):
    n = len(self.A)
    for k in range(n-1, -1, -1):    # A[n-1] -> A[0]
        self.heapify_down(k, n)
def heapify_down(self, k, n):
    # n = 힙 전체 노드 수(heap_sort에 쓸거임)
    # A[k]를 힙 성질 만족하는 위치로 내려가면서 재배치
    while 2*k+1 < n:
        L, R = 2*k + 1, 2*k + 2
        if self.A[L] > self.A[k]:
            m = L
        else:
            m = k
        if R < n and self.A[R] > self.A[m]:
            m = R
        # m = A[k], A[L], A[R] 중 최대값의 인덱스
        
        if m != k:   # A[k]가 최대값이 아니면 힙 성질 위배
            self.A[k], self.A[k] = self.A[m], self.A[k]
            k = m
        else:
            break
```

**2. insert 연산**

![image](https://user-images.githubusercontent.com/79195793/120484606-33957e00-c3ee-11eb-95c0-3ed09d1749b3.png)

**3. find_max & delete_max**

-> 루트 노드(제일 큼)를 삭제 후 맨 마지막 leaf노드를 루트노드에 넣은 후 heapify_down()

```python
def find-max:
    return A[0]
def delete-max:
    if len(A) == 0: return None
    key = A[0]
    A[0], A[len(A) - 1] = A[len(A) - 1], A[0]
    A.pop()
    heapify-down(0, len(A))
    return key
```
**4. heap_sort()**
-> 힙이 아니면, make_heap으로 힙만든 후, heapify down으로 오름차순 반복 적용하여 재배치하는 함수
![image](https://user-images.githubusercontent.com/79195793/120917991-19f57e80-c6ed-11eb-9428-4392de02c8a9.png)




* **시간복잡도**
  1. make_heap: O(n), O(nlog n)
  2. insert: O(log n)
  3. find-max: O(1)
  4. delete-max: O(log n)
  5. heapify-down: O(h) = O(log n)
  6. heapify - up: O(h) = O(log n)
  7. heap_sort : makeheap= O(n) & sawp+heapify_down = n-1번 => O(nlogn)
![image](https://user-images.githubusercontent.com/79195793/120484654-3f814000-c3ee-11eb-975a-004727414078.png)

![image](https://user-images.githubusercontent.com/79195793/120918281-a48aad80-c6ee-11eb-8e73-f7fa6e84d801.png)

