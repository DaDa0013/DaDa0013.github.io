---
title: "Data Structure_#10 Balanced BST(균형이진탐색트리)"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
---

# 균형이진탐색트리(Balanced BST)

: 연산속도를 빠리게 하도록 높이를 O(log n)로 유지하는 BST

* 최악의 경우 트리의 높이 h만큼 계산해야 함  -> **연산(search, insert, delete)이 0(h)시간만큼 소요함**

=> 즉,  **높이가 작을수록** 시간이 적게 걸림 !

![image](https://user-images.githubusercontent.com/79195793/120919177-2da3e380-c6f3-11eb-9184-2219eae9c9b7.png)


# rotate

: 트리의 높이를 최소로 유지하기 위한 연산

**회전 후에도 BST의 값의 순서가 그대로 유지(inorder나 preorder, postorder같이..)**

=>  log(n+1)<=h<=n-1이므로 **h를 강제로 log(n+1)**로 유지

* 방법: right rotation, left rotation

![image](https://user-images.githubusercontent.com/79195793/120805672-0200ec00-c581-11eb-934d-95581d16eca3.png)
```python
    def rotateLeft(self, x):
        if x == None: return
        v = x.right
        if v == None: return
        b = v.left
        v.parent = x.parent
        if x.parent:
            if x.parent.right == x:
                x.parent.right = v
            else:
                x.parent.left = v
        v.left = x
        x.parent = v
        x.right = b
        if b:
            b.parent = x
        if x == self.root:
            self.root = v
        self.fixHeight(x)

    def rotateRight(self, x):  
        if x == None: return
        v = x.left
        if v == None: return
        b = v.right
        v.parent = x.parent
        if x.parent != None:
            if x.parent.left == x:
                x.parent.left = v
            else:
                x.parent.right = v
        v.right = x
        x.parent = v
        x.left = b
        if b:
            b.parent = x
        if x == self.root:
            self.root = v
        self.fixHeight(v)
```

# AVL 트리

: 모든 노드에 대해서 노드의 **왼쪽 트리와 오른쪽 트리의 높이차가 1이하인 BST**
![image](https://user-images.githubusercontent.com/79195793/120919300-c89cbd80-c6f3-11eb-9062-c8e053a806f1.png)
    ->BalanceFactor 표시

=> Nh=높이가 h인 AVL 트리중에서 최소 노드 개수

**Nh=N(h-1) +N(h-2)+1**

​	Ex) N0=1, N1=2, N2=4, N3=7 

​		=> 위 점화식을 통해 h=0(log n)임을 알 수 있음

![image](https://user-images.githubusercontent.com/79195793/120805703-0b8a5400-c581-11eb-944c-527656ed5d53.png)
## 연산

1. **Insert(self,key):**

   ​	=> rebalance: **rotation** **최대 2번**

   
   1. V=super(AVL(클래스 이름),self).insert(key) #부모함수받기     -> **O(h)=O(log n)**

   2. Find x, y, z(z는 처음으로 AVL조건 깨진 노드)					 -> **O(h)=O(log n)**

   3. W=rebalance(x,y,z) # AVL 조건을 만족 못할 때 rebalance     -> **O(1)시간**

   4. If w.parent==None:   															->**O(1)**

      ​			Self.root=w



​		=> **총 O(log n)시간 걸림**

​			> insert 된곳부터 거슬러 올라가 rebalance할 곳 찾음

** right-right : leftR/ left- left: RightR/ right- left: RightR & LeftR/ left-right: LeftR/RightR

![image](https://user-images.githubusercontent.com/79195793/120805724-1218cb80-c581-11eb-9817-85e25edfc445.png)
![image](https://user-images.githubusercontent.com/79195793/120805749-18a74300-c581-11eb-8ba1-e355318dba2c.png)
2.  **Delete**

   ​	=> **rebalance : 0(log n) rotations**
          -> 트리의 높이만큼 반복

   * 높이가 무거운 쪽에서 가벼운 쪽으로 rebalance	

   * worst case: 0(log n)(높이만큼) ( 매 level에서 rotation 해야 할 수 있음)

   * **rebalance 할 2가지 경우로 분리 (2회인경우 x를 z까지)(무거운 쪽부터 x y z)**
   *  **균형을 맞추기 위해선, v가 속한 부트리의 높이를 증가시켜야하고, 이를 위해 y,x를 z에서 v가 속하지 않은(높이가 더 큰)부트리에 속하는 노드(z-y-x)로 정의해야함★ *(insert 와 다름)* **

     => 코드 참조

![image](https://user-images.githubusercontent.com/79195793/120805775-1f35ba80-c581-11eb-909e-dcd2a39b3ade.png)
## 시간복잡도

![image](https://user-images.githubusercontent.com/79195793/120805793-24930500-c581-11eb-942d-2d6a960ab522.png)
