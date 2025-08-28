---
layout: post
title:  "PCA부터 t-SNE까지, 차원 축소 완전 정복! 🚀"
date:   2025-08-28 20:00:00 +0900
categories: [Machine_Learning, Woori FISA]
tags: [PCA, t-SNE, 비지도학습, 차원축소, Manifold Learning, Scikit-learn, Python, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 😵‍💫 차원의 저주, 데이터가 너무 많아도 문제?

데이터의 세계는 종종 너무 많은 정보로 가득 찬 거대한 도서관과 같다. 모든 책(특성)을 다 읽으려다 길을 잃기 십상이다. 모델 학습도 마찬가지다. 특성(차원)이 너무 많아지면 오히려 성능이 떨어지는 **'차원의 저주'** 에 걸리게 된다.

> #### 💡 "차원이 늘어날수록 데이터 공간은 급격히 넓어지고, 데이터는 희소(sparse)해진다."
> 모델이 학습할 데이터는 한정적인데, 데이터가 존재하는 공간만 넓어지니 학습이 제대로 될 리가 없다. 연산량 증가는 덤이다.

이 저주를 풀기 위한 마법이 바로 **차원 축소**이고, 오늘 우리는 그중 가장 강력한 마법인 **주성분 분석(PCA)** 과 시각화의 달인 **t-SNE**를 배웠다.

---

### 🧙‍♂️ 주성분 분석(PCA), 데이터의 본질을 찾아서

PCA는 데이터에 흩어져 있는 여러 특성들의 상관관계를 파악하고, 이들을 가장 잘 설명하는 새로운 축, 즉 **주성분(Principal Component)** 을 찾아내는 기법이다. 단순히 몇몇 특성을 버리는 것이 아니라, 기존 특성들을 조합하여 데이터의 분산을 가장 잘 보존하는 새로운 특성을 만들어내는 것이다.

> 선형대수학의 관점에서 PCA는 데이터의 **공분산 행렬**을 **고유값 분해**하는 것과 같다. 이때 얻어지는 **고유벡터**가 데이터의 분산이 가장 큰 방향을 나타내는 주성분이 된다. 어렵게 들리지만, **'데이터를 가장 잘 설명하는 새로운 축을 찾는다'** 는 개념만 기억해도 충분하다!

---

### 🚀 붓꽃 데이터셋으로 PCA 실습하기

백문이 불여일견! `Scikit-learn`의 붓꽃 데이터셋으로 PCA의 위력을 직접 확인해봤다. 4개의 특성을 2개로 줄여 원본과 성능을 비교했다.

#### 1. 데이터 준비 및 스케일링

먼저 데이터를 불러와 표준화를 진행했다. PCA는 데이터의 스케일에 영향을 받기 때문에 스케일링은 필수적인 전처리 과정이다.

```python
import numpy as np
import pandas as pd
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

# 데이터 로드 및 스케일링
iris = load_iris()
scaler = StandardScaler()
iris_scaled = scaler.fit_transform(iris.data)
```

#### 2. PCA 적용 및 성능 비교

4개의 특성을 단 2개의 주성분으로 줄여봤다.

```python
# 2차원으로 PCA 변환
pca = PCA(n_components=2)
iris_pca = pca.fit_transform(iris_scaled)

# 랜덤 포레스트 분류기로 성능 비교
rcf = RandomForestClassifier(random_state=156)

# 원본 데이터 교차 검증
scores = cross_val_score(rcf, iris.data, iris.target, scoring='accuracy', cv=3)
print(f'원본 데이터 평균 정확도: {np.mean(scores):.2f}')

# PCA 데이터 교차 검증
scores_pca = cross_val_score(rcf, iris_pca, iris.target, scoring='accuracy', cv=3)
print(f'PCA 변환 데이터 평균 정확도: {np.mean(scores_pca):.2f}')
```

> **결과:** 원본 데이터 평균 정확도는 `0.96`, PCA 변환 데이터 평균 정확도는 `0.89`가 나왔다.
> 4개의 특성을 단 2개로 줄였는데도 성능 저하가 크지 않았다. PCA가 정보 손실을 최소화하며 차원을 효과적으로 축소했음을 보여준다. 성공! 🎉

---

### 🎨 매니폴드 학습과 t-SNE로 시각화하기

PCA가 데이터의 전체적인 구조를 보존하는 선형적인 방법이라면, **t-SNE**는 이웃 데이터 간의 관계를 보존하며 고차원 데이터를 2차원이나 3차원으로 시각화하는 데 탁월한 비선형 기법이다.

`Scikit-learn`의 숫자 데이터셋(64차원)을 t-SNE로 시각화해봤다.

```python
from sklearn.datasets import load_digits
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt
import matplotlib.patheffects as PathEffects

# 데이터 로드 및 t-SNE 변환
digits = load_digits()
tsne = TSNE(n_components=2, init='pca', random_state=123)
X_digits_tsne = tsne.fit_transform(digits.data)

# t-SNE 결과 시각화
def plot_projection(x, colors):
  f = plt.figure(figsize=(8,8))
  ax = plt.subplot(aspect='equal')
  for i in range(10):
    plt.scatter(x[colors==i, 0], x[colors==i, 1])
  for i in range(10):
    xtext, ytext = np.median(x[colors==i, :], axis=0)
    txt = ax.text(xtext, ytext, str(i), fontsize=24)
    txt.set_path_effects([PathEffects.Stroke(linewidth=5, foreground="w"), PathEffects.Normal()])

plot_projection(X_digits_tsne, digits.target)
plt.show()
```
결과를 보니, 같은 숫자끼리 옹기종기 모여 군집을 이루는 모습이 정말 신기했다. 데이터의 숨겨진 구조를 한눈에 파악할 수 있었다.

---

### 🛠️ 파이프라인으로 PCA와 모델링을 한번에!

위스콘신 유방암 데이터셋(특성 30개)을 `make_pipeline`으로 전처리, 차원 축소, 모델 학습을 한번에 처리했다.

```python
from sklearn.datasets import load_breast_cancer
from sklearn.pipeline import make_pipeline
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

# 데이터 로드 및 분할
cancer = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    cancer.data, cancer.target, test_size=0.2, random_state=42
)

# 파이프라인 생성 및 학습
pipe = make_pipeline(StandardScaler(), PCA(n_components=2), LogisticRegression())
pipe.fit(X_train, y_train)

print(f"Train Score: {pipe.score(X_train, y_train):.4f}")
print(f"Test Score: {pipe.score(X_test, y_test):.4f}")
```
> **결과:** 30개의 특성을 단 2개로 줄였는데도 테스트 정확도가 **97.37%** 라는 놀라운 성능을 보여줬다. 파이프라인 덕분에 코드가 훨씬 간결하고 체계적으로 변했다.

---

### ✨ 오늘의 회고

오늘은 비지도 학습의 대표적인 차원 축소 기법인 PCA와 t-SNE를 깊이 있게 배웠다. 단순히 특성 개수를 줄이는 것을 넘어, 데이터의 분산을 최대한 보존하는 '주성분'을 찾는 PCA의 원리와, 데이터의 국소적 구조를 보존하며 시각화하는 t-SNE의 강력함을 직접 확인했다.

붓꽃, MNIST, 유방암 데이터셋 등 다양한 실습을 통해 차원을 크게 줄여도 모델 성능이 크게 저하되지 않는 것을 보고 차원 축소의 위력을 실감했다. 특히 파이프라인을 사용하니 복잡한 머신러닝 워크플로우를 효율적으로 관리할 수 있었다.

앞으로 복잡한 데이터셋을 마주했을 때, 무작정 모델에 넣기 전에 PCA와 t-SNE를 통해 데이터의 본질을 파악하고 효율적으로 분석하는 습관을 들여야겠다. 차원의 저주, 이젠 두렵지 않다! 😄