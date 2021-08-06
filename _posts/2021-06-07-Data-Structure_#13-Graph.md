---
title: "Data Structure_#13 Graph"
categories:
  - Data Structure
  - Python
tags:
  - Data Structure
  - Python
toc: true
toc_sticky: true
---

# Graph

:**가장 복잡한(일반적인) 자료구조**

- G=(V,E)      v=vertex(정점) set, E= edge(에지) set

* **degree(분지수)**: 어떤 노드에 연결된 에지 수

* **인접성**

​      (1) adjacent: 에지(u, v)가 존재할 때 u, v는 인접(adjacent)

​      (2) incident: 에지 e=(u, v)에 대해, e는 u와 v에 인접(incident)

* **경로(path)**

​      (1)단순 경로

​      (2)경로의 길이

* **사이클**

​      (1) 트리 : 사이클이 없는 연결 그래프 (두 노드를 연결하는 경로가 무조건 1개)

​      (2) 포리스트: 연결 트리의 집합 

- **방향그래프/무방향그래프**

![image](https://user-images.githubusercontent.com/79195793/120956839-71e4c180-c78f-11eb-9259-c812aadc4761.png)

## 표현법

1.  인접행렬(adjacency matrix): (가중치를 직접 적음) 대각선은 0이나 1로 통일 =>**메모리 낭비**

   ​		=> **2차원 행렬(배열)로**

2. 인접리스트(adjacency list): 에지만 표현하는 것 =>**인접행렬 보완**

​                 =>**1차원 배열들로**



![image](https://user-images.githubusercontent.com/79195793/120956848-76a97580-c78f-11eb-8de3-770a5fcfd5d4.png)

### 기본연산시간

![image](https://user-images.githubusercontent.com/79195793/120956887-8fb22680-c78f-11eb-933a-56f1478ea3b4.png)

    Sparse(에지수<노드수)  dense(에지수>노드수)
​							

## 그래프 순회

**1. DFS(Depth First Search)**

​	: 길이우선탐색

=> **현재 방문 노드에서 방문하지 않는 이웃 노드를 방문 & backtrack으로 다시 안 가본곳 방문**

=> 그 과정을 통해 만들어진 트리가 **DFS 트리**

![image](https://user-images.githubusercontent.com/79195793/120956915-9c367f00-c78f-11eb-8895-9c10f9738c90.png)

![image](https://user-images.githubusercontent.com/79195793/120956921-a0629c80-c78f-11eb-8f4f-a813fa88f0a1.png)![image](https://user-images.githubusercontent.com/79195793/120956930-a48eba00-c78f-11eb-970a-c5a20e2c8a3b.png)

![image](https://user-images.githubusercontent.com/79195793/120956936-a8224100-c78f-11eb-8aa6-1d7297c58ae8.png)

​											[비재귀적]

![image](https://user-images.githubusercontent.com/79195793/120956940-ac4e5e80-c78f-11eb-915e-ca7a61a83b0d.png)

​																**backdege 존재=사이클 존재**

![image](https://user-images.githubusercontent.com/79195793/120956943-af494f00-c78f-11eb-9a72-50a989575af2.png)

2.  **DAG(Directed Acyclic Graph)**

   ![image](https://user-images.githubusercontent.com/79195793/120956952-b5d7c680-c78f-11eb-8413-10fbc0df4044.png)

   

   ​		-> 사이클이 없는 방향 그래프

   ![image](https://user-images.githubusercontent.com/79195793/120956972-bf612e80-c78f-11eb-82f9-4e98c0eb0bff.png)


    Post가 가장 늦게 끝나는( 다 받아야하는) 것이 제일 나중으로 (post time이 가장 작은 것!)

   ​							=> Post 역순으로(큰 것부터 나열)!

   ​			Ex) A, C, B, F, H, D, I, G

   

3. BFS( 너비우선)

​      =>  같은 레벨기준으로 안 가본 곳 가기
