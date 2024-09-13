# 머신러닝

- 기계(컴퓨터 알고리즘) 스스로 데이터를 학습하여 서로 다른 변수간의 관계를 찾아 나가는 과정

- 해결하려는 문제에 따라 예측(prediction), 분류(classification), 군집(clustering) 알고리즘으로 분류

## 지도 학습 vs 비지도 학습

- 지도학습 : 정답 데이터를 다른 데이터와 함께 컴퓨터 알고리즘에 입력하는 방식

- 비지도 학습 : 정답 데이터 없이 컴퓨터 알고리즘 스스로 데이터로부터 숨은 패턴을 찾아네는 방식


| 학습 유형  | 알고리즘 (분석모형) | 특징|
|------------|-------------------|-----------|
| 지도학습   | 회귀 분석 <br> 분류 | 정답을 알고 있는 상태에서 학습 <br> 모형 평가 방법이 다양한 편|
| 비지도 학습 | 군집 분석 | 정답이 없는 상태에서 서로 비슷한 데이터를 찾아서 그룹화 <br> 모형 평가 방법이 제한적 |


## 머신러닝 프로세스

데이터 정리 -> 데이터 분리(훈련/검증) -> 알고리즘 준비 -> 모형 학습(훈련 데이터)

-> 예측(검증 데이터) -> 모형 평가 -> 모형 활용

----

# 회귀분석

머신러닝 알고리즘 중에서도 비교적 이해하기 쉽고 널리 활용되고 있는 대표적인 알고리즘

가격, 매출, 주가, 환율, 수량 등 연속적인 값을 갖는 연속 변수를 예측하는 데 주로 활용

분석 모형이 예측하고자 하는 목표를 종속(dependent) 변수 또는 예측(ptrdictor) 변수라고 부른다

예측을 위해 모형이 사용하는 속성을 독립(independent) 변수 또는 설명(explanatory) 변수

간단하게 말하면 **값을 예측하는 것**.

## 단순회귀분석

- 데이터 확보(수집)
- 데이터 전처리
- 데이터 분할(학습용, 테스트) 8:2 or 7:3
- baseline 모델 선택
- 학습
  - 평가(회귀는 정답과 예측값의 차이 즉, 오차)
  - if 평가가 안좋으면 모델을 변경하거나 파라메터를 튜닝해서 반복
  - or 데이터를 다시 전처리 및 특성을 추가
- 배포
- 평가 방법
  - 오차평가
  - MSE
 
----

```python
# Step 1 - 데이터 준비

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

path = '/content/drive/MyDrive/Colab Notebooks/encore/csv/auto-mpg.csv'

df = pd.read_csv(path, header=None)
df.columns = ['mpg', 'cylinders','displacement','horsepower','weight','acceleration','model year','origin','name']
df.head()

```

```python
# Step 2 - 데이터 탐색
# 학습 전까지의 데이터 처리 작업 - 탐색적 데이터 분석 EDA

# 데이터 자료형 확인
df.info()

# 데이터 통계 요약정보 확인
df.describe()

# 누락 데이터 확인
df.isnull().sum()

# 중복 데이터 확인
df.duplicated().sum()

# 상관계수 분석 - 데이터프레임
corr = df.corr(numeric_only = True)

```

```python
# Step 3 - 데이터 전처리
# 결측치 처리 : NA, NA와 같은 데이터(불가능한 데이터)들 처리 

df['horsepower'].unique()
# unique를 통해 '?'를 확인

df['horsepower'] = df['horsepower'].replace('?', np.nan)
df['horsepower'] = df['horsepower'].astype('float')
df['horsepower'].unique()

# '?'를 nan 값으로 수정
```

```python

# 결측치 제거
print(df['horsepower'].isnull().sum())
df_nan = df.dropna(subset=['horsepower'], axis=0)
print(df_nan['horsepower'].isnull().sum())

# 6 -> 0
# 또는 결측치 대체
```

```python
### 종속 변수 Y인 "연비(mpg)"와 다른 변수 간의 선형 관계 그래프(산점도)로 확인
# Pandas 함수로 산점도 그리기

ndf.plot(kind='scatter', x='weight', y='mpg', c='coral', s=10, figsize=(10,5), grid=True)
```

![image](https://github.com/user-attachments/assets/3f3e6b2d-9549-4528-87b8-b8f461560f92)

```python
# seaborn으로 산점도 그리기
fig = plt.figure(figsize=(10,5))
ax1 = fig.add_subplot(1,2,1)
ax2 = fig.add_subplot(1,2,2)
sns.regplot(x='weight', y='mpg', data=ndf, ax=ax1)
sns.regplot(x='weight', y='mpg', data=ndf, ax=ax2, fit_reg = False)
plt.show()
```

![image](https://github.com/user-attachments/assets/49e08654-420b-4f56-8cd7-e6890ce75174)


```python
# seaborn 조인트 그래프 - 산점트, 히스토그램
sns.jointplot(x='weight', y='mpg', data=ndf)
sns.jointplot(x='weight', y='mpg', kind='reg', data=ndf)
```

![image](https://github.com/user-attachments/assets/0b678936-691d-47ae-af72-7d36028938f1)

![image](https://github.com/user-attachments/assets/85d0def8-bee3-4dd2-b708-d113c1d5f76e)


```python
# Step 5 - 데이터셋 구분 - 훈련용(train data)/검증용(test data)

# 속성(변수) 선택
X = ndf[['weight']] # 독립 변수 X
# X가 2차원으로 입력되야하므로 [[]]를 사용하여 넣어준다.
y = ndf['mpg'] # 종속 변수 Y

# train data 와 test data로 구분(7:3 비율)

from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, # 독립 변수
                                                    y, # 종속 변수
                                                    test_size=0.3, # 검증 30%
                                                    random_state=10) # 랜덤 추출 값

print('train data 개수: ', len(X_train))
print('test data 개수: ', len(X_test))

# train data 개수:  278
# test data 개수:  120
```

```python
# Step 6 - 모델 학습 및 검증

# sklearn 라이브러리에서 선형회귀분석 모듈 가져오기
from sklearn.linear_model import LinearRegression

lr = LinearRegression() # 객체 생성
lr.fit(X_train, y_train) # 학습

r_square = lr.score(X_test, y_test)
print('R^2 결정계수 : ', r_square)

# 회귀식의 기울기
print('기울기 a : ', lr.coef_)
# 회귀식의 y절편
print('y절편 b : ', lr.intercept_)

# R^2 결정계수 :  0.689363809315209
# 기울기 a :  [-0.0076554]
# y절편 b :  46.603650522246326
```

```python
# 모델에 test data 데이터를 입력하여 예측한 값 y_hat을 실제 값 y와 비교
y_hat = lr.predict(X_test)

# 오차 계산
test_preds = pd.DataFrame(y_test)
test_preds.columns = ['y_test']
test_preds['y_hat'] = y_hat
test_preds['squared_error'] = (test_preds['y_test'] - test_preds['y_hat'])**2
test_preds
```

![image](https://github.com/user-attachments/assets/cfc4d796-55d1-4483-ba16-fa35f5d4533e)


```python
# 평균 제곱 오차
mse = test_preds['squared_error'].mean()
print('평균 제곱 오차 : ', mse)

# 평균 제곱 오차 :  17.898336128759947
```

```python
# 오차 분석
fig, axes = plt.subplots(1,2,figsize=(10,5))
sns.regplot(x='y_test', y='y_hat', data=test_preds, ax=axes[0])
sns.kdeplot(x='squared_error', data=test_preds, ax=axes[1])
plt.show()
```

![image](https://github.com/user-attachments/assets/aacf998c-e5e2-43f7-9c8e-b384c71d703a)

```python
from sklearn.model_selection import train_test_split

# 학습용 데이터, 테스트용 데이터 섞고 분리
x_train, x_test, y_train, y_test = train_test_split(X,y,random_state=10)
print(x_train.shape, y_train.shape)
print(x_test.shape, y_test.shape)

from sklearn.linear_model import LinearRegression
# sklearn 계열의 모델은 학습 순서 및 사용 메소드가 정해져있음
# sklearn 계열은 2차원의 데이터를 원한다.
model = LinearRegression() # 기본 객체 생성

model.fit(x_train, y_train) # 학습
y_pred = model.predict(x_test) # 예측

print(model.score(x_test, y_test)) # 평가

from sklearn.metrics import mean_squared_error
mse = mean_squared_error(y_test, y_pred)
print(mse)

# 결정계수 : R^2 모델이 데이터의 변동성을 잘 설명하는지 나타내는 지표
# 결정계수는 1에 가까울 수록 성능이 좋음
# 1 -(SSR /SST)
# SSR : 잔차, 정답-예측 차이의 제곱의 합
# SST : 실제값 - 평균값 사이의 제곱합
```

----

## 다항회귀분석

2차함수 이상의 다항 함수를이용하여 두 변수 간의 선형관계를 설명하는 알고리즘

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

path = '/content/drive/MyDrive/Colab Notebooks/encore/csv/auto-mpg.csv'

df = pd.read_csv(path, header=None)

df.columns = ['mpg', 'cylinders','displacement','horsepower','weight','acceleration','model year','origin','name']

# horsepower 열의 자료형 변경(문자열 -> 숫자)
df['horsepower'] = df['horsepower'].replace('?', np.nan)
df['horsepower'] = df['horsepower'].astype('float')

# 결측치 대체
df['horsepower'] = df['horsepower'].fillna(df['horsepower'].mean())

# 분석에 활용할 열(속성)을 선택(연비, 실린더, 출력, 중량)
ndf = df[['mpg','cylinders','horsepower','weight']]

# ndf 데이터를 train data 와 test data로 구분(7:3 비율)
X = ndf[['weight']] # 독립 변수 X
y = ndf['mpg'] # 종속 변수 Y

from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, # 독립 변수
                                                    y, # 종속 변수
                                                    test_size=0.3, # 검증 30%
                                                    random_state=77)

print('train data 개수: ', len(X_train))
print('test data 개수: ', len(X_test))

# train data 개수:  278
# test data 개수:  120
```

```python
# Step 6 - 다항회귀분석 모형

# sklearn 라이브러리에서 필요한 모듈 가져오기
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures

# 다항식으로 변환
poly = PolynomialFeatures(degree=2)
X_train_poly = poly.fit_transform(X_train)

print('원본 데이터: ', X_train.shape)
print('2차 다항식으로 변환 데이터: ', X_train_poly.shape)

# 원본 데이터:  (278, 1)
# 2차 다항식으로 변환 데이터:  (278, 3)

```

```python
pr = LinearRegression()
pr.fit(X_train_poly, y_train)

X_test_poly = poly.fit_transform(X_test)
r_square = pr.score(X_test_poly, y_test)
print('R^2 결정계수: ', r_square)

# R^2 결정계수:  0.6640132359160891
```

```python
y_hat_test = pr.predict(X_test_poly)

fig, axes = plt.subplots(figsize=(10,5))
axes.plot(X_train, y_train, 'o', label='train data')
axes.plot(X_test, y_hat_test, 'r+', label='predicted value')
axes.legend(loc='best')
plt.xlabel('weight')
plt.ylabel('mpg')
plt.show()
```

![image](https://github.com/user-attachments/assets/c316ae44-aeb4-4bf3-8808-eb34ccc48938)

```python
# 모델에 test data 데이터를 입력하여 예측한 값 y_hat_test를 실제 값 y_test와 비교
X_ploy = poly.fit_transform(X_test)

# 오차 계산
test_preds = pd.DataFrame(y_test)
test_preds.columns = ['y_test']
test_preds['y_hat'] = y_hat_test
test_preds['squared_error'] = (test_preds['y_test'] - test_preds['y_hat'])**2


# 평균 제곱 오차
mse = test_preds['squared_error'].mean()
print('평균 제곱 오차 : ', mse)

# 평균 제곱 오차 :  18.49811318238353
```

```python
fig, axes = plt.subplots(1,2,figsize=(10,5))
sns.regplot(x='y_test', y='y_hat', data=test_preds, ax=axes[0])
sns.kdeplot(x='squared_error', data=test_preds, ax=axes[1])
plt.show()
```

![image](https://github.com/user-attachments/assets/872931bc-eb98-45b4-bbf5-cb9bac03dd15)


-----
## 다중회귀분석

여러 개의 독립 변수가 종속 변수에 영향을 주고 선형 관계를 갖는 경우에 사용

모델의 예측값인 종속 변수에 대한 실제 데이터를 알고 있는 상태에서 학습하기 때문에 지도학습으로 분류

```python

# train data의 산점도와 test data로 예측한 회귀선을 그래프로 출력
y_hat_test = lr.predict(X_test)

fig, axes = plt.subplots(1,3,figsize=(10,5))

for i, col in enumerate(X_train.columns):
    axes[i].plot(X_train[col], y_train, 'o', label='train data')
    axes[i].plot(X_test[col], y_hat_test, 'r+', label='predicted value')
    axes[i].set_xlabel(col)
    axes[i].set_ylabel(col)
    axes[i].legend(loc='best')
plt.show()
```

![image](https://github.com/user-attachments/assets/77717e71-fff8-4809-bbae-8debb5e3c8fa)


```python
# 사이킷런 함수 활용(평균 제곱 오차)
from sklearn.metrics import mean_squared_error
mse = mean_squared_error(y_test, y_hat_test)
print(np.round(mse,2))

from sklearn.metrics import mean_absolute_percentage_error
mae = mean_absolute_percentage_error(y_test, y_hat_test)
print(np.round(mae,2))

# 19.05
# 0.15
```

# 회귀 정리
- 선형회귀(단항, 다항) - 직선의 방정식
- 비선형회귀(다중회귀) 차수를 2차원 이상으로 올려서 학습
- 평가방법은 단순히 오차를 계산, 이 값이 좋은지 나쁜지는 모른다
- 0 ~ 1 사이의 값을 가지는 R-Square 결정계수 -> 1에 가까울수록 높은 성능
- model.score 함수를 이용해 평가

----

# 분류

예측하려는 대상의 특성(설명 변수)을 입력받고, 목표 변수가 갖고 있는 카테고리(범주형) 값 중에서 어느 하나로 분류하여 예측하는 기법

## KNN

K-Neareat-Neighbors

새로운 관측값이 주어지면 기존 데이터 중에서 가장 특성이 비슷한 k개의 이웃을 먼저 찾는다.

그리고 가까운 이웃들이 갖고 있는 목표 값과 같은 값으로 분류하여 예측한다.

```python
# Step 1 - 데이터 준비

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = sns.load_dataset('titanic')
df.head()

# Step - 2 데이터 탐색

df.info()
df.describe()
df.describe(include='object')
df.isnull().sum()
df.duplicated().sum()
df['survived'].value_counts()
```

```python
sns.countplot(data=df, x='survived')
# 데이터 불균형
# 이처럼 분류 문제에서 데이터 불균형은 클래스 간에 샘플 수가 현저하게 차이나는 경우
# 머신러닝 모델이 다수 클래스에 치우친 학습을 하는 경향이 있어, 결과적으로 소수 클래스를 제대로 예측하지 못하는 문제 발생
# 데이터 불균형 문제에 대응하기 위한 방안으로는 소수 클래스의 샘플을 증가시키는 오버 샘플링, 다수의 샘플을 줄이는 언더 샘플링 등이 있음
# 모델의 성는 평가지표를 선택할 때에도 정확도 대신 정밀도, 재현율 F1 점수 등 불균형 데이터에 적합한 성능 지표를 사용하여 모델을 평가

```

![image](https://github.com/user-attachments/assets/a3ac50f5-2a85-4575-99ae-3e7e01cea8f5)

```python
# 시각화
g = sns.FacetGrid(df, col='survived', row='pclass', hue='sex')
g.map(sns.kdeplot, 'age', alpha=0.5, fill=True)
g.add_legend()
```

![image](https://github.com/user-attachments/assets/b5672ebc-b758-4a75-8934-e71fe8f303cc)


```python
# 시각화
sns.displot(x='sibsp', kind='hist', hue='survived', data=df, multiple='fill')
```

![image](https://github.com/user-attachments/assets/0bea3256-e353-4090-aa57-db4f5fb48f75)

```python
sns.displot(x='parch', kind='hist', hue='survived', data=df, multiple='fill')
```

![image](https://github.com/user-attachments/assets/51fe45ef-d0df-4f6b-8d5a-6ba2a67d19c9)

```python
sns.boxplot(x='embarked', y='age', hue='survived', data=df)
```

![image](https://github.com/user-attachments/assets/5b6946d0-b506-445d-8ea1-15b07ef3791c)


```python
# Step 3 - 데이터 전처리
print('중복 제거 이전: ', df.shape)
df = df.drop_duplicates()
print('중복 제거 이후: ', df.shape)

# 머신러닝에서 중복 데이터를 제거하는 의미
# 데이터의 품질을 확보하고 모델의 성능을 향상시키는 데 필요한 과정
# 중복 데이터가 존재할 경우, 머신러닝 모델이 중복 데이터가 갖는 특정 패턴에 과적합되어 일반화 성능이 떨어지게 하는 문제가 발생
# 데이터 불균형 문제를 심화시키거나 학습 효율성을 저하, 최종적으로 모델 평가의 정확성을 왜곡할 수 있음
# 데이터 전처리 단계에서 중복 데이터의 식별 및 제거는 반드시 거쳐야 하는 필수적인 과정
```

----

# 정리 

- 인공지능은 사람처럼 학습하고 추론할 수 있는 지능을 가진 시스템을 만드는 기술

  - 인공지능은 강인공지능과 약인공지능으로 나눌 수 있다.

- 머신러닝은 규칙을 프로그래밍하지 않아도 자동으로 데이터에서 규칙을 학습하는 알고리즘을 연구하는 분야

  - 사이킷런이 대표적인 라이브러리

- 딥러닝은 인공 신경망이라고도 하며, 텐서플로와 파이토치가 대표적인 라이브러리

- **특성**은 데이터를 표현하는 하나의 성질

- **훈련** : 머신러닝 알고리즘이 데이터에서 규칙을 찾는 과정

  - 사이킷런에서는 fit() 메서드가 하는 역할

- **k-최근점 이웃 알고리즘**은 가장 간단한 머신러닝 알고리즘

  - 어떤 규칙을 찾기보다는 전체 데이터를 메모리에 가지고 있는 것이 전부

- 머신러닝 프로그램에서는 알고지름이 구현된 객체를 **모델**이라고 부른다

  - 종종 알고리즘 자체를 모델이라고도 부름

- **정확도**는 정확한 답을 몇 개 맞혔는지를 백분율로 나타낸 값

  - 사이킷런에서는 0~1 사이의 값으로 출력

  - 정확도 = (정확히 맞힌 개수) / (전체 데이터 개수)
