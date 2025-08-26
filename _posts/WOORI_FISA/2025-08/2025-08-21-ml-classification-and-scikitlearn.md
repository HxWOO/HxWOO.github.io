---
layout: post
title:  "🤖 Scikit-Learn으로 떠나는 머신러닝 분류 탐험기 (코드 보강판)"
date:   2025-08-21 10:00:00 +0900
categories: [Machine Learning, Woori FISA]
tags: [머신러닝, 분류, Scikit-learn, 퍼셉트론, 로지스틱 회귀, SVM, 결정 트리, KNN, 규제, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🚀 머신러닝의 세계, 분류(Classification)로 첫발 내딛기

머신러닝의 세계에 첫발을 내딛는 당신, 어디부터 시작해야 할지 막막한가요? 오늘은 그 첫걸음으로, 수많은 모델의 기초가 되는 **분류(Classification)** 알고리즘들을 **Scikit-Learn**이라는 강력한 무기와 함께 정복해본 경험을 공유하려 한다. 딱딱한 이론서를 넘어, 직접 코드를 만져보며 각 알고리즘의 심장을 들여다보는 여정이었다. 😄

> ### 💡 "이메일이 스팸일까, 아닐까?"
> 우리 주변의 수많은 AI 서비스는 바로 이 '분류'에서 시작된다. 데이터를 보고 '이것'과 '저것'으로 나누는 지혜, 즉 분류의 원리를 이해하는 것은 머신러닝의 바다를 항해하기 위한 필수 나침반과도 같다.

---

### 🛠️ 1. 우리의 든든한 장비, Scikit-Learn

탐험을 떠나기 전, 최고의 장비부터 챙겨야 한다. Scikit-learn은 마치 잘 만들어진 스위스 아미 나이프처럼, 다양한 알고리즘을 일관된 방식으로 제공해준다.

![scikit-learn logo](https://scikit-learn.org/stable/_static/scikit-learn-logo-small.png)

**Scikit-learn 사용법은 딱 5단계만 기억하면 된다.**

1.  **모델 선택**: `from sklearn.some_module import SomeModel`
2.  **모델 생성**: `model = SomeModel(hyperparameter=value)`
3.  **데이터 준비**: `X_train, y_train`
4.  **훈련**: `model.fit(X_train, y_train)` ✨
5.  **예측**: `model.predict(X_new)`

이 간단한 패턴만 익히면, 퍼셉트론부터 SVM까지 모든 모델을 같은 방식으로 다룰 수 있다. 정말 멋지지 않은가!

---

### 🗺️ 2. 본격적인 탐험: 6개의 분류 알고리즘 정복기

이제 본격적으로 6개의 주요 분류 알고리즘을 하나씩 만나보았다.

#### (1) 퍼셉트론 (Perceptron): 가장 단순한 시작
가장 기본적인 선형 분류기. `sklearn.linear_model.Perceptron`으로 쉽게 구현할 수 있다. 선형적으로 분리 가능한 문제에서는 여전히 강력한 모습을 보여준다.
```python
from sklearn.linear_model import Perceptron

# 퍼셉트론 훈련
ppn = Perceptron(eta0=0.1, random_state=1)
ppn.fit(X_train_std, y_train)
```

#### (2) 로지스틱 회귀 (Logistic Regression): 확률의 세계로
이름은 '회귀'지만, 사실은 분류 모델이다. 각 클래스에 속할 **확률**을 계산해주는 점이 매력적이다. 이 과정에서 모델이 훈련 데이터에만 너무 익숙해지는 **과대적합(Overfitting)** 이라는 문제를 만났다.

> #### 🧐 발견! 과대적합을 막는 마법, 규제(Regularization)
> 모델이 너무 복잡해지지 않도록 **페널티**를 주는 기술이다. Scikit-learn에서는 `LogisticRegression(C=1.0, penalty='l2')`처럼 간단한 파라미터로 이 마법을 부릴 수 있다. `C` 값이 작을수록 강한 규제가 적용되어 모델이 더 단순해진다. 이 원리는 다른 많은 모델에서도 사용되는 아주 중요한 개념이었다.

```python
from sklearn.linear_model import LogisticRegression

# L2 규제를 사용하는 로지스틱 회귀 훈련
lr = LogisticRegression(C=1.0, penalty='l2', random_state=1)
lr.fit(X_train_std, y_train)
```

#### (3) 서포트 벡터 머신 (SVM): 최적의 경계선을 찾아서
클래스 간의 **마진(간격)을 최대화**하는 가장 안정적인 경계선을 찾는 똑똑한 알고리즘. 특히 **커널 트릭**을 사용하면 선형으로 분리할 수 없는 복잡한 데이터도 거뜬히 분류해낸다. `SVC(kernel='rbf')`가 바로 그 마법의 주문이다.
```python
from sklearn.svm import SVC

# RBF 커널을 사용하는 SVM 훈련
svm = SVC(kernel='rbf', random_state=1, gamma=0.10, C=10.0)
svm.fit(X_train_std, y_train)
```

#### (4) 결정 트리 (Decision Tree): 스무고개 하듯 명쾌하게
"이 특징이 A보다 큰가요?" 같은 질문을 반복하며 데이터를 분류하는, 마치 스무고개 같은 모델이다. 모델의 결정 과정을 사람이 쉽게 이해할 수 있다는 엄청난 장점이 있다. 하지만 너무 깊게 파고들면 과대적합의 덫에 빠지기 쉬우니 `max_depth`로 깊이를 조절하는 지혜가 필요하다.
```python
from sklearn.tree import DecisionTreeClassifier

# 결정 트리 훈련 (최대 깊이 4로 제한)
tree = DecisionTreeClassifier(criterion='gini', max_depth=4, random_state=1)
tree.fit(X_train, y_train)
```

#### (5) 랜덤 포레스트 (Random Forest): 집단지성의 힘
결정 트리가 한 명의 전문가라면, 랜덤 포레스트는 수많은 전문가 집단이다. 여러 개의 결정 트리를 만들어 투표(앙상블)를 하니, 성능이 훨씬 안정적이고 강력해진다. 역시 함께할 때 더 강하다! 🧠
```python
from sklearn.ensemble import RandomForestClassifier

# 100개의 트리로 구성된 랜덤 포레스트 훈련
forest = RandomForestClassifier(n_estimators=100, random_state=1)
forest.fit(X_train, y_train)
```

#### (6) K-최근접 이웃 (KNN): "친구를 보면 너를 알 수 있다"
새로운 데이터가 들어오면, 주변의 가장 가까운 k개 이웃들을 보고 클래스를 결정하는 단순하고 직관적인 알고리즘. 모델을 미리 만들어두지 않아 '게으른 학습'이라고도 불린다.

```python
from sklearn.neighbors import KNeighborsClassifier

# 5개의 이웃을 참고하는 KNN 훈련
# 특징을 표준화(Standardization)하는 전처리가 매우 중요하다!
knn = KNeighborsClassifier(n_neighbors=5, p=2, metric='minkowski')
knn.fit(X_train_std, y_train)
```
---

### ✨ 오늘의 회고

단순히 Scikit-learn의 함수를 사용하는 것을 넘어, 각 분류 모델이 어떤 원리로 동작하고, 왜 특정 상황에서 특정 파라미터(규제, 커널 등)가 중요한지 몸소 체감하는 시간이었다. 😊

오늘 배운 6개의 알고리즘은 앞으로 마주할 더 복잡한 모델들의 든든한 기초가 되어줄 것이다. 다음에는 오늘 배운 모델들을 활용해 실제 캐글 데이터셋으로 예측 모델을 만들어 볼 생각에 가슴이 뛴다! 🔥
