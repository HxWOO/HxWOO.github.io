---
layout: post
title:  "K-Means로 고객 마음 뽀개기! 🛍️ Customer Segmentation 실습"
date:   2025-08-29 10:00:00 +0900
categories: [Machine_Learning, Woori FISA]
tags: [K-Means, Clustering, Customer Segmentation, Python, Scikit-learn, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🛍️ 고객들의 마음, 어떻게 알아볼 수 있을까?

수많은 고객 데이터 속에서 비슷한 취향을 가진 사람들을 찾아 그룹으로 묶을 수 있다면 어떨까? 오늘은 비지도학습의 대표주자, K-Means를 활용해 고객 데이터를 분석하고 숨겨진 소비 패턴을 찾아내는 Customer Segmentation 실습을 진행했다.

> #### 🗣️: "데이터 속에 숨어있는 고객의 페르소나를 찾아라!"
> K-Means 클러스터링은 정답이 없는 데이터 속에서 스스로 패턴을 학습하여 고객을 그룹화한다. 이를 통해 각 그룹의 특징을 파악하고 맞춤형 마케팅 전략을 세울 수 있다.

---

### 🎯 추천 시스템과 K-Means

추천 시스템은 크게 콘텐츠 기반, 유저 기반, 그리고 둘을 합친 협업 필터링 방식으로 나뉜다. 하지만 이런 방식들은 신규 유저나 새로운 상품에 대한 데이터가 부족한 '콜드 스타트(Cold Start)' 문제에 취약하다는 한계가 있다.

이때 K-Means와 같은 비지도 학습 클러스터링을 사용하면 초기 고객군이나 상품군을 미리 분류하여 콜드 스타트 문제를 효과적으로 해결할 수 있다.

---

### 📜 1. 데이터 탐험하기 (EDA)

먼저 고객들의 결제 데이터를 불러와 어떤 정보가 담겨있는지 살펴보았다.

- **데이터셋**: 100명의 고객(`cc_num`)이 11개 카테고리(`category`)에 대해 결제한 24만 건의 데이터(`amt`)
- **목표**: 소비 패턴이 비슷한 고객들을 n개의 클러스터로 나누고, 각 클러스터의 특징을 파악하여 마케팅 전략 수립하기!

```python
import pandas as pd
import numpy as np

# 구글 드라이브 마운트 및 데이터 로드
from google.colab import drive
drive.mount('/content/drive')
customer = pd.read_csv('/content/drive/MyDrive/customer.csv')

# 데이터 확인
customer.info()
customer.describe(include='all')
customer.category.value_counts()
```

> 24만개의 데이터를 그대로 사용하기는 힘드니, 100명의 고객이 각 11개 카테고리에 총 얼마를 썼는지 알 수 있는 형태로 데이터를 가공해야 했다.

---

### 🛠️ 2. 데이터 전처리 및 스케일링

K-Means는 거리 기반 알고리즘이라 데이터의 스케일에 큰 영향을 받는다. 따라서 각 고객의 소비 데이터를 공평하게 비교하기 위해 두 가지 전처리 과정을 거쳤다.

#### 가. 고객-카테고리 행렬 만들기

`get_dummies`를 이용한 원-핫 인코딩으로 각 고객(`cc_num`)이 카테고리별로 얼마를 소비했는지 나타내는 행렬을 만들었다.

```python
# 원-핫 인코딩
customer_cat = pd.get_dummies(customer, columns=['category'])
category_list = customer_cat.iloc[:, 2:]

# 카테고리별 소비 금액 계산
for i in category_list:
    customer_cat[i] = customer_cat[i]*customer_cat['amt']

# 고객(cc_num) 기준으로 그룹화
customer_cat_to_cc = customer_cat.groupby('cc_num').sum()
customer_cat_to_cc.head()
```

#### 나. 데이터 스케일링 (StandardScaler)

각 고객의 카테고리별 소비액을 평균 0, 분산 1을 갖도록 표준화했다. 이를 통해 특정 카테고리에 소비액이 큰 고객의 영향력을 줄이고 모든 카테고리를 공평하게 비교할 수 있게 된다.

```python
from sklearn.preprocessing import StandardScaler

std_scaler = StandardScaler()
std_customer_cat_to_cc = pd.DataFrame(std_scaler.fit_transform(customer_cat_to_cc), columns=customer_cat_to_cc.columns, index=customer_cat_to_cc.index)
std_customer_cat_to_cc.describe()
```

---

### 🤖 3. K-Means 모델링: 최적의 K 찾기

이제 몇 개의 클러스터(K)로 나누는 것이 가장 이상적일지 찾아야 했다. `Elbow Method`와 `Silhouette Score` 두 가지 방법을 사용했다.

#### 가. Elbow Method

클러스터의 개수를 늘려가면서 각 클러스터 내 데이터들의 응집도를 나타내는 `inertia` 값의 변화를 관찰한다. 그래프가 팔꿈치처럼 급격히 꺾이는 지점이 최적의 K가 될 수 있다.

```python
from sklearn.cluster import KMeans
import seaborn as sns

list_ = []
for k in range(2, 30):
    kmeans = KMeans(n_clusters=k)
    kmeans.fit(std_customer_cat_to_cc)
    list_.append(kmeans.inertia_)

sns.lineplot(x=range(2,30), y=list_, marker='x') # 4 ~ 13 근처에서 완만해짐
```
> 그래프를 보니 K=4 이후부터 `inertia` 값의 감소폭이 완만해지는 것을 확인할 수 있었다.

#### 나. Silhouette Score

실루엣 점수는 클러스터가 얼마나 잘 분리되었는지를 나타내는 지표다. 1에 가까울수록 좋다.

```python
from sklearn.metrics import silhouette_score

list_2 = []
for k in range(4, 15):
    kmeans = KMeans(n_clusters=k, random_state=121)
    kmeans.fit(std_customer_cat_to_cc)
    y_pred = kmeans.predict(std_customer_cat_to_cc)
    list_2.append(silhouette_score(std_customer_cat_to_cc, y_pred))

sns.lineplot(x=range(4,15), y=list_2, marker='x')
```

실루엣 점수와 각 클러스터의 분포를 시각화하는 함수를 통해 확인한 결과, K=4일 때 가장 안정적인 모습을 보였다.

```python
# (visualize_silhouette 함수 코드는 생략)
visualize_silhouette([4, 5, 6, 13, 14], std_customer_cat_to_cc)
```

> K=4일 때 모든 클러스터가 평균 실루엣 점수(붉은 점선) 이상을 기록했고, 크기도 비교적 균일했다. 최종 클러스터 개수는 4개로 결정했다! 🎉

---

### 📊 4. 클러스터 분석 및 해석

K=4로 최종 모델을 학습하고, 각 고객이 어떤 클러스터에 속하는지 확인했다.

```python
# 최종 모델 선정 및 학습
kmeans = KMeans(n_clusters=4, random_state=121)
kmeans.fit(std_customer_cat_to_cc)
y_pred = kmeans.predict(std_customer_cat_to_cc)

# 데이터프레임에 클러스터 라벨 추가
std_customer_cat_to_cc['label'] = y_pred
std_customer_cat_to_cc.label.value_counts()
# 0    35
# 1    30
# 2    23
# 3    12

# 각 클러스터의 특징(평균 소비 패턴) 확인
clusteredDF = std_customer_cat_to_cc.groupby('label').mean()
```

`heatmap`으로 각 클러스터의 소비 패턴을 시각화하니 그룹별 특징이 명확하게 드러났다.

```python
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(12, 8))
sns.heatmap(clusteredDF, annot=True, fmt='.2f');
```

- **Cluster 0**: `category_10`과 `category_8`에 소비가 집중된 그룹.
- **Cluster 1**: `category_5`와 `category_9`에 압도적인 소비를 보이는 그룹.
- **Cluster 2**: `category_2`와 `category_4`에 소비가 많은 그룹.
- **Cluster 3**: `category_1`과 `category_3`에 소비가 집중된 그룹.

> 각 클러스터는 특정 카테고리에 뚜렷한 소비 성향을 보였다. 이제 이 특징을 바탕으로 마케팅 페르소나를 만들고 전략을 세워볼 수 있다.

---

### ✨ 오늘의 회고

오늘은 K-Means 클러스터링을 이용해 고객 데이터를 분석하는 실습을 진행했다. 막연하게만 느껴졌던 비지도 학습을 직접 코드로 구현하고, 데이터 속에서 의미 있는 고객 그룹을 찾아내는 과정이 정말 흥미로웠다. 

특히 `Elbow Method`와 `Silhouette Score`를 통해 최적의 클러스터 개수를 찾아가는 과정은 데이터 기반의 의사결정이 왜 중요한지를 다시 한번 깨닫게 해주었다.

이제 각 클러스터의 페르소나를 정의하고, 그들에게 맞는 상품 추천이나 마케팅 캠페인을 기획하는 다음 단계로 나아갈 수 있을 것 같다. 

데이터 분석이 단순히 숫자를 다루는 것을 넘어, 비즈니스의 방향을 제시하는 강력한 도구가 될 수 있다는 것을 체감했다! 😄