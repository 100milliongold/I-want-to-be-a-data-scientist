[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://bit.ly/open-data-01-apt-price-input)

# 전국 신규 민간 아파트 분양가격 동향

2013년부터 최근까지 부동산 가격 변동 추세가 아파트 분양가에도 반영될까요? 공공데이터 포털에 있는 데이터를 Pandas 의 melt, concat, pivot, transpose 와 같은 reshape 기능을 활용해 분석해 봅니다. 그리고 groupby, pivot_table, info, describe, value_counts 등을 통한 데이터 요약과 분석을 해봅니다. 이를 통해 전혀 다른 형태의 두 데이터를 가져와 정제하고 병합하는 과정을 다루는 방법을 알게 됩니다. 전처리 한 결과에 대해 수치형, 범주형 데이터의 차이를 이해하고 다양한 그래프로 시각화를 할 수 있게 됩니다.


## 다루는 내용
* 공공데이터를 활용해 전혀 다른 두 개의 데이터를 가져와서 전처리 하고 병합하기
* 수치형 데이터와 범주형 데이터를 바라보는 시각을 기르기
* 데이터의 형식에 따른 다양한 시각화 방법 이해하기

## 실습
* 공공데이터 다운로드 후 주피터 노트북으로 로드하기
* 판다스를 통해 데이터를 요약하고 분석하기
* 데이터 전처리와 병합하기
* 수치형 데이터와 범주형 데이터 다루기
* 막대그래프(bar plot), 선그래프(line plot), 산포도(scatter plot), 상관관계(lm plot), 히트맵, 상자수염그림, swarm plot, 도수분포표, 히스토그램(distplot) 실습하기

## 데이터셋
* 다운로드 위치 : https://www.data.go.kr/dataset/3035522/fileData.do

### 전국 평균 분양가격(2013년 9월부터 2015년 8월까지)
* 전국 공동주택의 3.3제곱미터당 평균분양가격 데이터를 제공

###  주택도시보증공사_전국 평균 분양가격(2019년 12월)
* 전국 공동주택의 연도별, 월별, 전용면적별 제곱미터당 평균분양가격 데이터를 제공
* 지역별 평균값은 단순 산술평균값이 아닌 가중평균값임


```python
%ls data
```

    '전국 평균 분양가격(2013년 9월부터 2015년 8월까지).csv'
    '주택도시보증공사_전국 평균 분양가격(2019년 12월).csv'



```python
# 파이썬에서 쓸 수 있는 엑셀과도 유사한 판다스 라이브러리를 불러옵니다.
import pandas as pd
```

## 데이터 로드
### 최근 파일 로드
공공데이터 포털에서 "주택도시보증공사_전국 평균 분양가격"파일을 다운로드 받아 불러옵니다.
이 때, 인코딩을 설정을 해주어야 한글이 깨지지 않습니다.
보통 엑셀로 저장된 한글의 인코딩은 cp949 혹은 euc-kr로 되어 있습니다.
df_last 라는 변수에 최근 분양가 파일을 다운로드 받아 로드합니다.

* 한글인코딩 : [‘설믜를 설믜라 못 부르는’ 김설믜씨 “제 이름을 지켜주세요” : 사회일반 : 사회 : 뉴스 : 한겨레](http://www.hani.co.kr/arti/society/society_general/864914.html)

데이터를 로드한 뒤 shape를 통해 행과 열의 갯수를 출력합니다.


```python
# 최근 분양가 파일을 로드해서 df_last 라는 변수에 담습니다.
# 파일로드시 OSError가 발생한다면, engine="python"을 추가해 보세요.
# 윈도우에서 파일탐색기의 경로를 복사해서 붙여넣기 했는데도 파일을 불러올 수 없다면
# 아마도 경로에 있는 ₩ 역슬래시 표시를 못 읽어왔을 가능성이 큽니다. 
# r"경로명" 으로 적어주세요.
# r"경로명"으로 적게 되면 경로를 문자 그대로(raw) 읽으라는 의미입니다.
df_last = pd.read_csv("data/주택도시보증공사_전국 평균 분양가격(2019년 12월).csv", encoding="cp949")
df_last.shape
```




    (4335, 5)




```python
# head 로 파일을 미리보기 합니다.
# 메소드 뒤에 ?를 하면 자기호출 이라는 기능을 통해 메소드의 docstring을 출력합니다.
# 메소드의 ()괄호 안에서 Shift + Tab키를 눌러도 같은 문서를 열어볼 수 있습니다.
# Shift + Tab + Tab 을 하게 되면 팝업창을 키울 수 있습니다.
df_last.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>전체</td>
      <td>2015</td>
      <td>10</td>
      <td>5841</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울</td>
      <td>전용면적 60㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5652</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울</td>
      <td>전용면적 60㎡초과 85㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5882</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울</td>
      <td>전용면적 85㎡초과 102㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5721</td>
    </tr>
    <tr>
      <th>4</th>
      <td>서울</td>
      <td>전용면적 102㎡초과</td>
      <td>2015</td>
      <td>10</td>
      <td>5879</td>
    </tr>
  </tbody>
</table>
</div>




```python
# tail 로도 미리보기를 합니다.
df_last.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4330</th>
      <td>제주</td>
      <td>전체</td>
      <td>2019</td>
      <td>12</td>
      <td>3882</td>
    </tr>
    <tr>
      <th>4331</th>
      <td>제주</td>
      <td>전용면적 60㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4332</th>
      <td>제주</td>
      <td>전용면적 60㎡초과 85㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>3898</td>
    </tr>
    <tr>
      <th>4333</th>
      <td>제주</td>
      <td>전용면적 85㎡초과 102㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4334</th>
      <td>제주</td>
      <td>전용면적 102㎡초과</td>
      <td>2019</td>
      <td>12</td>
      <td>3601</td>
    </tr>
  </tbody>
</table>
</div>



### 2015년 부터 최근까지의 데이터 로드
전국 평균 분양가격(2013년 9월부터 2015년 8월까지) 파일을 불러옵니다.
df_first 라는 변수에 담고 shape로 행과 열의 갯수를 출력합니다.


```python
# 해당되는 폴더 혹은 경로의 파일 목록을 출력해 줍니다.
%ls data
```

    '전국 평균 분양가격(2013년 9월부터 2015년 8월까지).csv'
    '주택도시보증공사_전국 평균 분양가격(2019년 12월).csv'



```python
# df_first 에 담고 shape로 행과 열의 수를 출력해 봅니다.
df_first = pd.read_csv("data/전국 평균 분양가격(2013년 9월부터 2015년 8월까지).csv", encoding="cp949")
df_first.shape
```




    (17, 22)




```python
# df_first 변수에 담긴 데이터프레임을 head로 미리보기 합니다.
df_first.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역</th>
      <th>2013년12월</th>
      <th>2014년1월</th>
      <th>2014년2월</th>
      <th>2014년3월</th>
      <th>2014년4월</th>
      <th>2014년5월</th>
      <th>2014년6월</th>
      <th>2014년7월</th>
      <th>2014년8월</th>
      <th>...</th>
      <th>2014년11월</th>
      <th>2014년12월</th>
      <th>2015년1월</th>
      <th>2015년2월</th>
      <th>2015년3월</th>
      <th>2015년4월</th>
      <th>2015년5월</th>
      <th>2015년6월</th>
      <th>2015년7월</th>
      <th>2015년8월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>18189</td>
      <td>17925</td>
      <td>17925</td>
      <td>18016</td>
      <td>18098</td>
      <td>19446</td>
      <td>18867</td>
      <td>18742</td>
      <td>19274</td>
      <td>...</td>
      <td>20242</td>
      <td>20269</td>
      <td>20670</td>
      <td>20670</td>
      <td>19415</td>
      <td>18842</td>
      <td>18367</td>
      <td>18374</td>
      <td>18152</td>
      <td>18443</td>
    </tr>
    <tr>
      <th>1</th>
      <td>부산</td>
      <td>8111</td>
      <td>8111</td>
      <td>9078</td>
      <td>8965</td>
      <td>9402</td>
      <td>9501</td>
      <td>9453</td>
      <td>9457</td>
      <td>9411</td>
      <td>...</td>
      <td>9208</td>
      <td>9208</td>
      <td>9204</td>
      <td>9235</td>
      <td>9279</td>
      <td>9327</td>
      <td>9345</td>
      <td>9515</td>
      <td>9559</td>
      <td>9581</td>
    </tr>
    <tr>
      <th>2</th>
      <td>대구</td>
      <td>8080</td>
      <td>8080</td>
      <td>8077</td>
      <td>8101</td>
      <td>8267</td>
      <td>8274</td>
      <td>8360</td>
      <td>8360</td>
      <td>8370</td>
      <td>...</td>
      <td>8439</td>
      <td>8253</td>
      <td>8327</td>
      <td>8416</td>
      <td>8441</td>
      <td>8446</td>
      <td>8568</td>
      <td>8542</td>
      <td>8542</td>
      <td>8795</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인천</td>
      <td>10204</td>
      <td>10204</td>
      <td>10408</td>
      <td>10408</td>
      <td>10000</td>
      <td>9844</td>
      <td>10058</td>
      <td>9974</td>
      <td>9973</td>
      <td>...</td>
      <td>10020</td>
      <td>10020</td>
      <td>10017</td>
      <td>9876</td>
      <td>9876</td>
      <td>9938</td>
      <td>10551</td>
      <td>10443</td>
      <td>10443</td>
      <td>10449</td>
    </tr>
    <tr>
      <th>4</th>
      <td>광주</td>
      <td>6098</td>
      <td>7326</td>
      <td>7611</td>
      <td>7346</td>
      <td>7346</td>
      <td>7523</td>
      <td>7659</td>
      <td>7612</td>
      <td>7622</td>
      <td>...</td>
      <td>7752</td>
      <td>7748</td>
      <td>7752</td>
      <td>7756</td>
      <td>7861</td>
      <td>7914</td>
      <td>7877</td>
      <td>7881</td>
      <td>8089</td>
      <td>8231</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>




```python
# df_first 변수에 담긴 데이터프레임을 tail로 미리보기 합니다.
df_first.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역</th>
      <th>2013년12월</th>
      <th>2014년1월</th>
      <th>2014년2월</th>
      <th>2014년3월</th>
      <th>2014년4월</th>
      <th>2014년5월</th>
      <th>2014년6월</th>
      <th>2014년7월</th>
      <th>2014년8월</th>
      <th>...</th>
      <th>2014년11월</th>
      <th>2014년12월</th>
      <th>2015년1월</th>
      <th>2015년2월</th>
      <th>2015년3월</th>
      <th>2015년4월</th>
      <th>2015년5월</th>
      <th>2015년6월</th>
      <th>2015년7월</th>
      <th>2015년8월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>전북</td>
      <td>6282</td>
      <td>6281</td>
      <td>5946</td>
      <td>5966</td>
      <td>6277</td>
      <td>6306</td>
      <td>6351</td>
      <td>6319</td>
      <td>6436</td>
      <td>...</td>
      <td>6583</td>
      <td>6583</td>
      <td>6583</td>
      <td>6583</td>
      <td>6542</td>
      <td>6551</td>
      <td>6556</td>
      <td>6601</td>
      <td>6750</td>
      <td>6580</td>
    </tr>
    <tr>
      <th>13</th>
      <td>전남</td>
      <td>5678</td>
      <td>5678</td>
      <td>5678</td>
      <td>5696</td>
      <td>5736</td>
      <td>5656</td>
      <td>5609</td>
      <td>5780</td>
      <td>5685</td>
      <td>...</td>
      <td>5768</td>
      <td>5784</td>
      <td>5784</td>
      <td>5833</td>
      <td>5825</td>
      <td>5940</td>
      <td>6050</td>
      <td>6243</td>
      <td>6286</td>
      <td>6289</td>
    </tr>
    <tr>
      <th>14</th>
      <td>경북</td>
      <td>6168</td>
      <td>6168</td>
      <td>6234</td>
      <td>6317</td>
      <td>6412</td>
      <td>6409</td>
      <td>6554</td>
      <td>6556</td>
      <td>6563</td>
      <td>...</td>
      <td>6881</td>
      <td>6989</td>
      <td>6992</td>
      <td>6953</td>
      <td>6997</td>
      <td>7006</td>
      <td>6966</td>
      <td>6887</td>
      <td>7035</td>
      <td>7037</td>
    </tr>
    <tr>
      <th>15</th>
      <td>경남</td>
      <td>6473</td>
      <td>6485</td>
      <td>6502</td>
      <td>6610</td>
      <td>6599</td>
      <td>6610</td>
      <td>6615</td>
      <td>6613</td>
      <td>6606</td>
      <td>...</td>
      <td>7125</td>
      <td>7332</td>
      <td>7592</td>
      <td>7588</td>
      <td>7668</td>
      <td>7683</td>
      <td>7717</td>
      <td>7715</td>
      <td>7723</td>
      <td>7665</td>
    </tr>
    <tr>
      <th>16</th>
      <td>제주</td>
      <td>7674</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7914</td>
      <td>7914</td>
      <td>7914</td>
      <td>...</td>
      <td>7724</td>
      <td>7739</td>
      <td>7739</td>
      <td>7739</td>
      <td>7826</td>
      <td>7285</td>
      <td>7285</td>
      <td>7343</td>
      <td>7343</td>
      <td>7343</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



### 데이터 요약하기


```python
# info 로 요약합니다.
df_last.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 5 columns):
     #   Column   Non-Null Count  Dtype 
    ---  ------   --------------  ----- 
     0   지역명      4335 non-null   object
     1   규모구분     4335 non-null   object
     2   연도       4335 non-null   int64 
     3   월        4335 non-null   int64 
     4   분양가격(㎡)  4058 non-null   object
    dtypes: int64(2), object(3)
    memory usage: 169.5+ KB


### 결측치 보기

isnull 혹은 isna 를 통해 데이터가 비어있는지를 확인할 수 있습니다.
결측치는 True로 표시되는데, True == 1 이기 때문에 이 값을 다 더해주면 결측치의 수가 됩니다.


```python
# isnull 을 통해 결측치를 봅니다.
df_last.isnull()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4330</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4331</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4332</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4333</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4334</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>4335 rows × 5 columns</p>
</div>




```python
# isnull 을 통해 결측치를 구합니다.
df_last.isnull().sum()
```




    지역명          0
    규모구분         0
    연도           0
    월            0
    분양가격(㎡)    277
    dtype: int64




```python
# isna 를 통해 결측치를 구합니다.
df_last.isna().sum()
```




    지역명          0
    규모구분         0
    연도           0
    월            0
    분양가격(㎡)    277
    dtype: int64



### 데이터 타입 변경
분양가격이 object(문자) 타입으로 되어 있습니다. 문자열 타입을 계산할 수 없기 때문에 수치 데이터로 변경해 줍니다. 결측치가 섞여 있을 때 변환이 제대로 되지 않습니다. 그래서 pd.to_numeric 을 통해 데이터의 타입을 변경합니다.


```python
df_last["분양가격"] = pd.to_numeric(df_last["분양가격(㎡)"], errors='coerce')
df_last["분양가격"].mean()
```




    3238.128632802628



### 평당분양가격 구하기
공공데이터포털에 올라와 있는 2013년부터의 데이터는 평당분양가격 기준으로 되어 있습니다.
분양가격을 평당기준으로 보기위해 3.3을 곱해서 "평당분양가격" 컬럼을 만들어 추가해 줍니다.


```python
df_last["평당분양가격"] = df_last["분양가격"] * 3.3
df_last
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>전체</td>
      <td>2015</td>
      <td>10</td>
      <td>5841</td>
      <td>5841.0</td>
      <td>19275.3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울</td>
      <td>전용면적 60㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5652</td>
      <td>5652.0</td>
      <td>18651.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울</td>
      <td>전용면적 60㎡초과 85㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5882</td>
      <td>5882.0</td>
      <td>19410.6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울</td>
      <td>전용면적 85㎡초과 102㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5721</td>
      <td>5721.0</td>
      <td>18879.3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>서울</td>
      <td>전용면적 102㎡초과</td>
      <td>2015</td>
      <td>10</td>
      <td>5879</td>
      <td>5879.0</td>
      <td>19400.7</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4330</th>
      <td>제주</td>
      <td>전체</td>
      <td>2019</td>
      <td>12</td>
      <td>3882</td>
      <td>3882.0</td>
      <td>12810.6</td>
    </tr>
    <tr>
      <th>4331</th>
      <td>제주</td>
      <td>전용면적 60㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4332</th>
      <td>제주</td>
      <td>전용면적 60㎡초과 85㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>3898</td>
      <td>3898.0</td>
      <td>12863.4</td>
    </tr>
    <tr>
      <th>4333</th>
      <td>제주</td>
      <td>전용면적 85㎡초과 102㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4334</th>
      <td>제주</td>
      <td>전용면적 102㎡초과</td>
      <td>2019</td>
      <td>12</td>
      <td>3601</td>
      <td>3601.0</td>
      <td>11883.3</td>
    </tr>
  </tbody>
</table>
<p>4335 rows × 7 columns</p>
</div>



### 분양가격 요약하기


```python
# info를 통해 분양가격을 봅니다.
df_last.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 7 columns):
     #   Column   Non-Null Count  Dtype  
    ---  ------   --------------  -----  
     0   지역명      4335 non-null   object 
     1   규모구분     4335 non-null   object 
     2   연도       4335 non-null   int64  
     3   월        4335 non-null   int64  
     4   분양가격(㎡)  4058 non-null   object 
     5   분양가격     3957 non-null   float64
     6   평당분양가격   3957 non-null   float64
    dtypes: float64(2), int64(2), object(3)
    memory usage: 237.2+ KB



```python
# 변경 전 컬럼인 분양가격(㎡) 컬럼을 요약합니다.
df_last["분양가격(㎡)"].describe()
```




    count     4058
    unique    1753
    top       2221
    freq        17
    Name: 분양가격(㎡), dtype: object




```python
# 수치데이터로 변경된 분양가격 컬럼을 요약합니다.

df_last["분양가격"].describe()
```




    count     3957.000000
    mean      3238.128633
    std       1264.309933
    min       1868.000000
    25%       2441.000000
    50%       2874.000000
    75%       3561.000000
    max      12728.000000
    Name: 분양가격, dtype: float64



### 규모구분을 전용면적 컬럼으로 변경
규모구분 컬럼은 전용면적에 대한 내용이 있습니다. 전용면적이라는 문구가 공통적으로 들어가고 규모구분보다는 전용면적이 좀 더 직관적이기 때문에 전용면적이라는 컬럼을 새로 만들어주고 기존 규모구분의 값에서 전용면적, 초과, 이하 등의 문구를 빼고 간결하게 만들어 봅니다.

이 때 str 의 replace 기능을 사용해서 예를들면 "전용면적 60㎡초과 85㎡이하"라면 "60㎡~85㎡" 로 변경해 줍니다.

* pandas 의 string-handling 기능을 좀 더 보고 싶다면 :
https://pandas.pydata.org/pandas-docs/stable/reference/series.html#string-handling


```python
# 규모구분의 unique 값 보기

df_last["규모구분"].unique()
```




    array(['전체', '전용면적 60㎡이하', '전용면적 60㎡초과 85㎡이하', '전용면적 85㎡초과 102㎡이하',
           '전용면적 102㎡초과'], dtype=object)




```python
# 규모구분을 전용면적으로 변경하기
df_last["전용면적"] = df_last["규모구분"].str.replace("전용면적","").str.replace("초과","~").str.replace("이하","").str.replace(" ","").str.strip()
# df_last["전용면적"] = df_last["전용면적"].str.replace("초과","~")
# df_last["전용면적"] = df_last["전용면적"].str.replace("이하","")
# df_last["전용면적"] = df_last["전용면적"].str.replace(" ","").str.strip()

df_last["전용면적"]
```




    0             전체
    1            60㎡
    2        60㎡~85㎡
    3       85㎡~102㎡
    4          102㎡~
              ...   
    4330          전체
    4331         60㎡
    4332     60㎡~85㎡
    4333    85㎡~102㎡
    4334       102㎡~
    Name: 전용면적, Length: 4335, dtype: object



### 필요없는 컬럼 제거하기
drop을 통해 전처리 해준 컬럼을 제거합니다. pandas의 데이터프레임과 관련된 메소드에는 axis 옵션이 필요할 때가 있는데 행과 열중 어떤 기준으로 처리를 할 것인지를 의미합니다. 보통 기본적으로 0으로 되어 있고 행을 기준으로 처리함을 의미합니다. 메모리 사용량이 줄어들었는지 확인합니다.


```python
# info로 정보 보기
df_last.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 8 columns):
     #   Column   Non-Null Count  Dtype  
    ---  ------   --------------  -----  
     0   지역명      4335 non-null   object 
     1   규모구분     4335 non-null   object 
     2   연도       4335 non-null   int64  
     3   월        4335 non-null   int64  
     4   분양가격(㎡)  4058 non-null   object 
     5   분양가격     3957 non-null   float64
     6   평당분양가격   3957 non-null   float64
     7   전용면적     4335 non-null   object 
    dtypes: float64(2), int64(2), object(4)
    memory usage: 271.1+ KB



```python
# drop 사용시 axis에 유의 합니다.
# axis 0:행, 1:열
df_last = df_last.drop(["규모구분" ,"분양가격(㎡)"], axis=1)

```


```python
# 제거가 잘 되었는지 확인 합니다.

df_last.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
      <th>전용면적</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>5841.0</td>
      <td>19275.3</td>
      <td>전체</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 컬럼 제거를 통해 메모리 사용량이 줄어들었는지 확인합니다.
df_last.info()

```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 6 columns):
     #   Column  Non-Null Count  Dtype  
    ---  ------  --------------  -----  
     0   지역명     4335 non-null   object 
     1   연도      4335 non-null   int64  
     2   월       4335 non-null   int64  
     3   분양가격    3957 non-null   float64
     4   평당분양가격  3957 non-null   float64
     5   전용면적    4335 non-null   object 
    dtypes: float64(2), int64(2), object(2)
    memory usage: 203.3+ KB


## groupby 로 데이터 집계하기
groupby 를 통해 데이터를 그룹화해서 연산을 해봅니다.


```python
# 지역명으로 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.
# df.groupby(["인덱스로 사용할 컬럼명"])["계산할 컬럼 값"].연산()

df_last.groupby(["지역명"])["평당분양가격"].mean()
```




    지역명
    강원     7890.750000
    경기    13356.895200
    경남     9268.778138
    경북     8376.536515
    광주     9951.535821
    대구    11980.895455
    대전    10253.333333
    부산    12087.121200
    서울    23599.976400
    세종     9796.516456
    울산    10014.902013
    인천    11915.320732
    전남     7565.316532
    전북     7724.235484
    제주    11241.276712
    충남     8233.651883
    충북     7634.655600
    Name: 평당분양가격, dtype: float64




```python
# 전용면적으로 분양가격의 평균을 구합니다.
df_last
df_last.groupby(["전용면적"])["평당분양가격"].mean()
```




    전용면적
    102㎡~       11517.705634
    60㎡         10375.137421
    60㎡~85㎡     10271.040071
    85㎡~102㎡    11097.599573
    전체          10276.086207
    Name: 평당분양가격, dtype: float64




```python
# 지역명, 전용면적으로 평당분양가격의 평균을 구합니다.

df_last.groupby(["전용면적","지역명"])["평당분양가격"].mean().unstack().round()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>8311.0</td>
      <td>14772.0</td>
      <td>10358.0</td>
      <td>9157.0</td>
      <td>11042.0</td>
      <td>13087.0</td>
      <td>14877.0</td>
      <td>13208.0</td>
      <td>23446.0</td>
      <td>10107.0</td>
      <td>9974.0</td>
      <td>14362.0</td>
      <td>8168.0</td>
      <td>8194.0</td>
      <td>10523.0</td>
      <td>8689.0</td>
      <td>8195.0</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>7567.0</td>
      <td>13252.0</td>
      <td>8689.0</td>
      <td>7883.0</td>
      <td>9431.0</td>
      <td>11992.0</td>
      <td>9176.0</td>
      <td>11354.0</td>
      <td>23213.0</td>
      <td>9324.0</td>
      <td>9202.0</td>
      <td>11241.0</td>
      <td>7210.0</td>
      <td>7610.0</td>
      <td>14022.0</td>
      <td>7911.0</td>
      <td>7103.0</td>
    </tr>
    <tr>
      <th>60㎡~85㎡</th>
      <td>7486.0</td>
      <td>12524.0</td>
      <td>8619.0</td>
      <td>8061.0</td>
      <td>9911.0</td>
      <td>11779.0</td>
      <td>9711.0</td>
      <td>11865.0</td>
      <td>22787.0</td>
      <td>9775.0</td>
      <td>10503.0</td>
      <td>11384.0</td>
      <td>7269.0</td>
      <td>7271.0</td>
      <td>10621.0</td>
      <td>7819.0</td>
      <td>7264.0</td>
    </tr>
    <tr>
      <th>85㎡~102㎡</th>
      <td>8750.0</td>
      <td>13678.0</td>
      <td>10018.0</td>
      <td>8774.0</td>
      <td>9296.0</td>
      <td>11141.0</td>
      <td>9037.0</td>
      <td>12073.0</td>
      <td>25944.0</td>
      <td>9848.0</td>
      <td>8861.0</td>
      <td>11528.0</td>
      <td>7909.0</td>
      <td>8276.0</td>
      <td>10709.0</td>
      <td>9120.0</td>
      <td>8391.0</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>7478.0</td>
      <td>12560.0</td>
      <td>8659.0</td>
      <td>8079.0</td>
      <td>9904.0</td>
      <td>11771.0</td>
      <td>9786.0</td>
      <td>11936.0</td>
      <td>22610.0</td>
      <td>9805.0</td>
      <td>10493.0</td>
      <td>11257.0</td>
      <td>7284.0</td>
      <td>7293.0</td>
      <td>10785.0</td>
      <td>7815.0</td>
      <td>7219.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 연도, 지역명으로 평당분양가격의 평균을 구합니다.
g = df_last.groupby(["연도","지역명"])["평당분양가격"].mean()
g
# g.unstack().round().T
```




    연도    지역명
    2015  강원      7188.060000
          경기     11060.940000
          경남      8459.220000
          경북      7464.160000
          광주      7916.700000
                     ...     
    2019  전남      8219.275862
          전북      8532.260000
          제주     11828.469231
          충남      8748.840000
          충북      7970.875000
    Name: 평당분양가격, Length: 85, dtype: float64



## pivot table 로 데이터 집계하기
* groupby 로 했던 작업을 pivot_table로 똑같이 해봅니다.


```python
# 지역명을 index 로 평당분양가격 을 values 로 구합니다.
pd.pivot_table(df_last , index = ["지역명"], values = ["평당분양가격"], aggfunc="mean")

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>7890.750000</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>13356.895200</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>9268.778138</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>8376.536515</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>9951.535821</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>11980.895455</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>10253.333333</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>12087.121200</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>23599.976400</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>9796.516456</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>10014.902013</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>11915.320732</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>7565.316532</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>7724.235484</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>11241.276712</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>8233.651883</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>7634.655600</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_last.groupby(["전용면적"])["평당분양가격"].mean()

```




    전용면적
    102㎡~       11517.705634
    60㎡         10375.137421
    60㎡~85㎡     10271.040071
    85㎡~102㎡    11097.599573
    전체          10276.086207
    Name: 평당분양가격, dtype: float64




```python
pd.pivot_table(df_last , index="전용면적" , values="평당분양가격")
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>11517.705634</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>10375.137421</td>
    </tr>
    <tr>
      <th>60㎡~85㎡</th>
      <td>10271.040071</td>
    </tr>
    <tr>
      <th>85㎡~102㎡</th>
      <td>11097.599573</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>10276.086207</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 전용면적을 index 로 평당분양가격 을 values 로 구합니다.


```


```python
# 지역명, 전용면적으로 평당분양가격의 평균을 구합니다.
# df_last.groupby(["전용면적", "지역명"])["평당분양가격"].mean().unstack().round()
df_last.pivot_table(index=["전용면적"], columns=[ "지역명"] , values="평당분양가격").round()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>8311.0</td>
      <td>14772.0</td>
      <td>10358.0</td>
      <td>9157.0</td>
      <td>11042.0</td>
      <td>13087.0</td>
      <td>14877.0</td>
      <td>13208.0</td>
      <td>23446.0</td>
      <td>10107.0</td>
      <td>9974.0</td>
      <td>14362.0</td>
      <td>8168.0</td>
      <td>8194.0</td>
      <td>10523.0</td>
      <td>8689.0</td>
      <td>8195.0</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>7567.0</td>
      <td>13252.0</td>
      <td>8689.0</td>
      <td>7883.0</td>
      <td>9431.0</td>
      <td>11992.0</td>
      <td>9176.0</td>
      <td>11354.0</td>
      <td>23213.0</td>
      <td>9324.0</td>
      <td>9202.0</td>
      <td>11241.0</td>
      <td>7210.0</td>
      <td>7610.0</td>
      <td>14022.0</td>
      <td>7911.0</td>
      <td>7103.0</td>
    </tr>
    <tr>
      <th>60㎡~85㎡</th>
      <td>7486.0</td>
      <td>12524.0</td>
      <td>8619.0</td>
      <td>8061.0</td>
      <td>9911.0</td>
      <td>11779.0</td>
      <td>9711.0</td>
      <td>11865.0</td>
      <td>22787.0</td>
      <td>9775.0</td>
      <td>10503.0</td>
      <td>11384.0</td>
      <td>7269.0</td>
      <td>7271.0</td>
      <td>10621.0</td>
      <td>7819.0</td>
      <td>7264.0</td>
    </tr>
    <tr>
      <th>85㎡~102㎡</th>
      <td>8750.0</td>
      <td>13678.0</td>
      <td>10018.0</td>
      <td>8774.0</td>
      <td>9296.0</td>
      <td>11141.0</td>
      <td>9037.0</td>
      <td>12073.0</td>
      <td>25944.0</td>
      <td>9848.0</td>
      <td>8861.0</td>
      <td>11528.0</td>
      <td>7909.0</td>
      <td>8276.0</td>
      <td>10709.0</td>
      <td>9120.0</td>
      <td>8391.0</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>7478.0</td>
      <td>12560.0</td>
      <td>8659.0</td>
      <td>8079.0</td>
      <td>9904.0</td>
      <td>11771.0</td>
      <td>9786.0</td>
      <td>11936.0</td>
      <td>22610.0</td>
      <td>9805.0</td>
      <td>10493.0</td>
      <td>11257.0</td>
      <td>7284.0</td>
      <td>7293.0</td>
      <td>10785.0</td>
      <td>7815.0</td>
      <td>7219.0</td>
    </tr>
  </tbody>
</table>
</div>




```python

```


```python
# 연도, 지역명으로 평당분양가격의 평균을 구합니다.
# g = df_last.groupby(["연도", "지역명"])["평당분양가격"].mean()
p = df_last.pivot_table(index=["연도", "지역명"] , values="평당분양가격").round()
p.loc[2017]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>7274.0</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>12305.0</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>8787.0</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>8281.0</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>9614.0</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>12207.0</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>9957.0</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>11561.0</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>21831.0</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>9133.0</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>10667.0</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>11641.0</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>7373.0</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>7399.0</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>12567.0</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>8198.0</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>7473.0</td>
    </tr>
  </tbody>
</table>
</div>



## 최근 데이터 시각화 하기
### 데이터시각화를 위한 폰트설정
한글폰트 사용을 위해 matplotlib의 pyplot을 plt라는 별칭으로 불러옵니다.


```python
from matplotlib import font_manager
for i in font_manager.fontManager.ttflist:
    if 'Nanum' in i.name:
        print(i.name, i.fname)
```

    NanumGothic /home/atcis/anaconda3/lib/python3.8/site-packages/matplotlib/mpl-data/fonts/ttf/NanumGothic.ttf
    NanumMyeongjo Eco /usr/share/fonts/TTF/NanumMyeongjoEcoR.ttf
    NanumGothic /usr/share/fonts/TTF/NanumGothicLight.ttf
    NanumGothic Eco /usr/share/fonts/TTF/NanumGothicEcoR.ttf
    NanumBarunGothic /usr/share/fonts/TTF/NanumBarunGothicBold.ttf
    NanumSquareRound /usr/share/fonts/TTF/NanumSquareRoundL.ttf
    NanumBarunGothic YetHangul /usr/share/fonts/TTF/NanumBarunGothic-YetHangul.ttf
    NanumBarunGothic /usr/share/fonts/TTF/NanumBarunGothicLight.ttf
    NanumBarunGothic /usr/share/fonts/TTF/NanumBarunGothicUltraLight.ttf
    NanumGothic /usr/share/fonts/TTF/NanumGothic.ttf
    NanumMyeongjo /usr/share/fonts/TTF/NanumMyeongjo.ttf
    NanumSquare_ac /usr/share/fonts/TTF/NanumSquare_acL.ttf
    NanumMyeongjo /usr/share/fonts/TTF/NanumMyeongjoExtraBold.ttf
    NanumSquareRound /usr/share/fonts/TTF/NanumSquareRoundB.ttf
    NanumGothic /usr/share/fonts/TTF/NanumGothicExtraBold.ttf
    NanumMyeongjo YetHangul /usr/share/fonts/TTF/NanumMyeongjo-YetHangul.ttf
    NanumSquare_ac /usr/share/fonts/TTF/NanumSquare_acEB.ttf
    NanumGothicCoding /usr/share/fonts/TTF/NanumGothicCoding.ttf
    Nanum Pen Script /usr/share/fonts/TTF/NanumPen.ttf
    NanumSquare_ac /usr/share/fonts/TTF/NanumSquare_acB.ttf
    NanumMyeongjo /usr/share/fonts/TTF/NanumMyeongjoBold.ttf
    NanumSquare /usr/share/fonts/TTF/NanumSquareL.ttf
    NanumSquare /usr/share/fonts/TTF/NanumSquareB.ttf
    NanumSquareRound /usr/share/fonts/TTF/NanumSquareRoundEB.ttf
    NanumBarunpen /usr/share/fonts/TTF/NanumBarunpenB.ttf
    NanumSquare /usr/share/fonts/TTF/NanumSquareR.ttf
    NanumGothicCoding /usr/share/fonts/TTF/NanumGothicCodingBold.ttf
    NanumSquare_ac /usr/share/fonts/TTF/NanumSquare_acR.ttf
    Nanum Brush Script /usr/share/fonts/TTF/NanumBrush.ttf
    NanumSquare /usr/share/fonts/TTF/NanumSquareEB.ttf
    NanumBarunGothic /usr/share/fonts/TTF/NanumBarunGothic.ttf
    NanumGothic /usr/share/fonts/TTF/NanumGothicBold.ttf
    NanumSquareRound /usr/share/fonts/TTF/NanumSquareRoundR.ttf
    NanumBarunpen /usr/share/fonts/TTF/NanumBarunpenR.ttf



```python
import matplotlib.font_manager as fm

font_location = '/usr/share/fonts/TTF/NanumGothic.ttf'  
                    # ex - 'C:/asiahead4.ttf'
font_name = fm.FontProperties(fname = font_location).get_name()
font_name
```




    'NanumGothic'




```python
import matplotlib.pyplot as plt

# plt.rc("font", family="Malgun Gothic")
# plt.rc("font", family="AppleGothic")
# plt.rc("font", family="NanumGothic")
plt.rc("font", family=font_name)
```

### Pandas로 시각화 하기 - 선그래프와 막대그래프
pandas의 plot을 활용하면 다양한 그래프를 그릴 수 있습니다.
seaborn을 사용했을 때보다 pandas를 사용해서 시각화를 할 때의 장점은 미리 계산을 하고 그리기 때문에 속도가 좀 더 빠릅니다.


```python
# 지역명으로 분양가격의 평균을 구하고 선그래프로 시각화 합니다.
g =df_last.groupby(["지역명"])["평당분양가격"].mean().sort_values(ascending=False)
g.plot()
```




    <AxesSubplot:xlabel='지역명'>




    
![svg](output_53_1.svg)
    



```python
# 지역명으로 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.

g.plot.bar(rot=0, figsize=(10, 3))
```




    <AxesSubplot:xlabel='지역명'>




    
![svg](output_54_1.svg)
    


전용면적별 분양가격의 평균값을 구하고 그래프로 그려봅니다.


```python
# 전용면적으로 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.

df_last.groupby(["전용면적"])["평당분양가격"].mean().plot.bar()
```




    <AxesSubplot:xlabel='전용면적'>




    
![svg](output_56_1.svg)
    



```python
# 연도별 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.
df_last.groupby(["연도"])["평당분양가격"].mean().plot.bar(rot=0, figsize=(10, 3))

```




    <AxesSubplot:xlabel='연도'>




    
![svg](output_57_1.svg)
    


### box-and-whisker plot | diagram

* https://pandas.pydata.org/pandas-docs/stable/user_guide/visualization.html
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.boxplot.html

* [상자 수염 그림 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EC%83%81%EC%9E%90_%EC%88%98%EC%97%BC_%EA%B7%B8%EB%A6%BC)
* 가공하지 않은 자료 그대로를 이용하여 그린 것이 아니라, 자료로부터 얻어낸 통계량인 5가지 요약 수치로 그린다.
* 5가지 요약 수치란 기술통계학에서 자료의 정보를 알려주는 아래의 다섯 가지 수치를 의미한다.


1. 최솟값
1. 제 1사분위수
1. 제 2사분위수( ), 즉 중앙값
1. 제 3 사분위 수( )
1. 최댓값

* Box plot 이해하기 : 
    * [박스 플롯에 대하여 :: -[|]- Box and Whisker](https://boxnwhis.kr/2019/02/19/boxplot.html)
    * [Understanding Boxplots – Towards Data Science](https://towardsdatascience.com/understanding-boxplots-5e2df7bcbd51)


```python
# index를 월, columns 를 연도로 구하고 평당분양가격 으로 pivot_table 을 구하고 상자수염그림을 그립니다.
df_last.pivot_table(index="월",columns="연도", values="평당분양가격").plot.box()


```




    <AxesSubplot:>




    
![svg](output_59_1.svg)
    



```python
# columns 에 "연도", "전용면적"을 추가해서 pivot_table 을 만들고 시각화 합니다.
p=df_last.pivot_table(index="월",columns=["연도","전용면적"], values="평당분양가격")
p.plot.box(figsize=(15,3), rot=30)

```




    <AxesSubplot:>




    
![svg](output_60_1.svg)
    



```python
p=df_last.pivot_table(index="연도",columns=["지역명"], values="평당분양가격")
p.plot(figsize=(15,3), rot=30)

```




    <AxesSubplot:xlabel='연도'>




    
![svg](output_61_1.svg)
    



```python
# index를 월, columns 를 지역명으로 구하고 평당분양가격 으로 pivot_table 을 구하고 선그래프를 그립니다.

p=df_last.pivot_table(index="월",columns=["지역명"], values="평당분양가격")
p.plot(figsize=(15,3), rot=30)

```




    <AxesSubplot:xlabel='월'>




    
![svg](output_62_1.svg)
    


### Seaborn 으로 시각화 해보기


```python
# 라이브러리 로드하기
import seaborn as sns

%matplotlib inline
```


```python
# barplot으로 지역별 평당분양가격을 그려봅니다.
plt.figure(figsize=(10,3))

# ci 신뢰구간 설정
sns.barplot(data=df_last , x="지역명", y="평당분양가격")

```




    <AxesSubplot:xlabel='지역명', ylabel='평당분양가격'>




    
![svg](output_65_1.svg)
    



```python
# barplot으로 연도별 평당분양가격을 그려봅니다.

sns.barplot(data=df_last , x="연도", y="평당분양가격")
```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_66_1.svg)
    



```python
# catplot 으로 서브플롯 그리기
sns.catplot(data=df_last , x="연도", y="평당분양가격", kind="bar", col="지역명", col_wrap=4)
```




    <seaborn.axisgrid.FacetGrid at 0x7f9a3b386b50>




    
![svg](output_67_1.svg)
    


https://stackoverflow.com/questions/30490740/move-legend-outside-figure-in-seaborn-tsplot


```python
# lineplot으로 연도별 평당분양가격을 그려봅니다.
# hue 옵션을 통해 지역별로 다르게 표시해 봅니다.

plt.figure(figsize=(10, 5))
sns.lineplot(data=df_last , x="연도", y="평당분양가격", hue="지역명")
plt.legend(bbox_to_anchor=(1.02, 1), loc=2, borderaxespad=0.)

```




    <matplotlib.legend.Legend at 0x7f9a397e1d00>




    
![svg](output_69_1.svg)
    



```python
# relplot 으로 서브플롯 그리기
# sns.relplot?
sns.relplot(data=df_last, x="연도", y="평당분양가격", 
    kind="line" , hue="지역명", col="지역명", col_wrap=4, ci=None)

```




    <seaborn.axisgrid.FacetGrid at 0x7f9a39764550>




    
![svg](output_70_1.svg)
    


### boxplot과 violinplot


```python
# 연도별 평당분양가격을 boxplot으로 그려봅니다.
# 최솟값
# 제 1사분위수
# 제 2사분위수( ), 즉 중앙값
# 제 3 사분위 수( )
# 최댓값
sns.boxplot(data=df_last , x="연도", y="평당분양가격")

```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_72_1.svg)
    



```python
# hue옵션을 주어 전용면적별로 다르게 표시해 봅니다.
plt.figure(figsize=(12,3))
sns.boxplot(data=df_last , x="연도", y="평당분양가격"
    , hue="전용면적")


```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_73_1.svg)
    



```python
# 연도별 평당분양가격을 violinplot으로 그려봅니다.
sns.violinplot(data=df_last , x="연도", y="평당분양가격"
    )

```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_74_1.svg)
    


### lmplot과 swarmplot 


```python
# 연도별 평당분양가격을 lmplot으로 그려봅니다. 
# hue 옵션으로 전용면적을 표현해 봅니다.
sns.lmplot(data=df_last , x="연도" , y="평당분양가격", hue="전용면적", col="전용면적",col_wrap=3)


```




    <seaborn.axisgrid.FacetGrid at 0x7f9a38db5640>




    
![svg](output_76_1.svg)
    



```python
# 연도별 평당분양가격을 swarmplot 으로 그려봅니다. 
# swarmplot은 범주형(카테고리) 데이터의 산점도를 표현하기에 적합합니다.
plt.figure(figsize=(15,3))
sns.swarmplot(data=df_last , x="연도", y="평당분양가격", hue="전용면적" )

```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_77_1.svg)
    


### 이상치 보기


```python
# 평당분양가격의 최대값을 구해서 max_price 라는 변수에 담습니다.
df_last["평당분양가격"].describe()
```




    count     3957.000000
    mean     10685.824488
    std       4172.222780
    min       6164.400000
    25%       8055.300000
    50%       9484.200000
    75%      11751.300000
    max      42002.400000
    Name: 평당분양가격, dtype: float64




```python
max_price =  df_last["평당분양가격"].max()
max_price 
```




    42002.399999999994




```python
# 서울의 평당분양가격이 특히 높은 데이터가 있습니다. 해당 데이터를 가져옵니다.
df_last[df_last["평당분양가격"] == max_price]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
      <th>전용면적</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3743</th>
      <td>서울</td>
      <td>2019</td>
      <td>6</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~102㎡</td>
    </tr>
    <tr>
      <th>3828</th>
      <td>서울</td>
      <td>2019</td>
      <td>7</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~102㎡</td>
    </tr>
    <tr>
      <th>3913</th>
      <td>서울</td>
      <td>2019</td>
      <td>8</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~102㎡</td>
    </tr>
    <tr>
      <th>3998</th>
      <td>서울</td>
      <td>2019</td>
      <td>9</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~102㎡</td>
    </tr>
    <tr>
      <th>4083</th>
      <td>서울</td>
      <td>2019</td>
      <td>10</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~102㎡</td>
    </tr>
    <tr>
      <th>4168</th>
      <td>서울</td>
      <td>2019</td>
      <td>11</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~102㎡</td>
    </tr>
    <tr>
      <th>4253</th>
      <td>서울</td>
      <td>2019</td>
      <td>12</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~102㎡</td>
    </tr>
  </tbody>
</table>
</div>



### 수치데이터 히스토그램 그리기

distplot은 결측치가 있으면 그래프를 그릴 때 오류가 납니다. 
따라서 결측치가 아닌 데이터만 따로 모아서 평당분양가격을 시각화하기 위한 데이터를 만듭니다.
데이터프레임의 .loc를 활용하여 결측치가 없는 데이터에서 평당분양가격만 가져옵니다.


```python

```


```python

```


```python
# 결측치가 없는 데이터에서 평당분양가격만 가져옵니다. 그리고 price라는 변수에 담습니다.
# .loc[행]
# .loc[행, 열]
h = df_last["평당분양가격"].hist(bins = 10)

```


    
![svg](output_86_0.svg)
    



```python
price = df_last.loc[df_last["평당분양가격"].notnull(), "평당분양가격"]
price
```




    0       19275.3
    1       18651.6
    2       19410.6
    3       18879.3
    4       19400.7
             ...   
    4327    10114.5
    4328    10715.1
    4330    12810.6
    4332    12863.4
    4334    11883.3
    Name: 평당분양가격, Length: 3957, dtype: float64




```python
# distplot으로 평당분양가격을 표현해 봅니다.
sns.distplot(price)

```




    <AxesSubplot:xlabel='평당분양가격', ylabel='Density'>




    
![svg](output_88_1.svg)
    



```python
# sns.distplot(price, hist=False, rug=True)
sns.kdeplot(price, cumulative=True)

```




    <AxesSubplot:xlabel='평당분양가격', ylabel='Density'>




    
![svg](output_89_1.svg)
    


* distplot을 산마루 형태의 ridge plot으로 그리기
* https://seaborn.pydata.org/tutorial/axis_grids.html#conditional-small-multiples
* https://seaborn.pydata.org/examples/kde_ridgeplot.html


```python
# subplot 으로 표현해 봅니다.
g = sns.FacetGrid(df_last, row="지역명",height=1.7, aspect=4)
g.map(sns.distplot, "평당분양가격", hist=False, rug=True)


```




    <seaborn.axisgrid.FacetGrid at 0x7f9a38920340>




    
![svg](output_91_1.svg)
    



```python
# pairplot
df_last_notnull = df_last.loc[df_last["평당분양가격"].notnull(),
    ["연도" , "월" , "평당분양가격", "지역명" , "전용면적"]]
sns.pairplot(df_last_notnull, hue="지역명")

```




    <seaborn.axisgrid.PairGrid at 0x7f9a3892adc0>




    
![svg](output_92_1.svg)
    



```python
# 규모구분(전용면적)별로 value_counts를 사용해서 데이터를 집계해 봅니다.
df_last["전용면적"].value_counts()

```




    전체          867
    102㎡~       867
    60㎡~85㎡     867
    60㎡         867
    85㎡~102㎡    867
    Name: 전용면적, dtype: int64



## 2015년 8월 이전 데이터 보기


```python
# 모든 컬럼이 출력되게 설정합니다.
pd.options.display.max_columns = 25
df_first

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역</th>
      <th>2013년12월</th>
      <th>2014년1월</th>
      <th>2014년2월</th>
      <th>2014년3월</th>
      <th>2014년4월</th>
      <th>2014년5월</th>
      <th>2014년6월</th>
      <th>2014년7월</th>
      <th>2014년8월</th>
      <th>2014년9월</th>
      <th>2014년10월</th>
      <th>2014년11월</th>
      <th>2014년12월</th>
      <th>2015년1월</th>
      <th>2015년2월</th>
      <th>2015년3월</th>
      <th>2015년4월</th>
      <th>2015년5월</th>
      <th>2015년6월</th>
      <th>2015년7월</th>
      <th>2015년8월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>18189</td>
      <td>17925</td>
      <td>17925</td>
      <td>18016</td>
      <td>18098</td>
      <td>19446</td>
      <td>18867</td>
      <td>18742</td>
      <td>19274</td>
      <td>19404</td>
      <td>19759</td>
      <td>20242</td>
      <td>20269</td>
      <td>20670</td>
      <td>20670</td>
      <td>19415</td>
      <td>18842</td>
      <td>18367</td>
      <td>18374</td>
      <td>18152</td>
      <td>18443</td>
    </tr>
    <tr>
      <th>1</th>
      <td>부산</td>
      <td>8111</td>
      <td>8111</td>
      <td>9078</td>
      <td>8965</td>
      <td>9402</td>
      <td>9501</td>
      <td>9453</td>
      <td>9457</td>
      <td>9411</td>
      <td>9258</td>
      <td>9110</td>
      <td>9208</td>
      <td>9208</td>
      <td>9204</td>
      <td>9235</td>
      <td>9279</td>
      <td>9327</td>
      <td>9345</td>
      <td>9515</td>
      <td>9559</td>
      <td>9581</td>
    </tr>
    <tr>
      <th>2</th>
      <td>대구</td>
      <td>8080</td>
      <td>8080</td>
      <td>8077</td>
      <td>8101</td>
      <td>8267</td>
      <td>8274</td>
      <td>8360</td>
      <td>8360</td>
      <td>8370</td>
      <td>8449</td>
      <td>8403</td>
      <td>8439</td>
      <td>8253</td>
      <td>8327</td>
      <td>8416</td>
      <td>8441</td>
      <td>8446</td>
      <td>8568</td>
      <td>8542</td>
      <td>8542</td>
      <td>8795</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인천</td>
      <td>10204</td>
      <td>10204</td>
      <td>10408</td>
      <td>10408</td>
      <td>10000</td>
      <td>9844</td>
      <td>10058</td>
      <td>9974</td>
      <td>9973</td>
      <td>9973</td>
      <td>10016</td>
      <td>10020</td>
      <td>10020</td>
      <td>10017</td>
      <td>9876</td>
      <td>9876</td>
      <td>9938</td>
      <td>10551</td>
      <td>10443</td>
      <td>10443</td>
      <td>10449</td>
    </tr>
    <tr>
      <th>4</th>
      <td>광주</td>
      <td>6098</td>
      <td>7326</td>
      <td>7611</td>
      <td>7346</td>
      <td>7346</td>
      <td>7523</td>
      <td>7659</td>
      <td>7612</td>
      <td>7622</td>
      <td>7802</td>
      <td>7707</td>
      <td>7752</td>
      <td>7748</td>
      <td>7752</td>
      <td>7756</td>
      <td>7861</td>
      <td>7914</td>
      <td>7877</td>
      <td>7881</td>
      <td>8089</td>
      <td>8231</td>
    </tr>
    <tr>
      <th>5</th>
      <td>대전</td>
      <td>8321</td>
      <td>8321</td>
      <td>8321</td>
      <td>8341</td>
      <td>8341</td>
      <td>8341</td>
      <td>8333</td>
      <td>8333</td>
      <td>8333</td>
      <td>8048</td>
      <td>8038</td>
      <td>8067</td>
      <td>8067</td>
      <td>8067</td>
      <td>8067</td>
      <td>8067</td>
      <td>8145</td>
      <td>8272</td>
      <td>8079</td>
      <td>8079</td>
      <td>8079</td>
    </tr>
    <tr>
      <th>6</th>
      <td>울산</td>
      <td>8090</td>
      <td>8090</td>
      <td>8090</td>
      <td>8153</td>
      <td>8153</td>
      <td>8153</td>
      <td>8153</td>
      <td>8153</td>
      <td>8493</td>
      <td>8493</td>
      <td>8627</td>
      <td>8891</td>
      <td>8891</td>
      <td>8526</td>
      <td>8526</td>
      <td>8629</td>
      <td>9380</td>
      <td>9192</td>
      <td>9190</td>
      <td>9190</td>
      <td>9215</td>
    </tr>
    <tr>
      <th>7</th>
      <td>경기</td>
      <td>10855</td>
      <td>10855</td>
      <td>10791</td>
      <td>10784</td>
      <td>10876</td>
      <td>10646</td>
      <td>10266</td>
      <td>10124</td>
      <td>10134</td>
      <td>10501</td>
      <td>10397</td>
      <td>10356</td>
      <td>10379</td>
      <td>10391</td>
      <td>10355</td>
      <td>10469</td>
      <td>10684</td>
      <td>10685</td>
      <td>10573</td>
      <td>10518</td>
      <td>10573</td>
    </tr>
    <tr>
      <th>8</th>
      <td>세종</td>
      <td>7601</td>
      <td>7600</td>
      <td>7532</td>
      <td>7814</td>
      <td>7908</td>
      <td>7934</td>
      <td>8067</td>
      <td>8067</td>
      <td>8141</td>
      <td>8282</td>
      <td>8527</td>
      <td>8592</td>
      <td>8560</td>
      <td>8560</td>
      <td>8560</td>
      <td>8555</td>
      <td>8546</td>
      <td>8546</td>
      <td>8671</td>
      <td>8669</td>
      <td>8695</td>
    </tr>
    <tr>
      <th>9</th>
      <td>강원</td>
      <td>6230</td>
      <td>6230</td>
      <td>6230</td>
      <td>6141</td>
      <td>6373</td>
      <td>6350</td>
      <td>6350</td>
      <td>6268</td>
      <td>6268</td>
      <td>6419</td>
      <td>6631</td>
      <td>6365</td>
      <td>6365</td>
      <td>6348</td>
      <td>6350</td>
      <td>6182</td>
      <td>6924</td>
      <td>6846</td>
      <td>6986</td>
      <td>7019</td>
      <td>7008</td>
    </tr>
    <tr>
      <th>10</th>
      <td>충북</td>
      <td>6589</td>
      <td>6589</td>
      <td>6611</td>
      <td>6625</td>
      <td>6678</td>
      <td>6598</td>
      <td>6587</td>
      <td>6586</td>
      <td>6586</td>
      <td>6584</td>
      <td>6529</td>
      <td>6724</td>
      <td>6743</td>
      <td>6749</td>
      <td>6747</td>
      <td>6783</td>
      <td>6790</td>
      <td>6805</td>
      <td>6682</td>
      <td>6601</td>
      <td>6603</td>
    </tr>
    <tr>
      <th>11</th>
      <td>충남</td>
      <td>6365</td>
      <td>6365</td>
      <td>6379</td>
      <td>6287</td>
      <td>6552</td>
      <td>6591</td>
      <td>6644</td>
      <td>6805</td>
      <td>6914</td>
      <td>6882</td>
      <td>6831</td>
      <td>6940</td>
      <td>6989</td>
      <td>6976</td>
      <td>6980</td>
      <td>7161</td>
      <td>7017</td>
      <td>6975</td>
      <td>6939</td>
      <td>6935</td>
      <td>6942</td>
    </tr>
    <tr>
      <th>12</th>
      <td>전북</td>
      <td>6282</td>
      <td>6281</td>
      <td>5946</td>
      <td>5966</td>
      <td>6277</td>
      <td>6306</td>
      <td>6351</td>
      <td>6319</td>
      <td>6436</td>
      <td>6719</td>
      <td>6581</td>
      <td>6583</td>
      <td>6583</td>
      <td>6583</td>
      <td>6583</td>
      <td>6542</td>
      <td>6551</td>
      <td>6556</td>
      <td>6601</td>
      <td>6750</td>
      <td>6580</td>
    </tr>
    <tr>
      <th>13</th>
      <td>전남</td>
      <td>5678</td>
      <td>5678</td>
      <td>5678</td>
      <td>5696</td>
      <td>5736</td>
      <td>5656</td>
      <td>5609</td>
      <td>5780</td>
      <td>5685</td>
      <td>5804</td>
      <td>5753</td>
      <td>5768</td>
      <td>5784</td>
      <td>5784</td>
      <td>5833</td>
      <td>5825</td>
      <td>5940</td>
      <td>6050</td>
      <td>6243</td>
      <td>6286</td>
      <td>6289</td>
    </tr>
    <tr>
      <th>14</th>
      <td>경북</td>
      <td>6168</td>
      <td>6168</td>
      <td>6234</td>
      <td>6317</td>
      <td>6412</td>
      <td>6409</td>
      <td>6554</td>
      <td>6556</td>
      <td>6563</td>
      <td>6577</td>
      <td>6778</td>
      <td>6881</td>
      <td>6989</td>
      <td>6992</td>
      <td>6953</td>
      <td>6997</td>
      <td>7006</td>
      <td>6966</td>
      <td>6887</td>
      <td>7035</td>
      <td>7037</td>
    </tr>
    <tr>
      <th>15</th>
      <td>경남</td>
      <td>6473</td>
      <td>6485</td>
      <td>6502</td>
      <td>6610</td>
      <td>6599</td>
      <td>6610</td>
      <td>6615</td>
      <td>6613</td>
      <td>6606</td>
      <td>6767</td>
      <td>6881</td>
      <td>7125</td>
      <td>7332</td>
      <td>7592</td>
      <td>7588</td>
      <td>7668</td>
      <td>7683</td>
      <td>7717</td>
      <td>7715</td>
      <td>7723</td>
      <td>7665</td>
    </tr>
    <tr>
      <th>16</th>
      <td>제주</td>
      <td>7674</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7914</td>
      <td>7914</td>
      <td>7914</td>
      <td>7833</td>
      <td>7724</td>
      <td>7724</td>
      <td>7739</td>
      <td>7739</td>
      <td>7739</td>
      <td>7826</td>
      <td>7285</td>
      <td>7285</td>
      <td>7343</td>
      <td>7343</td>
      <td>7343</td>
    </tr>
  </tbody>
</table>
</div>




```python
# head 로 미리보기를 합니다.
df_last.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
      <th>전용면적</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>5841.0</td>
      <td>19275.3</td>
      <td>전체</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>5652.0</td>
      <td>18651.6</td>
      <td>60㎡</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>5882.0</td>
      <td>19410.6</td>
      <td>60㎡~85㎡</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>5721.0</td>
      <td>18879.3</td>
      <td>85㎡~102㎡</td>
    </tr>
    <tr>
      <th>4</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>5879.0</td>
      <td>19400.7</td>
      <td>102㎡~</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df_first 변수에 담겨있는 데이터프레임의 정보를 info를 통해 봅니다.
df_first.info()


```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 17 entries, 0 to 16
    Data columns (total 22 columns):
     #   Column    Non-Null Count  Dtype 
    ---  ------    --------------  ----- 
     0   지역        17 non-null     object
     1   2013년12월  17 non-null     int64 
     2   2014년1월   17 non-null     int64 
     3   2014년2월   17 non-null     int64 
     4   2014년3월   17 non-null     int64 
     5   2014년4월   17 non-null     int64 
     6   2014년5월   17 non-null     int64 
     7   2014년6월   17 non-null     int64 
     8   2014년7월   17 non-null     int64 
     9   2014년8월   17 non-null     int64 
     10  2014년9월   17 non-null     int64 
     11  2014년10월  17 non-null     int64 
     12  2014년11월  17 non-null     int64 
     13  2014년12월  17 non-null     int64 
     14  2015년1월   17 non-null     int64 
     15  2015년2월   17 non-null     int64 
     16  2015년3월   17 non-null     int64 
     17  2015년4월   17 non-null     int64 
     18  2015년5월   17 non-null     int64 
     19  2015년6월   17 non-null     int64 
     20  2015년7월   17 non-null     int64 
     21  2015년8월   17 non-null     int64 
    dtypes: int64(21), object(1)
    memory usage: 3.0+ KB



```python
# 결측치가 있는지 봅니다.
df_first.isnull().sum()

```




    지역          0
    2013년12월    0
    2014년1월     0
    2014년2월     0
    2014년3월     0
    2014년4월     0
    2014년5월     0
    2014년6월     0
    2014년7월     0
    2014년8월     0
    2014년9월     0
    2014년10월    0
    2014년11월    0
    2014년12월    0
    2015년1월     0
    2015년2월     0
    2015년3월     0
    2015년4월     0
    2015년5월     0
    2015년6월     0
    2015년7월     0
    2015년8월     0
    dtype: int64



### melt로 Tidy data 만들기
pandas의 melt를 사용하면 데이터의 형태를 변경할 수 있습니다. 
df_first 변수에 담긴 데이터프레임은 df_last에 담겨있는 데이터프레임의 모습과 다릅니다. 
같은 형태로 만들어주어야 데이터를 합칠 수 있습니다. 
데이터를 병합하기 위해 melt를 사용해 열에 있는 데이터를 행으로 녹여봅니다.

* https://pandas.pydata.org/docs/user_guide/reshaping.html#reshaping-by-melt
* [Tidy Data 란?](https://vita.had.co.nz/papers/tidy-data.pdf)


```python
# head 로 미리보기 합니다.
df_first.head(1)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역</th>
      <th>2013년12월</th>
      <th>2014년1월</th>
      <th>2014년2월</th>
      <th>2014년3월</th>
      <th>2014년4월</th>
      <th>2014년5월</th>
      <th>2014년6월</th>
      <th>2014년7월</th>
      <th>2014년8월</th>
      <th>2014년9월</th>
      <th>2014년10월</th>
      <th>2014년11월</th>
      <th>2014년12월</th>
      <th>2015년1월</th>
      <th>2015년2월</th>
      <th>2015년3월</th>
      <th>2015년4월</th>
      <th>2015년5월</th>
      <th>2015년6월</th>
      <th>2015년7월</th>
      <th>2015년8월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>18189</td>
      <td>17925</td>
      <td>17925</td>
      <td>18016</td>
      <td>18098</td>
      <td>19446</td>
      <td>18867</td>
      <td>18742</td>
      <td>19274</td>
      <td>19404</td>
      <td>19759</td>
      <td>20242</td>
      <td>20269</td>
      <td>20670</td>
      <td>20670</td>
      <td>19415</td>
      <td>18842</td>
      <td>18367</td>
      <td>18374</td>
      <td>18152</td>
      <td>18443</td>
    </tr>
  </tbody>
</table>
</div>




```python
# pd.melt 를 사용하며, 녹인 데이터는 df_first_melt 변수에 담습니다. 
df_first_melt = df_first.melt(id_vars="지역", var_name="기간", value_name="평당분양가격")
df_first_melt.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역</th>
      <th>기간</th>
      <th>평당분양가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2013년12월</td>
      <td>18189</td>
    </tr>
    <tr>
      <th>1</th>
      <td>부산</td>
      <td>2013년12월</td>
      <td>8111</td>
    </tr>
    <tr>
      <th>2</th>
      <td>대구</td>
      <td>2013년12월</td>
      <td>8080</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인천</td>
      <td>2013년12월</td>
      <td>10204</td>
    </tr>
    <tr>
      <th>4</th>
      <td>광주</td>
      <td>2013년12월</td>
      <td>6098</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df_first_melt 변수에 담겨진 컬럼의 이름을 
# ["지역명", "기간", "평당분양가격"] 으로 변경합니다.
df_first_melt.columns = ["지역명", "기간", "평당분양가격"]
df_first_melt.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>기간</th>
      <th>평당분양가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2013년12월</td>
      <td>18189</td>
    </tr>
  </tbody>
</table>
</div>



### 연도와 월을 분리하기
* pandas 의 string-handling 사용하기 : https://pandas.pydata.org/pandas-docs/stable/reference/series.html#string-handling


```python
date = "2013년12월"
date
```




    '2013년12월'




```python
# split 을 통해 "년"을 기준으로 텍스트를 분리해 봅니다.
date.split("년")

```




    ['2013', '12월']




```python
# 리스트의 인덱싱을 사용해서 연도만 가져옵니다.
date.split("년")[0]

```




    '2013'




```python
# 리스트의 인덱싱과 replace를 사용해서 월을 제거합니다.
date.split("년")[-1].replace("월", "")


```




    '12'




```python
# parse_year라는 함수를 만듭니다.
# 연도만 반환하도록 하며, 반환하는 데이터는 int 타입이 되도록 합니다.
def parse_year(date):
    year = date.split("년")[0]
    year = int(year)
    return year

parse_year(date)
```




    2013




```python
# 제대로 분리가 되었는지 parse_year 함수를 확인합니다.
parse_year(date)

```




    2013




```python
# parse_month 라는 함수를 만듭니다.
# 월만 반환하도록 하며, 반환하는 데이터는 int 타입이 되도록 합니다.
def parse_month(date):
    month = date.split("년")[-1].replace("월", "")
    month = int(month)
    return month
parse_month(date)
```




    12




```python
# 제대로 분리가 되었는지 parse_month 함수를 확인합니다.
parse_month(date)

```




    12




```python
# df_first_melt 변수에 담긴 데이터프레임에서 
# apply를 활용해 연도만 추출해서 새로운 컬럼에 담습니다.
df_first_melt["연도"] = df_first_melt["기간"].apply(parse_year)
df_first_melt.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>기간</th>
      <th>평당분양가격</th>
      <th>연도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2013년12월</td>
      <td>18189</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>1</th>
      <td>부산</td>
      <td>2013년12월</td>
      <td>8111</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>2</th>
      <td>대구</td>
      <td>2013년12월</td>
      <td>8080</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인천</td>
      <td>2013년12월</td>
      <td>10204</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>4</th>
      <td>광주</td>
      <td>2013년12월</td>
      <td>6098</td>
      <td>2013</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df_first_melt 변수에 담긴 데이터프레임에서 
# apply를 활용해 월만 추출해서 새로운 컬럼에 담습니다.
df_first_melt["월"] = df_first_melt["기간"].apply(parse_month)
df_first_melt.head(1)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>기간</th>
      <th>평당분양가격</th>
      <th>연도</th>
      <th>월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2013년12월</td>
      <td>18189</td>
      <td>2013</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>부산</td>
      <td>2013년12월</td>
      <td>8111</td>
      <td>2013</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>대구</td>
      <td>2013년12월</td>
      <td>8080</td>
      <td>2013</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인천</td>
      <td>2013년12월</td>
      <td>10204</td>
      <td>2013</td>
      <td>12</td>
    </tr>
    <tr>
      <th>4</th>
      <td>광주</td>
      <td>2013년12월</td>
      <td>6098</td>
      <td>2013</td>
      <td>12</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 컬럼명을 리스트로 만들때 버전에 따라 tolist() 로 동작하기도 합니다.
# to_list() 가 동작하지 않는다면 tolist() 로 해보세요.

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
      <th>전용면적</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3463</th>
      <td>전북</td>
      <td>2019</td>
      <td>2</td>
      <td>3052.0</td>
      <td>10071.6</td>
      <td>85㎡~102㎡</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df_last와 병합을 하기 위해서는 컬럼의 이름이 같아야 합니다.
# sample을 활용해서 데이터를 미리보기 합니다.
df_last.columns.to_list()


```




    ['지역명', '연도', '월', '분양가격', '평당분양가격', '전용면적']




```python
cols = ['지역명', '연도', '월', '평당분양가격']
cols
```




    ['지역명', '연도', '월', '평당분양가격']




```python
# 최근 데이터가 담긴 df_last 에는 전용면적이 있습니다. 
# 이전 데이터에는 전용면적이 없기 때문에 "전체"만 사용하도록 합니다.
# loc를 사용해서 전체에 해당하는 면적만 copy로 복사해서 df_last_prepare 변수에 담습니다.
df_last_prepare = df_last.loc[df_last["전용면적"] == "전체", cols].copy()
df_last_prepare.head(1)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>평당분양가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>19275.3</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df_first_melt에서 공통된 컬럼만 가져온 뒤
# copy로 복사해서 df_first_prepare 변수에 담습니다.
df_first_prepare = df_first_melt[cols].copy()
df_first_prepare.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>평당분양가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2013</td>
      <td>12</td>
      <td>18189</td>
    </tr>
  </tbody>
</table>
</div>



### concat 으로 데이터 합치기
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.concat.html


```python
# df_first_prepare 와 df_last_prepare 를 합쳐줍니다.
df = pd.concat([df_first_prepare, df_last_prepare])
df.shape
```




    (1224, 4)




```python
# 제대로 합쳐졌는지 미리보기를 합니다.
df.tail()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>평당분양가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4310</th>
      <td>전북</td>
      <td>2019</td>
      <td>12</td>
      <td>8144.4</td>
    </tr>
    <tr>
      <th>4315</th>
      <td>전남</td>
      <td>2019</td>
      <td>12</td>
      <td>8091.6</td>
    </tr>
    <tr>
      <th>4320</th>
      <td>경북</td>
      <td>2019</td>
      <td>12</td>
      <td>9616.2</td>
    </tr>
    <tr>
      <th>4325</th>
      <td>경남</td>
      <td>2019</td>
      <td>12</td>
      <td>10107.9</td>
    </tr>
    <tr>
      <th>4330</th>
      <td>제주</td>
      <td>2019</td>
      <td>12</td>
      <td>12810.6</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 연도별로 데이터가 몇개씩 있는지 value_counts를 통해 세어봅니다.
df['연도'].value_counts(sort=False)

```




    2013     17
    2014    204
    2015    187
    2016    204
    2017    204
    2018    204
    2019    204
    Name: 연도, dtype: int64



### pivot_table 사용하기
* https://pandas.pydata.org/docs/user_guide/reshaping.html#reshaping-and-pivot-tables


```python
# 연도를 인덱스로, 지역명을 컬럼으로 평당분양가격을 피봇테이블로 그려봅니다.
t= pd.pivot_table(df ,index="연도" , columns="지역명", values="평당분양가격" ).round()
t
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>연도</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2013</th>
      <td>6230.0</td>
      <td>10855.0</td>
      <td>6473.0</td>
      <td>6168.0</td>
      <td>6098.0</td>
      <td>8080.0</td>
      <td>8321.0</td>
      <td>8111.0</td>
      <td>18189.0</td>
      <td>7601.0</td>
      <td>8090.0</td>
      <td>10204.0</td>
      <td>5678.0</td>
      <td>6282.0</td>
      <td>7674.0</td>
      <td>6365.0</td>
      <td>6589.0</td>
    </tr>
    <tr>
      <th>2014</th>
      <td>6332.0</td>
      <td>10509.0</td>
      <td>6729.0</td>
      <td>6536.0</td>
      <td>7588.0</td>
      <td>8286.0</td>
      <td>8240.0</td>
      <td>9180.0</td>
      <td>18997.0</td>
      <td>8085.0</td>
      <td>8362.0</td>
      <td>10075.0</td>
      <td>5719.0</td>
      <td>6362.0</td>
      <td>7855.0</td>
      <td>6682.0</td>
      <td>6620.0</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>6831.0</td>
      <td>10489.0</td>
      <td>7646.0</td>
      <td>7035.0</td>
      <td>7956.0</td>
      <td>8707.0</td>
      <td>8105.0</td>
      <td>9633.0</td>
      <td>19283.0</td>
      <td>8641.0</td>
      <td>9273.0</td>
      <td>10277.0</td>
      <td>6109.0</td>
      <td>6623.0</td>
      <td>7465.0</td>
      <td>7024.0</td>
      <td>6700.0</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>7011.0</td>
      <td>11220.0</td>
      <td>7848.0</td>
      <td>7361.0</td>
      <td>8899.0</td>
      <td>10310.0</td>
      <td>8502.0</td>
      <td>10430.0</td>
      <td>20663.0</td>
      <td>8860.0</td>
      <td>10209.0</td>
      <td>10532.0</td>
      <td>6489.0</td>
      <td>6418.0</td>
      <td>9129.0</td>
      <td>7331.0</td>
      <td>6770.0</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>7127.0</td>
      <td>11850.0</td>
      <td>8120.0</td>
      <td>7795.0</td>
      <td>9464.0</td>
      <td>11456.0</td>
      <td>9045.0</td>
      <td>11578.0</td>
      <td>21376.0</td>
      <td>9135.0</td>
      <td>11345.0</td>
      <td>10737.0</td>
      <td>7188.0</td>
      <td>7058.0</td>
      <td>10831.0</td>
      <td>7456.0</td>
      <td>6763.0</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>7681.0</td>
      <td>13186.0</td>
      <td>9019.0</td>
      <td>8505.0</td>
      <td>9856.0</td>
      <td>12076.0</td>
      <td>10180.0</td>
      <td>12998.0</td>
      <td>22889.0</td>
      <td>10355.0</td>
      <td>10241.0</td>
      <td>11274.0</td>
      <td>7789.0</td>
      <td>7626.0</td>
      <td>11891.0</td>
      <td>8013.0</td>
      <td>7874.0</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>8142.0</td>
      <td>14469.0</td>
      <td>9871.0</td>
      <td>8857.0</td>
      <td>11823.0</td>
      <td>13852.0</td>
      <td>11778.0</td>
      <td>13116.0</td>
      <td>26131.0</td>
      <td>11079.0</td>
      <td>10022.0</td>
      <td>12635.0</td>
      <td>7902.0</td>
      <td>8197.0</td>
      <td>12138.0</td>
      <td>8607.0</td>
      <td>7575.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 위에서 그린 피봇테이블을 히트맵으로 표현해 봅니다.

plt.figure(figsize=(15 , 7))
sns.heatmap(t , cmap="Blues", annot=True, fmt=".0f")

```




    <AxesSubplot:xlabel='지역명', ylabel='연도'>




    
![svg](output_125_1.svg)
    



```python
# transpose 를 사용하면 행과 열을 바꿔줄 수 있습니다.
t.transpose()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>연도</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
      <th>2017</th>
      <th>2018</th>
      <th>2019</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>6230.0</td>
      <td>6332.0</td>
      <td>6831.0</td>
      <td>7011.0</td>
      <td>7127.0</td>
      <td>7681.0</td>
      <td>8142.0</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>10855.0</td>
      <td>10509.0</td>
      <td>10489.0</td>
      <td>11220.0</td>
      <td>11850.0</td>
      <td>13186.0</td>
      <td>14469.0</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>6473.0</td>
      <td>6729.0</td>
      <td>7646.0</td>
      <td>7848.0</td>
      <td>8120.0</td>
      <td>9019.0</td>
      <td>9871.0</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>6168.0</td>
      <td>6536.0</td>
      <td>7035.0</td>
      <td>7361.0</td>
      <td>7795.0</td>
      <td>8505.0</td>
      <td>8857.0</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>6098.0</td>
      <td>7588.0</td>
      <td>7956.0</td>
      <td>8899.0</td>
      <td>9464.0</td>
      <td>9856.0</td>
      <td>11823.0</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>8080.0</td>
      <td>8286.0</td>
      <td>8707.0</td>
      <td>10310.0</td>
      <td>11456.0</td>
      <td>12076.0</td>
      <td>13852.0</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>8321.0</td>
      <td>8240.0</td>
      <td>8105.0</td>
      <td>8502.0</td>
      <td>9045.0</td>
      <td>10180.0</td>
      <td>11778.0</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>8111.0</td>
      <td>9180.0</td>
      <td>9633.0</td>
      <td>10430.0</td>
      <td>11578.0</td>
      <td>12998.0</td>
      <td>13116.0</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>18189.0</td>
      <td>18997.0</td>
      <td>19283.0</td>
      <td>20663.0</td>
      <td>21376.0</td>
      <td>22889.0</td>
      <td>26131.0</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>7601.0</td>
      <td>8085.0</td>
      <td>8641.0</td>
      <td>8860.0</td>
      <td>9135.0</td>
      <td>10355.0</td>
      <td>11079.0</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>8090.0</td>
      <td>8362.0</td>
      <td>9273.0</td>
      <td>10209.0</td>
      <td>11345.0</td>
      <td>10241.0</td>
      <td>10022.0</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>10204.0</td>
      <td>10075.0</td>
      <td>10277.0</td>
      <td>10532.0</td>
      <td>10737.0</td>
      <td>11274.0</td>
      <td>12635.0</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>5678.0</td>
      <td>5719.0</td>
      <td>6109.0</td>
      <td>6489.0</td>
      <td>7188.0</td>
      <td>7789.0</td>
      <td>7902.0</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>6282.0</td>
      <td>6362.0</td>
      <td>6623.0</td>
      <td>6418.0</td>
      <td>7058.0</td>
      <td>7626.0</td>
      <td>8197.0</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>7674.0</td>
      <td>7855.0</td>
      <td>7465.0</td>
      <td>9129.0</td>
      <td>10831.0</td>
      <td>11891.0</td>
      <td>12138.0</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>6365.0</td>
      <td>6682.0</td>
      <td>7024.0</td>
      <td>7331.0</td>
      <td>7456.0</td>
      <td>8013.0</td>
      <td>8607.0</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>6589.0</td>
      <td>6620.0</td>
      <td>6700.0</td>
      <td>6770.0</td>
      <td>6763.0</td>
      <td>7874.0</td>
      <td>7575.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 바뀐 행과 열을 히트맵으로 표현해 봅니다.

plt.figure(figsize=(15 , 7))
sns.heatmap(t.transpose() , cmap="Blues", annot=True, fmt=".0f")
```




    <AxesSubplot:xlabel='연도', ylabel='지역명'>




    
![svg](output_127_1.svg)
    



```python
# Groupby로 그려봅니다. 인덱스에 ["연도", "지역명"] 을 넣고 그려봅니다.
g = df.groupby(["연도","지역명"])["평당분양가격"].mean().unstack().round()
g
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>연도</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2013</th>
      <td>6230.0</td>
      <td>10855.0</td>
      <td>6473.0</td>
      <td>6168.0</td>
      <td>6098.0</td>
      <td>8080.0</td>
      <td>8321.0</td>
      <td>8111.0</td>
      <td>18189.0</td>
      <td>7601.0</td>
      <td>8090.0</td>
      <td>10204.0</td>
      <td>5678.0</td>
      <td>6282.0</td>
      <td>7674.0</td>
      <td>6365.0</td>
      <td>6589.0</td>
    </tr>
    <tr>
      <th>2014</th>
      <td>6332.0</td>
      <td>10509.0</td>
      <td>6729.0</td>
      <td>6536.0</td>
      <td>7588.0</td>
      <td>8286.0</td>
      <td>8240.0</td>
      <td>9180.0</td>
      <td>18997.0</td>
      <td>8085.0</td>
      <td>8362.0</td>
      <td>10075.0</td>
      <td>5719.0</td>
      <td>6362.0</td>
      <td>7855.0</td>
      <td>6682.0</td>
      <td>6620.0</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>6831.0</td>
      <td>10489.0</td>
      <td>7646.0</td>
      <td>7035.0</td>
      <td>7956.0</td>
      <td>8707.0</td>
      <td>8105.0</td>
      <td>9633.0</td>
      <td>19283.0</td>
      <td>8641.0</td>
      <td>9273.0</td>
      <td>10277.0</td>
      <td>6109.0</td>
      <td>6623.0</td>
      <td>7465.0</td>
      <td>7024.0</td>
      <td>6700.0</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>7011.0</td>
      <td>11220.0</td>
      <td>7848.0</td>
      <td>7361.0</td>
      <td>8899.0</td>
      <td>10310.0</td>
      <td>8502.0</td>
      <td>10430.0</td>
      <td>20663.0</td>
      <td>8860.0</td>
      <td>10209.0</td>
      <td>10532.0</td>
      <td>6489.0</td>
      <td>6418.0</td>
      <td>9129.0</td>
      <td>7331.0</td>
      <td>6770.0</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>7127.0</td>
      <td>11850.0</td>
      <td>8120.0</td>
      <td>7795.0</td>
      <td>9464.0</td>
      <td>11456.0</td>
      <td>9045.0</td>
      <td>11578.0</td>
      <td>21376.0</td>
      <td>9135.0</td>
      <td>11345.0</td>
      <td>10737.0</td>
      <td>7188.0</td>
      <td>7058.0</td>
      <td>10831.0</td>
      <td>7456.0</td>
      <td>6763.0</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>7681.0</td>
      <td>13186.0</td>
      <td>9019.0</td>
      <td>8505.0</td>
      <td>9856.0</td>
      <td>12076.0</td>
      <td>10180.0</td>
      <td>12998.0</td>
      <td>22889.0</td>
      <td>10355.0</td>
      <td>10241.0</td>
      <td>11274.0</td>
      <td>7789.0</td>
      <td>7626.0</td>
      <td>11891.0</td>
      <td>8013.0</td>
      <td>7874.0</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>8142.0</td>
      <td>14469.0</td>
      <td>9871.0</td>
      <td>8857.0</td>
      <td>11823.0</td>
      <td>13852.0</td>
      <td>11778.0</td>
      <td>13116.0</td>
      <td>26131.0</td>
      <td>11079.0</td>
      <td>10022.0</td>
      <td>12635.0</td>
      <td>7902.0</td>
      <td>8197.0</td>
      <td>12138.0</td>
      <td>8607.0</td>
      <td>7575.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(15 , 7))
sns.heatmap(g.T, annot=True, fmt=".0f", cmap="Greens")
```




    <AxesSubplot:xlabel='연도', ylabel='지역명'>




    
![svg](output_129_1.svg)
    


## 2013년부터 최근 데이터까지 시각화하기
### 연도별 평당분양가격 보기


```python
# barplot 으로 연도별 평당분양가격 그리기
sns.barplot(data=df, x="연도" , y="평당분양가격")
```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_131_1.svg)
    



```python
# pointplot 으로 연도별 평당분양가격 그리기
plt.figure(figsize=(12 , 4))
sns.pointplot(data=df, x="연도" , y="평당분양가격", hue="지역명")
plt.legend(bbox_to_anchor=(1.02, 1), loc=2, borderaxespad=0.)

```




    <matplotlib.legend.Legend at 0x7f9a220c59a0>




    
![svg](output_132_1.svg)
    



```python
# 서울만 barplot 으로 그리기
df_seoul = df[df["지역명"] == "서울"].copy()
print(df_seoul.shape)


sns.barplot(data=df_seoul , x="연도" , y="평당분양가격", color="Blue")
sns.pointplot(data=df_seoul , x="연도" , y="평당분양가격")
```

    (72, 4)





    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_133_2.svg)
    



```python
# 연도별 평당분양가격 boxplot 그리기
sns.boxplot(data=df, x="연도" , y="평당분양가격")

```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_134_1.svg)
    



```python
sns.boxenplot(data=df, x="연도" , y="평당분양가격")

```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_135_1.svg)
    



```python
# 연도별 평당분양가격 violinplot 그리기
plt.figure(figsize=(10 , 4))
sns.violinplot(data=df, x="연도" , y="평당분양가격")

```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![svg](output_136_1.svg)
    



```python
# 연도별 평당분양가격 swarmplot 그리기
sns.lmplot(data=df, x="연도" , y="평당분양가격")


```




    <seaborn.axisgrid.FacetGrid at 0x7f9a21dc3d30>




    
![svg](output_137_1.svg)
    



```python
plt.figure(figsize=(12 , 5))
sns.violinplot(data=df, x="연도" , y="평당분양가격")
sns.swarmplot(data=df, x="연도" , y="평당분양가격", hue="지역명")
plt.legend(bbox_to_anchor=(1.02, 1), loc=2, borderaxespad=0.)

```




    <matplotlib.legend.Legend at 0x7f9a21b758b0>




    
![svg](output_138_1.svg)
    


### 지역별 평당분양가격 보기


```python
# barplot 으로 지역별 평당분양가격을 그려봅니다.
plt.figure(figsize=(12 , 4))
sns.barplot(data=df, x="지역명", y="평당분양가격")

```




    <AxesSubplot:xlabel='지역명', ylabel='평당분양가격'>




    
![svg](output_140_1.svg)
    



```python
# boxplot 으로 지역별 평당분양가격을 그려봅니다.
plt.figure(figsize=(12 , 4))
sns.boxplot(data=df, x="지역명", y="평당분양가격")
```




    <AxesSubplot:xlabel='지역명', ylabel='평당분양가격'>




    
![svg](output_141_1.svg)
    



```python
plt.figure(figsize=(12 , 4))
sns.boxenplot(data=df, x="지역명", y="평당분양가격")
```




    <AxesSubplot:xlabel='지역명', ylabel='평당분양가격'>




    
![svg](output_142_1.svg)
    



```python
# violinplot 으로 지역별 평당분양가격을 그려봅니다.
plt.figure(figsize=(24 , 4))
sns.violinplot(data=df, x="지역명", y="평당분양가격")
```




    <AxesSubplot:xlabel='지역명', ylabel='평당분양가격'>




    
![svg](output_143_1.svg)
    



```python
# swarmplot 으로 지역별 평당분양가격을 그려봅니다.
plt.figure(figsize=(24 , 4))
sns.swarmplot(data=df, x="지역명", y="평당분양가격", hue="연도")

```




    <AxesSubplot:xlabel='지역명', ylabel='평당분양가격'>




    
![svg](output_144_1.svg)
    



```python

```


```python

```

<big> 고생 많으셨습니다! 다음 챕터도 화이팅 입니다!</big>


```python

```


```python

```
