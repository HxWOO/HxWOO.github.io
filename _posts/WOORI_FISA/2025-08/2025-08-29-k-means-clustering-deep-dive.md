---
layout: post
title:  "🫂 K-평균(K-Means) 알고리즘, 데이터 속 숨은 보석 찾기"
date:   2025-08-29 10:00:00 +0900
categories: [Machine_Learning, Woori FISA]
tags: [K-Means, Clustering, Python, Scikit-learn, ML, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🤖 데이터 속 숨은 그룹을 찾아내는 K-평균 알고리즘

오늘은 비지도 학습의 대표적인 군집화 알고리즘, **K-평균(K-Means)** 에 대해 깊이 파고들었다. 마치 정리되지 않은 거대한 도서관에서 비슷한 주제의 책들을 찾아 서가를 정리하는 사서처럼, K-평균은 데이터의 숨겨진 구조를 찾아 그룹으로 묶어주는 멋진 역할을 한다.

> #### ✨ "데이터의 친구들을 찾아주자!"
> K-평균을 사용하면 정답이 없는 데이터 속에서도 의미 있는 패턴과 그룹을 발견할 수 있다.

---

### 🛠️ K-평균(K-Means) vs. K-최근접 이웃(K-NN)

이름이 비슷해서 헷갈릴 수 있지만, 두 알고리즘은 완전히 다른 목표를 가지고 있다.

| 구분 | **K-평균 (K-Means)** | **K-최근접 이웃 (K-NN)** |
| :--- | :--- | :--- |
| **학습 방식** | 비지도학습 (Clustering) | 지도학습 (Classification) |
| **목표** | 레이블 없는 데이터를 K개의 군집으로 나눔 | 새 데이터의 클래스를 K개의 이웃을 통해 결정 |
| **공통점** | K개의 점을 지정하고, 거리를 기반으로 동작 |

---

### ⚙️ K-평균은 어떻게 동작할까?

K-평균은 중심점(Centroid)이라는 특별한 점을 계속 이동시키면서 최적의 군집을 찾아낸다.

1.  **초기화**: K개의 중심점을 임의의 위치에 배치한다.
2.  **군집 할당**: 각 데이터를 가장 가까운 중심점에 할당한다.
3.  **중심점 업데이트**: 각 군집의 중심으로 중심점을 이동시킨다.
4.  **반복**: 중심점이 더 이상 움직이지 않을 때까지 2, 3번 과정을 반복하면, 마침내 최적의 군집이 탄생한다! 🐜

---

### 🐍 Scikit-learn으로 K-평균 구현하기

붓꽃(Iris) 데이터셋으로 직접 K-평균 군집화를 실습해봤다.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_iris
from sklearn.cluster import KMeans

# 1. 데이터 로드
iris = load_iris()
irisDF = pd.DataFrame(iris.data, columns=iris.feature_names)

# 2. K-평균 모델 학습 (K=3)
kmeans = KMeans(n_clusters=3, random_state=121)
kmeans.fit(irisDF)

# 3. 군집 결과 확인
print(kmeans.labels_)
# [1 1 1 ... 0 0 0]
```

> `inertia_` 속성은 클러스터 내 데이터들이 얼마나 잘 뭉쳐있는지를 나타내는데, K가 커질수록 무조건 작아져서 최적의 K를 판단하는 절대적인 기준이 되기는 어렵다. 🤔

#### 최적의 K 찾기: 엘보우(Elbow) 기법

그렇다면 최적의 K는 어떻게 찾을까? 바로 **엘보우 기법**을 사용했다. K를 늘려가며 `inertia` 값의 변화를 그래프로 그렸을 때, 팔꿈치처럼 급격히 꺾이는 지점이 바로 최적의 K가 된다.

```python
# 엘보우 기법으로 최적의 K 찾기
inertias = []
K_range = range(1, 12)

for k in K_range:
    km = KMeans(n_clusters=k, random_state=121)
    km.fit(iris.data)
    inertias.append(km.inertia_)

# 그래프 시각화
plt.plot(K_range, inertias, '-o')
plt.xlabel('Number of clusters, K')
plt.ylabel('Inertia')
plt.show()
```
그래프를 보니 3에서 기울기가 완만해지는 것을 확인했다.

---

### 👍 K-평균의 장점과 단점 👎

| 장점 | 단점 |
| :--- | :--- |
| 이해와 구현이 쉽고 직관적이다. | **K값을 직접 설정**해야 한다. 😭 |
| 수렴이 보장된다. | **거리 기반**이라 차원이 많아지면 복잡도가 증가한다. |
| 대용량 데이터에도 적용 가능하다. | **이상치(Outlier)와 스케일**에 민감하다. |
| - | 초기 중심점 위치에 따라 결과가 달라질 수 있다. |

---

### 👍 성능 향상을 위한 기법들

K-평균의 단점을 보완하고 성능을 끌어올리기 위한 방법들도 적용해봤다.

-   **PCA (차원 축소)**: 고차원 데이터의 계산 복잡도를 줄여주었다.
-   **스케일링 (Scaling)**: `StandardScaler`로 피처 스케일을 맞추니 이상치에 대한 민감도를 줄일 수 있었다. 

```python
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

# 스케일링
scaler = StandardScaler()
X_scaled = scaler.fit_transform(iris.data)

# PCA로 2차원으로 축소
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

# K-Means 다시 학습
kmeans_pca = KMeans(n_clusters=3, random_state=121)
kmeans_pca.fit(X_pca)
```

---

### 📊 군집화, 얼마나 잘 됐을까? (성능 평가)

#### 1. 실루엣 계수 (Silhouette Coefficient)
-   **정답 레이블이 없을 때** 사용하는 지표다.
-   군집 내 데이터는 얼마나 가깝고, 다른 군집과는 얼마나 멀리 떨어져 있는지를 측정한다.
-   **1에 가까울수록** 좋은 군집화로 판단했다.

```python
from sklearn.metrics import silhouette_score

score = silhouette_score(iris.data, kmeans.labels_)
print('실루엣 점수: {0:.3f}'.format(score))
# 실루엣 점수: 0.553
```

#### 2. Homogeneity, Completeness, V-measure
-   **정답 레이블이 있을 때** 사용했다.
-   **Homogeneity (균질성)**: 각 군집이 동일한 실제 클래스로만 구성된 정도.
-   **Completeness (완전성)**: 실제 클래스의 데이터들이 모두 동일한 군집에 속한 정도.
-   **V-measure**: 위 두 값의 조화 평균이다.

```python
from sklearn.metrics import homogeneity_score, completeness_score, v_measure_score

print("균질성:", homogeneity_score(iris.target, kmeans.labels_))
print("완전성:", completeness_score(iris.target, kmeans.labels_))
print("V-measure:", v_measure_score(iris.target, kmeans.labels_))
```

---

### 🧐 K-평균의 대안: 밀도 기반 클러스터링 (DBSCAN)

K-평균은 원형 군집에 강하지만, 복잡한 모양의 군집은 잘 찾아내지 못한다. 이럴 때 **DBSCAN**을 사용할 수 있다.

-   데이터가 밀집된 지역을 찾아 군집화하는 방식이다.
-   K-평균과 달리 **클러스터 개수를 미리 정할 필요가 없고**, 노이즈(이상치)를 자동으로 분류해줘서 편리했다.

```python
from sklearn.cluster import DBSCAN

dbscan = DBSCAN(eps=0.6, min_samples=8)
dbscan_labels = dbscan.fit_predict(iris.data)

# -1은 노이즈(이상치)로 분류된 데이터
print(np.unique(dbscan_labels, return_counts=True))
```

---

### ✨ 오늘의 회고

K-평균 알고리즘은 간단하면서도 강력한 군집화 기법이었다. 특히 엘보우 기법으로 최적의 K를 찾아내는 과정이 흥미로웠다. 하지만 K값을 직접 정해야 하고, 이상치에 민감하다는 단점도 명확히 알게 되었다.

다음에는 오늘 배운 **DBSCAN**처럼 밀도 기반 클러스터링이나 다른 군집화 알고리즘을 더 깊이 파고들어, 데이터의 특성에 맞는 최적의 방법을 선택하는 능력을 길러야겠다. 😄
