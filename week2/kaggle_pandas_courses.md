# Creating, Reading and Writing
## 0\. Pandas 시작하기

천리길도 임포트부터

```
import pandas as pd
```

## 1\. 데이터 생성하기

Pandas에는 **DataFrame**과 **Series**라는 핵심 객체들이 있다.

### DataFrame

Dataframe은 특정 값을 담고 있는 개별 엔트리들의 배열을 포함한 테이블이다. 각 항목은 1개의 행, 1개의 열과 상응한다. `pd.DataFrame()` 구조체를 통해 DataFrame을 생성할 수 있다. 열 이름('Yes', 'No')을 key로, 엔트리 리스트를 value로 하는 딕셔너리를 통해 새 DataFrame을 정의할 수 있다.

```
pd.DataFrame({'Yes': [50, 21], 'No': [131, 2]})
```

아래의 표는 위와 같은 코드를 입력했을 때의 결과다.

|   | Yes | No |
| --- | --- | --- |
| 0 | 50 | 131 |
| 1 | 21 | 2 |

이 예제에서 "0, No" 엔트리의 값은 131이고, "0, Yes" 엔트리는 50의 값을 갖는다. 엔트리는 정수 외의 값도 가질 수 있다.  
`index`를 통해 row에 label을 붙일 수도 있다. 아래는 String 타입의 값과 index를 갖는 DataFrame 예시다.

```
pd.DataFrame({'Bob': ['I liked it.', 'It was awful.'], 
              'Sue': ['Pretty good.', 'Bland.']},
             index=['Product A', 'Product B'])
```

|   | Bob | Sue |
| --- | --- | --- |
| Product A | I liked it. | It was awful. |
| Product B | Pretty good. | Bland. |

### Series

Series는 데이터 값의 시퀀스다. DataFrame이 테이블이라면, Series는 리스트다. 본질적으로 Series는 DataFrame의 열 한 개이다. `pd.Series()` 구조체를 통해 생성 가능하다.

```
pd.Series([1, 2, 3, 4, 5])
```

```
0    1
1    2
2    3
3    4
4    5
```

Series는 DataFrame과는 다르게 열마다 이름을 갖지 않는 대신, Series 전체를 가리키는 하나의 이름을 갖는다(`name`). 그리고 DataFrame처럼 `index`를 통해 row label을 붙여줄 수 있다.

```
pd.Series([30, 35, 40], index=['2015 Sales', '2016 Sales', '2017 Sales'], name='Product A')
```

```
2015 Sales    30
2016 Sales    35
2017 Sales    40
Name: Product A, dtype: int64
```

## 3\. 데이터 읽어오기

`pd.read_csv()` 함수를 통해 CSV(Comma-Separated Values) 파일을 DataFrame으로 읽어들일 수 있다.

```
data = pd.read_csv("/data/mosigi.csv")
```

# Indexing, Selecting & Assigning (iloc, loc)
## 0\. Python 기본 접근자

DataFrame의 각 column에 접근할 수 있는 방법은 여러가지가 있다. 그 중, 파이썬이 기본적으로 제공하는 2가지 방법에 대해서 먼저 알아보자.

첫번째 방법으로, DataFrame의 column index를 하나의 attribute로서 접근하는 방법이다. 이 방법으로 위의 DataFrame의 `country` column에 `reviews.country` 코드를 통해 접근할 수 있다.

```
reviews.country
```

```
0            Italy
1         Portugal
            ...   
129969      France
129970      France
Name: country, Length: 129971, dtype: object
```

두번째는 파이썬 딕셔너리에서 indexing operator(`[]`)로 value 값을 가져오는 것과 동일한 형식으로 접근하는 방법이다. DataFrame에서도 column index를 key처럼 사용하여 해당 인덱스의 열을 가져올 수 있다.

```
reviews['country']
```

```
0            Italy
1         Portugal
            ...   
129969      France
129970      France
Name: country, Length: 129971, dtype: object
```

두 방법 모두 동일한 문법적 효력을 가지므로 둘 중에 본인에게 더 착착 감기는 방법을 사용하면 된다. 다만 첫번째 방법(`reviews.country`)는 column index에 공백 문자나 예약어가 들어가있는 경우(e.g. country providence) 작동하지 않으므로, 이 경우에는 무조건 두번째 방법(`reviews['country']`)를 사용해야 한다.

위의 방법을 통해 한 번 걸러진 데이터에서 `[]` 연산자를 한 번 더 사용하면 특정한 하나의 값만 가져올 수도 있다.

```
reviews['country'][0]
```

```
'Italy'
```

## 1\. Pandas 접근자

Pandas에도 접근 연산이 가능한 `loc`, `iloc`이 있다. (원문에는 얘네가 operators라고 되어 있는데, 얘네가 함수인지 연산자인지는 잘 모르겠다) 여기서 주의해야 할 점이 있는데, 보통 Python에서는 column-first, row-second를 따르는 반면 `loc`과 `iloc`는 row-first, column-second를 따른다는 것이다. `[]` 안에서 첫번째로 오는 것이 행 선택자고, 두번째로 오는 것이 열 선택자임을 기억하자.

### 인덱스 기반 선택

Pandas의 인덱싱은 2가지 패러다임으로 동작하는데, 첫번째가 인덱스 기반 선택이다. 데이터의 수적 위치에 기반하여 데이터를 선택하는 것이며, `iloc`이 이 패러다임을 따른다.  
아래 코드는 `iloc`을 사용해 DataFrame의 첫번째 행을 가져오는 예시다.

```
reviews.iloc[0] # row-first이므로 0번째 Row를 가져온다.
```

```
country                                                    Italy
description    Aromas include tropical fruit, broom, brimston...
                                     ...                        
variety                                              White Blend
winery                                                   Nicosia
Name: 0, Length: 13, dtype: object
```

`loc`과 `iloc`는 row-first, column-second을 따르므로 column 한 줄을 가져오는 것은 row 한 줄을 가져오는 것보다 조금 복잡하다.

```
reviews.iloc[:, 0] # ':'는 전체를 의미한다.
```

```
0            Italy
1         Portugal
            ...   
129969      France
129970      France
Name: country, Length: 129971, dtype: object
```

`x:y`는 x번째 값부터 y-1번째 값까지를 의미한다. 이 부분을 잘 이용해서 슬라이싱하면 된다.

### 레이블 기반 선택

`loc` 연산은 데이터의 위치 값이 아닌 인덱스 값으로 접근하는 레이블 기반 선택을 바탕으로 한다.

```
reviews.loc[0, 'country']
```

배열을 통해 여러 열을 선택할 수도 있다.

```
reviews.loc[:, ['taster_name', 'taster_twitter_handle', 'points']]
```

### `loc`과 `iloc`의 차이점

`iloc`는 파이썬의 기본 인덱싱 방법을 따른다. 즉, `0:10`은 0부터 9까지의 엔트리를 선택한다. 반면에 `loc`은 예외적으로 `0:10`이 0부터 10까지의 엔트리를 선택한다.  
이런 차이가 있는 이유는 `loc`이 string 등을 포함한 다른 모든 stdlib 타입으로 인덱싱할 수 있기 때문이다. 숫자 인덱스를 입력받는 `iloc`에서는 필요한 부분의 시작부터 (끝-1)까지 작성하는 것이 어려운 일이 아니다. 그냥 원하는 범위의 끝 숫자에서 1만 빼주면 되기 때문이다. 하지만, 예를 들어, 나에게 야채 이름을 인덱스로 가지는 DataFrame이 있고, 여기서 Apples부터 Potatoes까지의 열들을 loc으로 가져오고 싶은 상황이라고 해보자. 만약 `loc`가 `iloc`과 같은 인덱싱 매커니즘을 가졌다면 우리는 Potatoes의 다음 column index의 이름을 알아야 하는 수고가 필요하다.

## 2\. 인덱스 조작하기

레이블 기반 선택의 힘은 인덱스에 붙은 레이블에서 오지만, 이 인덱스도 우리 입맛대로 바꿀 수 있다.  
`set_index(keys)` 메서드를 통해 기존 DataFrame의 열 중 하나를 인덱스로 설정할 수 있다. keys 자리에 인덱스로 설정하길 원하는 열의 레이블 이름을 적으면 된다.

```
reviews.set_index("title")
```

## 3\. 조건부 선택

간단한 비교문을 통해 조건에 해당하는 열들은 True, 해당하지 않은 열에 False 값을 갖는 Series를 얻을 수 있다.

```
reviews.country == 'Italy'
```

```
0          True
1         False
          ...  
129969    False
129970    False
Name: country, Length: 129971, dtype: bool
```

이렇게 각 레코드의 country 값을 기반으로 생성된 boolean 값을 갖는 Series를 `loc`과 함께 사용하면 특정 데이터만 뽑아올 수 있다. 아래와 같이 코드를 작성하면 country 열의 값이 'Italy'인 레코드만 추출할 수 있다.

```
reviews.loc[reviews.country == 'Italy']
```

조건문과 함께 앰퍼샌드(`&`)와 파이프(`|`)를 사용할 수도 있다.

```
reviews.loc[(reviews.country == 'Italy') & (reviews.points >= 90)]
```

```
reviews.loc[(reviews.country == 'Italy') | (reviews.points >= 90)]
```

Pandas 내장 조건부 선택자 중 하나인 `isin`을 사용할 수도 있다. 리스트 중 포함하고 있는 값이 있는 레코드를 선택할 수 있다. 아래는 `isin`으로 country 항목에서 Italy나 France를 포함한 레코드를 추출하는 코드다.

```
reviews.loc[reviews.country.isin(['Italy', 'France'])]
```

또다른 Pandas 내장 조건부 선택자로 `isnull`, `notnull`이 있다. 이 메서드들은 NaN이거나, NaN이 아닌 레코드들만 가져온다.

```
reviews.loc[reviews.price.notnull()]
```

```
reviews.loc[reviews.price.isnull()]
```

## 4\. 데이터 할당

DataFrame에 상수값이나 iterable을 할당할 수 있다.

```
reviews['critic'] = 'everyone'
reviews['critic']
```

```
0         everyone
1         everyone
            ...   
129969    everyone
129970    everyone
Name: critic, Length: 129971, dtype: object
```

```
reviews['index_backwards'] = range(len(reviews), 0, -1)
reviews['index_backwards']
```

```
0         129971
1         129970
           ...  
129969         2
129970         1
Name: index_backwards, Length: 129971, dtype: int64
```

# Summary Functions and Maps
## 0\. 데이터의 개요 알아보기

Pandas는 데이터를 재구조화하여 보여주는 간단한 개요 함수들을 제공한다. `describe()` 메서드같은 경우는 주어진 열에 대해 높은 수준의 정보를 제공한다. `describe()`는 데이터의 타입마다 각기 다른 식의 개요를 보여준다. 여기서 사용하는 예시 DataFrame인 reviews는 아래와 같다.

```
reviews.points.describe()
```

```
count    129971.000000
mean         88.447138
             ...      
75%          91.000000
max         100.000000
Name: points, Length: 8, dtype: float64
```

위의 결과는 데이터 타입이 numerical인 경우이며, 아래는 데이터 타입이 string인 경우의 결과다.

```
reviews.taster_name.describe()
```

```
count         103727
unique            19
top       Roger Voss
freq           25514
Name: taster_name, dtype: object
```

`describe()`로 모든 개요 정보를 뭉텅이로 얻을 수도 있지만, 아래의 예제처럼 DataFrame이나 Series에서 특정한 통계값만 얻을 수도 있다.

```
reviews.points.mean()  # 평균
```

```
reviews.points.unique()  # 고유값들의 리스트
```

```
reviews.points.value_counts()  # 고유값들의 출현 빈도
```

## 1\. 데이터를 매핑하는 방법

`map`은 수학에서 채용한 용어로, 한 집합에서 다른 집합으로 "매핑"시키는 함수를 가리킨다. 데이터 사이언스에서 기존 데이터를 다른 방식으로 표현하고 싶을 때, 지금과 다른 포맷으로 변환해야 할 때 `map`을 사용한다.  
매핑 방법에는 2가지가 있으며, 첫번째 방법은 `map()` 함수를 사용하는 방법이다. `map()`은 단일 Series에 사용할 수 있으며, 아래는 `map()`을 points 속성(reviews.points)에 사용하여 points 속성을 'points 값과 평균의 오차'로 갱신하는 예시다.

```
reviews_points_mean = reviews.points.mean()
reviews.points.map(lambda p: p - review_points_mean)
```

```
0        -1.447138
1        -1.447138
            ...   
129969    1.552862
129970    1.552862
Name: points, Length: 129971, dtype: float64
```

단일 Series의 값만 바꾸는 `map()`과는 다르게, `apply()`는 사용자 정의 메서드를 각 행마다 적용하여 DataFrame 전체를 변형할 수 있다.

```
def remain_points(row):
    row.points -= review_points_mean
    return row

reviews.apply(remean_points, axis='columns')
```

호출 시 `axis='index'` 값을 주면 행마다 적용하는 대신 열마다 적용하게 된다.

## 2\. Pandas 내장 매핑 연산자를 사용한 매핑 방법

Pandas는 다양한 매핑 연산자를 내장 기능으로 제공한다. 아래는 내장 매핑 연산자를 이용해 points 속성을 재정의하는 더 빠른 방법이다.

```
review_points_mean = reviews.points.mean()
reviews.points - review_points_mean
```

```
0        -1.447138
1        -1.447138
            ...   
129969    1.552862
129970    1.552862
Name: points, Length: 129971, dtype: float64
```

Pandas는 길이가 같은 두 Series 사이의 매핑 연산도 가능하다.

```
reviews.country + " - " + reviews.region_1
```

```
0            Italy - Etna
1                     NaN
               ...       
129969    France - Alsace
129970    France - Alsace
Length: 129971, dtype: object
```

이런 식의 연산은 Pandas에 내장된 속도 향상을 위한 매커니즘을 따르기 때문에 `map()`, `apply()`보다 빠르다. 다른 기본 파이썬 연산자(`<`, `>`, `==` 등등)도 같은 방식으로 작동한다.  
하지만 얘네들은 `map(),` `apply()`보다 유연하게 새용하기는 힘드므로, 상황에 맞춰 필요한 연산자나 메서드를 찾아 쓰도록 하자.

# Grouping and Sorting
## 그룹별 분석 - `groupby`, `agg`

`groupby()`로 같은 값끼리 하나의 인덱스로 묶을 수 있다.

```
reviews.groupby('points').points.count()
```

```
points
80     397
81     692
      ... 
99      33
100     19
Name: points, Length: 21, dtype: int64
```

`groupby()`로 같은 포인트 값을 갖는 와인 리뷰끼리 그룹을 생성한 뒤, `points` column으로 묶고 `count()`로 각 그룹별 빈도수를 체크했다. 위 결과는 `value_counts()`와 동일한 결과이기도 하다.

`groupby()`는 기준에 따라 같은 값을 갖는 레코드끼리 DataFrame을 구성한다. 이 DataFrame은 `apply()`를 통해 직접 접근할 수 있다. 아래는 `groupby('winery')`로 와이너리별로 묶인 DataFrame에 `apply()`를 사용해 각 와이너리의 첫번째 와인 이름을 가져오는 예시다.

```
reviews.groupby('winery').apply(lambda df: df.title.iloc[0])
```

```
winery
1+1=3                          1+1=3 NV Rosé Sparkling (Cava)
10 Knots                 10 Knots 2010 Viognier (Paso Robles)
                                  ...                        
àMaurice    àMaurice 2013 Fred Estate Syrah (Walla Walla V...
Štoka                         Štoka 2009 Izbrani Teran (Kras)
Length: 16757, dtype: object
```

`groupby()`에는 단일 column명뿐만 아니라 리스트를 통해 여러개의 column도 전달할 수 있다. 아래는 `country`와 `province`로 데이터를 그룹화한 다음, 각 DataFrame에서 가장 높은 point를 가진 와인을 표시하는 예다.

```
reviews.groupby(['country', 'province']).apply(lambda df: df.loc[df.points.idxmax()])
```

`groupby()`와 함께 `agg()`를 사용하면 여러 개의 함수를 동시에 적용할 수 있다.

```
reviews.groupby(['country']).price.agg([len, min, max])
```

```
        len    min    max
country            
Argentina    3800    4.0    230.0
Armenia    2    14.0    15.0
...    ...    ...    ...
Ukraine    14    6.0    13.0
Uruguay    109    10.0    130.0
```

## 정렬 - `sort_values`

그룹핑된 데이터는 기본적으로 인덱스 순서에 따라 정렬되어 있다. 우리의 입맛에 맞게 데이터를 정렬하기 위해서는 `sort_values()` 메서드를 사용하면 된다. 기본적으로 `sort_values()`는 오름차순으로 정렬하며, 내림차순으로 정렬하고 싶을 때에는 `ascending=False` 값을 전달하면 된다. 인자로 아무런 값도 전달하지 않을 시에는 인덱스 순서대로 정렬한다. 리스트를 통해 하나 이상의 열을 기준으로 정렬할 수도 있다.

```
sorted_by_points = reviews.reset_index()

sorted_by_points.sort_values(by='points')
sorted_by_points.sort_values(by='points', ascending=False)
sorted_by_points.sort_values(by=['country', 'points'], ascending=False)
```
