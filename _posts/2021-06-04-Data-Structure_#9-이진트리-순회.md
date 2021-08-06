---
title: "Data Structure_#9 이진트리 순회"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
toc: true
toc_sticky: true
---

## 이진트리 순회(traversal)

: 노드를 빠짐없이 방문하는 법

1. preorder :부모, 왼쪽, 오른쪽 (MLR) : F B A D C E G I H
2. inorder : 왼쪽, 부모, 오른쪽 (LMR) : A B C D E F G H I
3. postorder : 왼쪽, 오른쪽, 부모 (LRM) : A C E D B H I G F

![image](https://user-images.githubusercontent.com/79195793/120802693-b3058780-c57d-11eb-962f-4b2edcc8328e.png)


* **노드 클래스의 메쏘드로 정의하기**

```python
class Node:
    def __init__(self,key=None, parent-None, left= None, right = None):
        self.key = key
        self.parent = parent
        self.left = left
        self.right = right
    def __str__(self):
        return str(self.key)
    
    def preorder(self):
        if self != None:
            print(self.key)
            if self.left: self.left.preorder()
            if self.right: self.right.preorder()
 
```

* **Tree/BST 클래스의 메쏘드로 정의**

```python
def preorder(self,v):
    if v != None:
        print(self.key)
        print(v.left)
        print(v.right)
```



## 이진탐색트리(Binary Search Tree): BST

: 각 노드의 왼쪽 subtree의 key 값은 노드의 key 값보다 작거나 같아야하고,

​					오른쪽 subtree의 key 값은 노드의 key 값보다 커야한다.(**L<M<R**)

```python
class BST:
    def __init__(self):
        self.root = None
        self.size = 0
    def __len__(self):
        return self.size
    def __iter__(self):
        return self.root.__iter__() #generator
    def __str__(self):
        return " - ".join(str(k) for k in self)
    
```



## 연산

1. 탐색연산

   ```python
   def search(self, key):
       p = self.find_loc(key)
       if p and p.key == key:
           return p
       else: return None
   def find_loc(self, key): # key있으면 그 노드 리턴, 없으면 그 노드를 넣을 자리의 부모 리턴
       if self.size == 0: return None
       p = None
       v = self.root
       while v:
           if v.key == key: return v
           elif v.key<key:
               p = v
               v = v.right
           else:
               p = v
               v = v.left
           return p
   ```

   

2. 삽입연산

   ```python
   def insert(self,key):
       p = self.find_loc(key)
       if p == None or p.key != key:
           v = Node(key)
           if p == None:
               self.root = v
           else:
               v.parent = p
               if p.key >= key: #check if left/right
                   p.left = v
               else:
                   p.right = v
            self.size = self.size + 1
           return v
       else:
           print("key is already in the tree!")
           return p #중복 key를 허용하지 않으면 None 리턴
   ```

   

3. 삭제연산

   (1) deleteByMerging

![image](https://user-images.githubusercontent.com/79195793/120802749-c153a380-c57d-11eb-8ceb-57a08b75822b.png)
   -> **삭제하고,  왼쪽 부트리를 삭제된 자리에 끼워넣기**



​	(2) deleteByCopying

![image](https://user-images.githubusercontent.com/79195793/120802790-cc0e3880-c57d-11eb-8c1f-b15c70846602.png)
![image](https://user-images.githubusercontent.com/79195793/120802811-d29cb000-c57d-11eb-92e7-3b547309c64a.png)
​		
-> **삭제할 노드에 왼쪽 부트리의 가장 큰 노드를 복사하고 큰 노드는 삭제**

