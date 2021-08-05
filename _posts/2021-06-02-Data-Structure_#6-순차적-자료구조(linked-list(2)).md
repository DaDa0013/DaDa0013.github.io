---
title: "Data Structure_#6 순차적 자료구조(linked list(2))"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
---

## 양방향 연결리스트(DoublyLinkedList)

: 마지막 노드와 첫 노드가 서로 연결된 **원형(circular)** 리스트

​			-> head 노드는 항상 dummy 노드(dummy노드는 "marker"기능) 

*  **장점: 한방향 연결리스트의 단점( search할 때 처음부터 따라가야 함) 보완**

* **단점: 관리해야 하는 링크가 두 배가 됨**

```python
class Node:
    def __init__(self, key = None):
        self.key = key
        self.next = self.prev = self # 자기로 향하는 링크
    def __str__(self):  		#print(node)인 경우 출력할 문자열
        return str(self.key)
    
class DoublyLinkedList:
    def __init__(self):
        self.head = Node() 	#빈 리스트는 dummy 노드만으로 ㅍ현
        self.size = 0
	def __iter__(self): # generator 정의
    	v = self.head	
		while v != None: 
			yield v
			v = v.next
            
	def __str__(self): # 연결 리스트의 값을 print
		return " -> ".join(str(v) for v in self)
        
	def __len__(self):
		return self.size
        
```

## 연산

★**Spilce 연산**

-> def splice(self, a, b, x)

: 노드 a부터 b까지 떼어내(cut) 노드 x 뒤에 붙여(paste) 넣는 연산( cut-and-paste )

* **조건**
  1. a==b 이거나 a다음 b
  2. head와 x는 a와 b사이에 포함되면 안됨

![image](https://user-images.githubusercontent.com/79195793/120356169-00e07c80-c33f-11eb-8fa7-473a92d66e7b.png)

```python
def splice(self, a, b, x):
    if a == None or b == None or x == None:
        return
    ap = a.prev
    bn = b.next
    
    #cut[a..b]
    ap.next = bn
    bn.prev = ap
    
    #insert[a..b] after x
    xn = x.next
    xn.prev = b
    b.next = xn
    a.prev = x
    x.next = a
```

1. **이동 연산**

   * moveAfter(self, a, x) : 노드 a를 x다음으로 이동 **-> splice(a,a,x)**
   * moveBefore(a, x): a를 x 전으로 이동 **->  splice(a, a, x.prev)**

2. **삽입 연산**

   * insertAfter(x, key): 노드 x뒤에 새로운 노드 key(head와 tail연결됨)를 삽입

     ​			**->moveAfter(Node(key),x)**

   * insertBefore(x, key): 노드 x앞에 새로운 노드 key를 삽입

     ​           **-> moveBefore(Node(key),x)**

   * pushFront(key): head 노드 다음에 새로운 노드 key 삽입 

     ​			**->insertAfter(self.head, key)**

   * pushBack(key): head 노드 앞에 새로운 노드 key 삽입

     ​			**->insertBefore(self.head, key)**

3. **탐색 연산**

   * search(key)

     ```python
     def search(self, key):
         v = self.head		#dummy node
         while v.next != self.head:
             if v.key == key:
                 return v
            	v = v.next
         return None
     ```

     

4. **삭제 연산**

   * remove(x)

     ```python
     def remove(self, x):
         if x == None or x == self.head
         	return
         x.prev.next. x.next.prev = x.next, x.prev
     ```

   * popFront() : head 다음에 있는 노드 삭제 &  리턴

     ```python
     def popFront(self):
         if self.isEmpty(): return None
         key = head.next.key
         self.remove(head.next)
         return key
     ```

   * popBack() : head 이전 노드 삭제 & 리턴

     ```python
     def popBack(self):
         if self.isEmpty(): return None
         s = self.head.prev
         key = s.prev.key
         self.remove(s.prev)
         return key
     ```

5. **기타 연산**
   * join(another_list): self 뒤에 another_list 연결
   * split(x) : self를 노드 x 이전과 x이후의 노드들로 구성된 두 개의 리스트로 분할

## 시간복잡도

moveAfter/Before(a,x)

insertAfter/Before(x,key)       **-> splice이용 하므로 0(1)**

pushFront/Back(key)

 

remove(x)                **-> search로 탐색하기 때문에 0(n)**

popFront/Back()            **-> remove 사용하므로 0(n)**

![image](https://user-images.githubusercontent.com/79195793/120356235-0f2e9880-c33f-11eb-84f6-9ff03034d327.png)
![image](https://user-images.githubusercontent.com/79195793/120892903-57a1cb00-c64b-11eb-9627-239e54e766ac.png)

