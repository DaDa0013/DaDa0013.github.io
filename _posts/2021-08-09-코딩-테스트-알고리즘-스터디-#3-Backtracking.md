---
title: "코딩 테스트 알고리즘 스터디 #3 Backtracking"
categories:
  - Coding Test
  - Algorithm
tags:
  - coding Test
  - Algorithm
  - C++
toc: true
toc_sticky: true
excerpt: "Backtracking"

---



## 단일 객체를 이동

### 겹치는지 여부 확인
![image](https://user-images.githubusercontent.com/79195793/128682683-d1b492a8-1c8d-47b0-bbba-dc0007179281.png)

1. **직접 순회**

   ```c++
   bool IsDuplicatePointExist(){
       for(int i = 1;i <= n;i++)
           for(int j = i + 1; j <= n;j++)
               if(points[i] == points[j])
                   return true;
       return false;
   }
   ```

   * 시간복잡도: O(N^2)

   * 공간복잡도: O(1)

     

2. **checked 배열**

   -> 격자를 만들어서 중복 위치 확인

   ```c++
   bool IsDuplicatePointExist(){
       for(int i = 1;i <= n;i++){
   		int target_x=points[i].first;
           int target_y=points[i],second;
           if(checked[target_x][target_y] == true)
               return true;
      		checked[target_x][target_y] = true;
       }
       return false;
   }
   ```

   * 시간복잡도: O(N)

   * 공간복잡도: O(R^2) (R= 격자 크기)

     

3. **Hash set**

   -> Hash로 저장해서 찾아보기

   ![image](https://user-images.githubusercontent.com/79195793/128682719-b0e1da2b-c385-4bf1-be84-ba2da04635dc.png)


   (아래는 python 코드)

   ```python
   def is_duplicate_point_exist():
       hash_set = set()
      
       for point in points:
           if point in hash_set:
               return True
           hash_set.add(point)
           
       return false;
   ```

   => c++은 map을 사용하면 됨!

   * 시간복잡도: O(N)

   * 공간복잡도: O(N)

   

## 여러 객체를 이동

### 다중 객체에 대한 표현

1. **1차원 배열**

   ```c++
   for(int i =1; i<=n;i++){
   	int x, y;
   	cin >> x >> y;
       marbles.push_back(make_tuple(x,y));
   }
   ```

   => marbles: (1,1) (1,3) (3,2)...

   * 탐색: O(N)

   * 조회: O(1)

   * 삽입: O(N)

   * 삭제: O(N)

     

2. **2차원 배열**

   => 격자를 직접 만듦

   ```c++
   for(int i=1; i<=n; i++){
   	int x, y;
       cin >> x >> y;
       marbles[x][y] = i;
   }
   ```

   * 탐색: O(1)
   * 조회(위치 조회): O(R^2) (R: 격자 크기)
   * 삽입: O(1)
   * 삭제: O(R^2)



# Backtracking

## Recursion

**: 자기 자신을 호출**

```pseudocode
if(종료 조건) => 종료 조건은 지극히 당연한걸로
	return ;
else
	return; <- 재귀 조건
```



문제: 1부터 N까지의 자연수의 합을 구하는 코드를 작성하시오



* recursion

  ```c++
  #include <iostream>
  
  using namespace std;
  int sum(int n){
      if(n==1)
          return 1;
      else
          return n + sum(n-1);
  }
  int main(){
      int n = 5;
      cout<< sum(n)<<endl;
  }
  ```

![image](https://user-images.githubusercontent.com/79195793/128682774-ace33882-1c0c-4f7c-9726-34b2458ca6d3.png)

### K개 중 하나를 N번 선택하기

문제: 

1부터 k사이의 숫자를 하나 고르는 행위를 N번 반복하여 

나올 수 있는 모든 서로 다른 순서쌍을 구해주는 코드를 작성하시오

단, 한 순열 내에서는 숫자들을 오름차순으로 정렬하여 출력한다.



```c++
int k = 3;
int n = 2;
vector<int> permutation;

void PrintPermuation(){
	for(int i=0; i<permuation.size(); i++)
        cout << permutation[i] << " ";
    cout << endl;
}
void FindPermutaitons(int cnt){
    if(cnt == n){
		PrintPermuation();
        return;
    }
    for(int i = 1;i<=k; i++){
        permutation.push_back(i);
        findPermutations(cnt + 1);
        permutation.pop_back();
    }
}
int main(){
    FindPermutation(0);
    return 0;
}
```



### N개 중에 M개 고르기

문제: N자리의 2진수중 1의 개수가 M개인 수들을 출력하는 코드 작성(증가하는 순으로)



* 구현 1

  ```c++
  int n = 3, m = 2;
  vector<int> answer;
  
  void Print(){
  	for(int i = 0; i < answer.size(); i++)
          cout<< answer[i]<<" ";
      cout<<endl;
  }
  void Choose(int curr_num){
      if(curr_num == n+1){
          int cnt == 0;
          for(int i=0;i<answer.size();i++)
              if(answer[i]==1)
                  cnt++;
          
          if(cnt == m)
              Print();
          return;
      }
      answer.push_back(0);
      Choose(curr_num + 1);
      answer.pop_back();
      
      answer.push_back(1);
      Choose(curr_num + 1);
      answer.pop_back();
      
      return;
  }
  int main(){
  	Choose(1);
      return 0;
  }
  ```

* 구현 2

  >  Choose(int curr_num, int cnt):
  >
  > 지금까지 선택한 1의 개수가 cnt개라 했을 때, curr_num번째 위치에 0혹은 1을 선택하는 함수

  ```c++
  int n = 3, m = 2;
  vector<int> answer;
  
  void Print(){
  	for(int i = 0; i < answer.size(); i++)
          cout<< answer[i]<<" ";
      cout<<endl;
  }
  void Choose(int curr_num, int cnt){
      if(curr_num == n+1){
          if(cnt == m)
          	Print();
          return;
      }
      
      answer.push_back(0);
      Choose(curr_num + 1,cnt);
      answer.pop_back();
      
      answer.push_back(1);
      Choose(curr_num + 1,cnt+1);
      answer.pop_back();
      
      return;
  }
  int main(){
  	Choose(1, 0);
      return 0;
  }
  ```

  

