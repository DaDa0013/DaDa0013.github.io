---
title: "Data Science_그래프 그리기"
categories:
  - Data Science
  - matplotlib
  - seaborn
tags:
  - Data Science
  - matplotlib
  - seaborn
toc: true
toc_sticky: true
---

# Matplotlib

: 2차원 플롯을 디자인하는 패키지

* 특징: **많은 운영 체제 & 그래픽 백엔드에서 잘 작동함**

* **Matplotlib.pyplot**

  : matplotlib가 **MATLAB처럼** 작동하도록 만드는 command style functions 집합

  * 역할: figure을 만들고 수정, figure에 plotting area 생성, plot에 label 추가..

  *  목적: interactive plot과 간단한 programmatic plot 생성

    

* **Dual interfaces(이중 인터페이스)**

  1. **편리한 pyplot(MATLAB-style state-based) interface**

     * 자동으로 pyplot 만들기 & figures와 axes 관리 & plotting을 위한 pyplot function 사용
     * plt command(명령)이 적용되는 **현재 figures와 axes**를 트래킹함 ★
     * 단순한 plot들에는 편리하고 빠름

     => **pyplot 모듈의 필요한 형태를 plotting 함수로 직접 그림 그리는 작성**

     

  2. **강력한 OO(objected-oriented) interface**

     * 명시적으로 figures와 axes 생성 =>복잡한 상황과 좀 더 정교한 control이 필요할 때 사용
     * OO interface의 plotting functions는 명시적 figure과 axes 객체의 메서드이다

     => **객체인 figures, Axes를 먼저 생성 후, 원하는 객체에 대하여 그래픽관련 메서드로 그림 작성**

     

### Figure 구성

1. **Figure**: 전체적인 모습

      * 여러 개의 축이 포함될 수 있지만 **일반적으로 하나 이상의 축이 포함**
      * 축을 생성하는 것이 편리=> but, 축을 추가 시 축 레이아웃이 복잡

2. **Axes**: 데이터 공간의 이미지 영역( plot )

   * 여러 개의 axes가 포함될 수 있지만, **주어진 axes object는 하나의 figure에만!**(중복 x)
   * OO interface의 기본

3. **Axis (축)**: 선

   * 그래프 한계를 설정 & 눈금(axis 표시) 및 눈금 label생성

4. **Artist**: figure의 모든것

   * Text, Line 2D, collections, Patch objects… 등이 포함

   *  대부분의 artist는 여러 개의 axes에 공유될 수 X

     ![image](https://user-images.githubusercontent.com/79195793/121133124-eb9ead00-c86c-11eb-9322-697fa1e90443.png)
## Adjusting the plot

**1. Line colors & styles**

​	-> plt.plot() 함수는 **선 색상 및 스타일**을 지정

* **문자열 인수(string arguments)를 허용하는 색상 키워드** 사용 가능 

  ex) plt.plot(x, np.sin(x), color=**"red"**)     //  plt.plot(x, x + 0, linestyle=**'solid'**)

* 지정된 색상이 없는 경우 Matplotlib는 여러 선의 기본 색상 집합을 자동으로 순환합니다.

* **라인 스타일 키워드(line style keyword)** 사용 가능

*  선 스타일과 색상 코드를 **하나의 비키워드 인수(nonkeyword argument)로 결합 가능**

* 매개변수를 명확하게 지정하는 것이 좋음(marker, linestyle, color)



**2. Axes Limits**

* plt.xlim() &  plt.ylim() => 축 한계를 조정
* plt.axis() => [xmin, xmax, ymin, ymax] 로 조정 가능



**3. Labeling plots**

* **plt.title() method** : 타이틀 지정
* **plt.xlabel(),** **plt.ylabel()** : 축 label 지정
* **plt.legend()** : single axes 에 mulpitle lines일 때 유용



## pyplot interface  -> OO-interface methods

* plt.xlabel() -> ax.set_xlabel()

* plt.ylabel() -> ax.set_ylabel()

* plt.xlim() -> ax.set_xlim()

* plt.ylim() -> ax.set_ylim()

* plt.title() -> ax.set_title()

* plt.subplot() -> fig.add_subplot()



## 그림 유형

• **Histogram**

* 간단한 데이터 세트(data set) 이해에 도움

* 매개변수 bin, color, density, cumulative 등을 이용한 다양한 형태 작성

* 여러 그룹을 겹치기 가능 & 누적 빈도 분석 가능



•**Bar chart(막대 그래프)** 

​	-> **height 정보 필수**

* bar(x, height)  => X값과 Y값으로 height를 지정

* 데이터프레임의 경우 height값이 산출 안될 때 => value_counts()로 heigh구하기

* 수평 막대그래프 => barh()



•**Pie chart (원 그래프)**

* 각 요소의 비율을 추출하고 추출된 비율에 따라 원형을 부채꼴로 분할한 그래프
* 각 요소의 비율을 비교하기 편리하다.



•**박스플롯** **(boxplot)**

* Boxplot은 중앙값, 4분위값, 이상값 등을 체계적으로 보여주는 그래프

* 여러 개 boxplot 동시에 작성 가능 

  ​	=> 여러 개를 하나의 그림에 작성하면 **중심위치 및 산포에 대한 비교가 편리**하다.

* 입력 데이터는 array or a sequence of vectors가 될 수 있다.
*  Labels로 각 sequence을 지정 가능

![image-20210608135127750](C:\Users\yttn0\AppData\Roaming\Typora\typora-user-images\image-20210608135127750.png)

•**Scatter** **plots**

* 점, 원 또는 다른 모양으로 개별적으로 표시

* plt.plot()으로 linestype 지정 가능 => 다양한 시각화 옵션 사용 가능

  ​					ex) plt.plot(x, np.sin(x), marker = 'o', linestyle = ' ', color = 'black')  

* plt.scatter() => 각 점들의 properties(속성)을 **개별적으로 제어 &데이터 매핑**

* 데이터 세트 커지면 -> scatter 가 더 유용



## Multiple Subplots

-> 여러 그림 나란히 비교

•**plt.subplot**

​	=>subplot 추가
 	-> subpot(n, n, i) : n*n개, 들어갈 인덱스

* Figure객체명.add_subplot() =>Axes 객체를 생성

  & 생성된 Axes객체에 **OO-interface** 방식을 사용 가능



•**plt.subplots**: the whole grid in one Go

* figure 와 supblots 집합 생성
*  단일 하위 그림을 만드는 대신 단일 선에 전체 gird of subplot를 만들어 Numpy 배열로 반환
* sharex & sharey 지정 =>grid inner lables 자동 제거하여 그림을 더 깨끗하게



## Seaborn

=> 파이썬의 **통계 그래픽**을 만드는 라이브러리

* matplotblib에 구축되었음 & pandas data structure과 밀접
* matplotlib로 plot그림
* seaborn로 웬만한 건 할 수 있지만, **customization은 matplotlib를 사용해야 함**

[제공 기능]

* 여러 변수 간의 관계를 조사하기 위한 dataset-oriented API
* 범주형 변수(categorical varibles)로 관측치 (observation)와 집계 통계량(aggregate statistics) 표시
* univariate 나 bivariate를 시각화 & data의 subsets 간에 비교하는 옵션
* 여러 종류 종속 변수에 대한 선형 회귀 모형(linear regression models)을  자동으로 estimate & plot
*  복잡한 데이터셋의 전체 구조에 대한 편리한 보기
* multi-plot grids를 위한 고급 추상화(abstractions) -> 복잡한 시각화를 쉽게 구축할 수 있도록
*  몇 가지 기본 제공 테마를 사용하여 매트플롯 리브 그림 스타일링에 대한 간결한 제어
*  데이터의 패턴을 충실하게 나타내는 색상 팔레트를 선택하는 도구



### seaborn plots

•**히스토그램 작성하기**

* sns.distplot() : 히스토그램과 확률밀도함수를 표현

* bins, hist, kde 매개변수를 지정하여 필요한 형태를 변경



•**막대그래프 작성하기**

* Countplot() :범주형 변수의 개수를 따로 산출하지 않고도 막대그래프를 작성

  ​	-> **사용하는 데이터프레임을 data 매개변수에 지정해야함**

* Barplot() : x 매개변수에 범주형변수, y 매개변수에 연속형변수 지정하여 범주별 평균값 막대 그래프산출



•**Boxplot** **작성하기**

–Draw a box plot to show distribution with respect to categories



•**Scatter plots**

* 여러 의미 그룹화의 가능성을 가진 산점도 그림
*  x- y 관계 -> 색조, 크기 및 스타일 매개 변수를 사용하여 데이터의 여러 하위 집합에 대해 표시 가능



•**Regression plots**

* regplot()함수: Plot data and a linear regression model plot.



•**Pairplot()**

* 데이터의 각 숫자 변수가 단일 행에 걸쳐 Y축에서 공유되고 단일 열에 걸쳐 x축에서 공유되는 grid of Axes 생성 

  

• **FacetGrid()**

* 그룹별로 여러 그림을 한꺼번에 작성하는 방법

* FacetGrid() 로 데이터프레임과 그룹을 구분할 **열이름**을 전달하여 객체를 생성

* 객체명.map()로 그래프의 종류 및 필요한 정보를 제공하여 그림을 그림

