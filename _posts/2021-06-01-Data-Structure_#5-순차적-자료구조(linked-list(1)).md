---
title: "Data Structure_#5 순차적 자료구조(linked list(1))"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
toc: true
toc_sticky: true
---

# 연결리스트(linked list)

**: 노드(node)가 링크(link)에 의해 기차처럼 연결된 순차(sequential) 자료구조**

* **Node = key(데이터 값) + link** 

  => 메모리상에 흩어져 있음 (node와 link로 연결)

* **장점** : **삽입(insert)이 O(1)시간으로 매우 빠름**

  ​		-> 넣을 곳 앞 뒤의 **link만 바꿔주면** 되기 때문( 단, 앞 뒤를 알고 있어야 함)

* **단점 : 읽기는 head node부터 link 따라가야함**
![image](https://user-images.githubusercontent.com/79195793/120322039-f31aff00-c31e-11eb-9825-1024c3924e3f.png)

  ## 한방향 연결리스트(singly Linked List)

  **: 노드들이 한쪽방향으로만(next 링크 따라) 연결된 리스트**

  ```pseudocode
  # 노드클래스
  class Node: 
  	def __init__(self, key=None, value=None):
  		self.key = key   # 노드를 다른 노드와 구분하는 key값
  		self.value = value #필요한 경우 추가 데이터 value
  		self.next = None	# 다음 노드로의 링크(초기값은 None)
  		
  	def __str__(self): 	#print(node)인 경우 출력할 문자열
  		return str(self.key) 	#str((self.key, self, value))
  
  # 한방향연결리스트 클래스
  class SinglyLinkedList:
  	def __init__(self):
  		self.head = None	# 연결리스트의 가장 앞의 노드 (head)
  		self.size = 0		# 리스트의 노드 개수
  		
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

  

* **지원 연산**

  L = SinglyLinkedList()

  ​	**[삽입]**

  1. L.pushFront(key) : key 값을 갖는 새 노드를 L의 가장 앞에 삽입

     ​			**-> L.head가 변경되야함**

  2.  L.pushBack(key) : key 값을 갖는 새 노드를 L의 가장 뒤에 삽입

     **[삭제]**

  3.  L.popFront(): L의 첫 노드(head 노드) 삭제 & key 리턴

  4.  L.popBack() : L의 마지막 노드 삭제 & key 리턴

  5.  L.remove(v) : L에서 노드 v를 제거

     **[찾기]**

  6.  L.search(key): L에서 key 값을 갖는 노드를 찾아 리턴



* **pushFront**

```pseudocode
def pushFront(self,key):
	new_node = Node(key, value)  # 새 노드 생성
	new_node.next = self.head
	self.head = new_node
	self.size += 1
```



* **pushBack**

  ```pseudocode
  def pushBack(self, key, value = None):
  	new_node = Node(key, value) #새 노드 생성
  	if self.size == 0: #empty list
  		self.head = new_node 
  	else: 	# 빈 리스트가 아니면
  		tail = self.head
  		while tail.next != None: #tail 찾기
  			tail = tail.next
  		tail.next = new_node
  	self.size += 1
  ```

  

* **popFront**

  ```pseudocode
  def popFront(self):
  	if self.size == 0: #or len(self) == 0
  		return None
  	else: 
  		x = self.head
  		key = x.key
  		self.head = x.next
  		self.size = self.size - 1
  		del x
  		return key
  
  ```

  

* **popBack**

  ```pseudocode
  def popBack(self):
  	if self.size == 0: #노드가 하나도 없는 경우
  		return None
  	else:
  		prev, tail = None, self.head
  		while tail.next != None:
  			prev = tail
  			tail = tail.next
  		if prev == None: #노드가 한개만 있을 때
  			self.head == None
  		else:
  			prev.next = None
  		key = tail. key
  		del tail
  		self.size -= 1
  		return key
  ```

* **search(key)**

  **[while]**

  ```pseudocode
  def search(self, key):
  	v = self.head
  	while v != None:
  		if v.key == key:
  			return v
  		v = v.next
  	return v
  ```

  **[for]**

  ```pseudocode
  def search(self,key):
  	for v in self:
  		if v.key == key:
  			return v
  	return None
  ```

  **[generator]**  --> 찾기 연산시 for 사용하면 됨

  ```pseudocode
  def __iterator__(self):
  	v = self.head
  	while v != None:
  		yield v      #for문에 대입해 None만나도록(stopiterator 로 for루프 중단)     
  		v = v.next
  # for x in L:
  #	print(x)
  ```

  

* **remove(v)**

  ```pseudocode
  def remove(self, v):
  	if self.size == 0: # 빈리스트
  		return None
  	else:
  		prev, x = None, self.head
  		while x.next != None:
  			prev = x
  			x = x.next
  		if prev == None: # 노드가 하나
  			self.popFront(x)
  		else:
  			prev.next = x.next
  			del x
  ```

  
