---
title: "Data Science_R"
categories:
  - Data Science
  - R
tags:
  - Data Science
  - R
toc: true
toc_sticky: true
---

# 2.2 데이터 조작

## 2.2.1 기준에 따라 데이터 쪼개고 붙이기

* **split() 함수**: 여러 그룹으로 쪼개어 새로운 데이터 셋 생성

* **subset() 함수**: 데이터프레임의 일부 행(row)를 추출

  ​	=>**subset(데이터, select = 추출대상변수(열), subset = (-)추출 조건)**

* **merge(대상, 대상, by= "지정변수")함수**:  지정한 변수 기준으로 두 데이터 프레임 결합

  ​	-> 지정변수 넣을 때 무조건 "" 붙이기

## 2.2.3  apply 계열 함수

* **apply()**: 행렬 & 데이터프레임의 각 행과 열에 대해 **같은 작업을 반복적 실행**

  ​	=> **새로운 벡터 생성**

   	=>**apply(x=행렬/데이터프레임, MARGIN = 방향, FUN = 함수)**

     * MARGIN = 1: 모든 행 / MARGIN = 2: 모든 열

  ex) apply(X = a, MARGIN = 1, FUN = mean)    # to every row

  

* **lapply(리스트, 적용할 함수)**: 리스트에 대해 apply()하여 **새로운 list로 반환**

  ex) lapply(Hong, length)   # returns a list

* **sapply(리스트 , 적용할 함수)**: 리스트에 대해 apply()하여 **새로운 vector로 반환**

  ex) sapply(Hong, length)   # returns a vector

* **tapply(그룹, 그룹,.., 적용할 함수)**: 특정 변수에 그룹별로 함수 적용

  ex) with(Cars93, tapply(Price, Origin, mean)) 

* **by(데이터, 데이터$그룹, 적용할 함수)**: 모든 변수에 함수 적용

  ex) by(Cars93, Cars93$Origin, summary)

# 2.3 Tidyverse 패키지

* Tidyverse 패키지: 데이터 과학을 위해 디자인된 R-패키지의 모음

- Core tidyverse packages: **`ggplot2`**, `dplyr`, `tidyr`, `readr`, `purrr`, `tibble`, `stringr`, `forcats`



# 2.4 ggplot2를 이용한 데이터 시각화

## 2.4.1 산점도

* **ggplot(data = 데이터셋) + geom_point(mapping= aes(x =, y= ), color="", alpha= , shape=, size= ) + facet_wrap(~)**

  ​	=>x와 y간의 산점도
  * geom_point(): 산점도 그리기

  * mapping= 인수를 이용해 시각화 대상 변수 지정

  * color: 색지정   ex) color= "blue" / color= class => 모든 변수에 색입히기

  * alpha: 명도      ex) alpha= class => 모든 변수에 명도 차

  * shape: 점모양  

  * size: 점크기    ex) size= cyl     => 모든 변수 사이즈 다르게

  * facet_wrap(): 지정한 범주형 변수의 값별로 산점도(1차원)

    ex) facet_wrap(~class, nrow=2) : class별로  2개 행으로 그래프 그리기

  * facet_grid(): 두 개의 범주형 변수값의 조합별로 산점도(2차원)

    ex) facet_grid(drv ~ cyl) : drv + cyl별로 구분

## 2.4.2 Geometric objects

* **geom_**: 데이터를 시각화할 방법(또는 결과적으로 생성되는 객체의 종류)wlwjd

  * geom_point()

  * geom_smooth() : 주변 오차표시

    * geom_smooth(se = FALSE) : 주변 막 없음
    * geom_smooth(data= filter(mpg, class==" ")) : class 지정(고르게)

    ex)

    ```
    ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) + 
      geom_point() +
      geom_smooth()
    ```

    ```pseudocode
    ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) +
      geom_point(mapping = aes(color = class)) + 
      geom_smooth(se = FALSE)
    ```

## 2.4.3 간단한 통계 분석 결과를 시각화

* `geom_bar()`: 막대그래프

  ex) 

  ```pseudocode
  ggplot(data = diamonds) +
    geom_bar(mapping = aes(x = cut, fill = cut)) # x항목별 색칠
  ```

  ex)

  ```pseudocode
  ggplot(data = diamonds) +
    geom_bar(mapping = aes(x = cut, color = clarity)) # Not good
  ```

  ![image-20210609174155935](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609174155935.png)

  ex) **geom_bar에 사용**

  ```pseudocode
  ggplot(data = diamonds) +
    geom_bar(mapping = aes(x = cut, fill = clarity))
  ```

![image-20210609174230559](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609174230559.png)

   * position: 점 표시 방식 지정

     ex) **geom_bar에 사용**

     ```pseudocode
     d <- ggplot(data = diamonds, mapping = aes(x = cut, fill = clarity))
     d + geom_bar(alpha = 1/5, position = "identity")
     ```

     ![image-20210609174421128](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609174421128.png)

     ex)  **geom_bar에 사용**

     ```pseudocode
     d + geom_bar(position = "fill")
     ```

     ![image-20210609174454070](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609174454070.png)

     ex)  **geom_bar에 사용**

     ```pseudocode
     d + geom_bar(position = "dodge")
     ```

     ![image-20210609174520001](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609174520001.png)

     ex)   **geom_point에 사용**

     ```pseudocode
     ggplot(data = mpg) +
       geom_point(
         mapping = aes(x = displ, y = hwy),
         position = "jitter"
       )
     ```

     ![image-20210609174635154](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609174635154.png)

     ex) **`stat_summary`에 사용(선+점**)

     ```pseudocode
     ggplot(data = diamonds) + 
       stat_summary(
         mapping = aes(x = cut, y = depth),
         fun.ymin = min,
         fun.ymax = max,
         fun.y = median
       )
     ```

     ![image-20210609174831836](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609174831836.png)

### 2.4.4 좌표축다루기

* `coord_flip()`: 가로 세로축 swap

  ex) 

  ```
  m + 
    geom_boxplot() +
    coord_flip()
  ```

![image-20210609175000660](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609175000660.png)

* `coord_polar()`: 극좌표 나타냄(원형차트)

  ex)

  ```pseudocode
  dbar <- ggplot(data = diamonds) + 
    geom_bar(
      mapping = aes(x = cut, fill = cut),
      show.legend = FALSE,
      width = 1
    ) +
    theme(aspect.ratio = 1) +
    labs(x = NULL, y = NULL)
    
  dbar + coord_polar()
  ```

  ![image-20210609175339289](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210609175339289.png)

* `coord_fixed()` : 가로축, 세로축 비율 고정

  ex) 

  ```pseudocode
  ggplot(data = mpg, mapping = aes(x = cty, y = hwy)) +
    geom_point() +
    geom_abline() +
    coord_fixed() # coord_fixed(ratio = 1): default
  ```

## 2.4.5 Layered grammer

* 최종 구성 (순서 유의)

  ```pseudocode
  ggplot(data = <DATA>) +
    <GEOM_FUNCTION>(
      mapping = aes(<MAPPINGS>),
      stat = <STAT>,
      position = <POSITION>
    ) +
    <COORDINATE_FUNCTION> +
    <FACET_FUNCTION>
  ```

# 2.5 dplyr package

* **dplyr**: 석가가 원하는 형태로 데이터를 변형하고 이리저리 핸들링하는 행위를 의미하는 신조어

## 2.5.2 기본 함수들

- `filter(대상, 조건)`: 지정 조건에 맞는 관측치 추출

  ex) 

  ```pseudocode
  filter(flights, month == 11 | month == 12)
  filter(flights, month %in% c(11, 12))  #  month 내에 c(11,12)값이 포함된 개수의 합
  ```

- `arrange(대상, 조건)`: 지정 조건에 따라 관측치 정렬

  ex)

  ```pseudocode
  arrange(flights, desc(arr_delay)) # 내림차순 정렬 
  ```

- `select(대상, 조건)`: 변수 추출

  ex)

  ```pseudocode
  select(flights, -(year:day))
  ```

  ```pseudocode
  select(flights, time_hour, air_time, everything())#지정변수가 맨앞으로 
  ```

- `rename()`: 이름변경

- `mutate(대상, 변수 = )`: 기존 변수를 이용해 새로운 변수 생성

  ex) 

  ```pseudocode
  flights_sml <- select(flights, 
                        year:day, 
                        ends_with("delay"), 
                        distance, 
                        air_time)
  mutate(flights_sml,
         gain = arr_delay - dep_delay,
         hours = air_time/60,
         gain_per_hour = gain/hours)
  ```

- `transmute(대상, 변수 =)`: 새로 만든 변수만 남기기

- ★`summarise(대상, 변수 = mean(dep_delay, na.rm=TRUE))`: 그룹별 요약

  ex) 

  ```pseudocode
  by_month <- group_by(flights, year, month) # 그룹 쪼개기
  summarise(by_month, delay = mean(dep_delay, na.rm = TRUE))
  								# by_month의 평균 
  ```

- `group_by(대상, 그룹 기준)`: 지정 변수에 따라 정의된 그룹별로 함수의 적용 범위를 제한

## 2.5.3 pipe 연산자

* **pipe 연산자(%>%)** : `magrittr` 패키지에서 제공하는 기능, 코드의 가독성 및 작성 시 편리 

  ​	=> 데이터 전달

  

