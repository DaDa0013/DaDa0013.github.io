---
title: "Data Structure_#11 균형이진탐색트리(Red-Black, 2-3-4트리)"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
---

# Red-Black Tree

![image](https://user-images.githubusercontent.com/79195793/120883548-c154b180-c618-11eb-9ba3-9f69dd00a97d.png)

​			Bh(13)=2 / bh(17)=2/ bh(25)=1 

[조건]

* 노드는 red/black

* root=black

* leaf(None)=black -> NIL: None 노드로 특별히 독립된 노드로 취급(leaf 노드)

* red node—child node: black (red 노드는 무조건 black 자식노드 가져야함)

* **각 노드에서 leaf nodes까지 경로에서의 black node수가 항상 같아야 함**

  ​				->NIL을 뺀 나머지 노드는 내부 노드

  [증명]

  (귀납)

![image](https://user-images.githubusercontent.com/79195793/120923004-36061980-c707-11eb-8547-3e4bfa37f322.png)
![image](https://user-images.githubusercontent.com/79195793/120923016-3ef6eb00-c707-11eb-8f23-a39d1be627ba.png)


## 연산

1. **삽입 연산**

![image](https://user-images.githubusercontent.com/79195793/120923388-3a333680-c709-11eb-9dec-88c9bba3f19d.png)
![image](https://user-images.githubusercontent.com/79195793/120923395-3dc6bd80-c709-11eb-9f1d-c37eb41830dc.png)

![image](https://user-images.githubusercontent.com/79195793/120923563-348a2080-c70a-11eb-99da-29a54b066fc5.png)


# 2,3,4 트리

:**이진트리 아님**
**모든 리프노드는 마지막 레벨에 위치해야 하는 탐색트리**

1. 자식 노드 개수 =2, 3, 4 (2-노드,3-노드,4-노드로 구분)

2. 모든 leaf 노드가 같은 level에 존재

3. 리프노드들은 원형 이중 연결리스트에 의해 좌-우로 연결

4. 높이: O(logn)
   ![image](https://user-images.githubusercontent.com/79195793/120883591-f52fd700-c618-11eb-8a82-e27a124b4c2f.png)

## 연산

1. **insert(key)**  => 0(log n)     

   ->1 split: O(1)

   ->가운데 key 값을 부모로 올리기
![image](https://user-images.githubusercontent.com/79195793/120923761-2b4d8380-c70b-11eb-86ea-fe8f5006ee19.png)
![image](https://user-images.githubusercontent.com/79195793/120923805-6780e400-c70b-11eb-9e1e-5e828699ba4c.png)


![image](https://user-images.githubusercontent.com/79195793/120883608-0d9ff180-c619-11eb-8fc9-7fa052dad75d.png)
![image](https://user-images.githubusercontent.com/79195793/120924227-81232b00-c70d-11eb-8f7f-af4f11c43174.png)

2. **Delete**:  지울 것이 leaf에 있다 가정 ->0(log n)시간
    
    =>루트에서부터 leaf까지에서 2노드면 3노드로 바꾸기(형제 중 가장 왼쪽이나 오른쪽을 부모로 올림) -> 0(log n)시간
     
     -> leaf에 지울 값이 없으면 leaf 랑 바꾸기 & leaf 지우기
        
     ->좌우 형제노드가 다 2노드일땐: 왼쪽이나 오른쪽으로 부모노드랑 합침(4노드로)



    1. key가 내부노드인 경우
   
        => successor(key)또는 predecessor(key) 찾아 swap -> 리프노드에 있는 값 지움
        
    2. key가 리프에 있는 경우
    
        (1) 3-노드, 4-노드: 단순삭제
        
        (2) 2-노드인경우
        
          -> 지우면 underflow발생
             
          -> 루트에서 리프로 내려가면서 2-노드만나면 무조건 3-노드, 4노드로 바꿈(단, 루트가 2-노드인경우는 제외)
             
          a. 그 과정에서 트리 높이 1감소
                  
          b. 삭제할 key값이 저장된 리프노드는 3-노드/ 4-노드가 되어 단순 삭제 가능
                  
   3. 2-노드를 3-노드/4-노드로 변경
   
        => **rotation**: 왼 또는 오른쪽 3-노드/4-노드 형제에서 값하나 빌려와서 **3-노드로 만듦**
        
   ![image](https://user-images.githubusercontent.com/79195793/120883617-198bb380-c619-11eb-8439-98c61d143fd1.png)





## 2-3-4 를 red-black으로!

**: 2nod를 black 노드로 & 3 node는 쪼개지기 & 4 node 중간을 black 한 후 양쪽을 밑으로(red)**

![image](https://user-images.githubusercontent.com/79195793/120883621-1ee8fe00-c619-11eb-9160-d74ea551310b.png)
