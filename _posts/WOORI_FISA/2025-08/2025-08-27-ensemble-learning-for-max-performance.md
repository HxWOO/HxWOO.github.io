---
layout: post
title:  "🤖 집단지성의 힘, 앙상블 학습으로 모델 성능 극대화하기"
date:   2025-08-27 20:00:00 +0900
categories: [Woori FISA, Machine_Learning]
tags: [ensemble, voting, bagging, boosting, random-forest, xgboost, 앙상블, 머신러닝, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 📜 백지장도 맞들면 낫다!

"백지장도 맞들면 낫다"는 속담이 있다. 머신러닝의 세계에도 이 속담이 그대로 적용된다. 바로 **앙상블(Ensemble) 학습** 기법을 통해서다. **앙상블**은 **여러 개의 모델(분류기)을 연결**하여 단일 모델보다 더 똑똑하고 강력한 하나의 모델을 만드는, 말 그대로 집단지성의 힘을 빌리는 방법이다.

오늘은 이 앙상블의 세계를 깊이 탐험해 보았다. 가장 기본적인 **투표 방식**부터, 안정적인 성능을 자랑하는 **랜덤 포레스트**, 그리고 캐글과 같은 경진대회에서 왕좌를 차지하고 있는 **부스팅** 모델들까지! 여러 모델을 조화롭게 사용해 예측 성능을 극대화하는 여정을 진행했다.

> #### ✨ "하나의 천재보다 여러 명의 전문가가 낫다."
> 앙상블 학습은 각기 다른 강점을 가진 여러 모델의 의견을 종합하여 더 신중하고 정확한 결정을 내리는, 현실 세계의 전문가 위원회와도 같다.

---

### 🗳️ 1. Voting: 가장 간단한 집단지성

가장 직관적인 앙상블 방법은 **투표(Voting)** 이다. 여러 모델이 각자 예측한 결과를 가지고 다수결로 최종 결정을 내리는 방식이다.

| 구분 | **Hard Voting** | **Soft Voting** |
| :--- | :--- | :--- |
| **방식** | 단순 다수결 투표 | 각 모델의 예측 확률의 평균을 내어 결정 |
| **특징** | 구현이 간단하다 | 일반적으로 성능이 더 우수하다 |

유방암 데이터셋을 3개의 다른 모델(로지스틱 회귀, KNN, 결정 트리)로 학습시킨 뒤, VotingClassifier로 묶어 성능을 확인해 보았다.

```python
from sklearn.ensemble import VotingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.neighbors import KNeighborsClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split

# 데이터 로드 및 분할
cancer = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(cancer.data, cancer.target, random_state=121, test_size=0.2)

# 개별 모델 및 VotingClassifier 생성
lr_clf = LogisticRegression(max_iter=10000)
knn_clf = KNeighborsClassifier()
dt_clf = DecisionTreeClassifier()

vo_clf_hard = VotingClassifier([('LR', lr_clf), ('KNN', knn_clf), ('DT', dt_clf)], voting="hard")

# 모델 학습 및 평가
vo_clf_hard.fit(X_train, y_train)
print(f"Hard Voting Accuracy: {vo_clf_hard.score(X_test, y_test)}")
```

> **결과**: Hard Voting으로 `97.3%`의 정확도를 얻었다. 각 모델을 단독으로 사용했을 때보다 안정적인 결과를 보여주었다. 👍

---

### 🌳 2. Bagging & Random Forest: 같은 모델, 다른 생각

**배깅(Bagging)** 은 같은 종류의 모델 여러 개에게 약간씩 다른 데이터를 주어 학습시키는 방식이다. 데이터의 일부를 **복원추출(Bootstrap)** 하여 여러 개의 데이터셋을 만들고, 각 모델이 이 데이터셋으로 병렬 학습을 진행한다. 이 과정을 통해 개별 모델의 약점은 보완되고 전체 모델의 과적합은 줄어든다.

이 배깅 방식의 대표주자가 바로 **랜덤 포레스트(Random Forest)** 이다.

```python
from sklearn.ensemble import RandomForestClassifier

# RandomForest 모델 생성 및 학습
rf_clf = RandomForestClassifier(random_state=121, n_estimators=100) # 100개의 결정 트리를 사용
rf_clf.fit(X_train, y_train)

print(f"Random Forest Accuracy: {rf_clf.score(X_test, y_test)}")
```

> **결과**: 무려 `98.2%`의 정확도를 달성했다! 별다른 튜닝 없이도 매우 강력한 성능을 보여주는, 정말 든든한 모델이다. 🎉

---

### ✨ 3. Boosting: 선배의 실수를 교훈 삼아

**부스팅(Boosting)** 은 약한 모델들을 순차적으로 학습시키는 점이 배깅과 다르다. 첫 번째 모델이 예측하고 나면, 그 모델이 틀린 문제에 가중치를 부여하여 다음 모델이 더 집중적으로 학습하게 만든다. 이렇게 선배의 실수를 교훈 삼아 계속 발전해나가는 방식이다.

#### **가. GBM (Gradient Boosting Machine)**
경사하강법을 이용해 오차를 보완해나가는 기본적인 부스팅 모델이다.

#### **나. XGBoost (eXtra Gradient Boosting)**
GBM의 성능을 극대화한 모델로, 병렬 처리(GPU) 지원, 과적합 방지 규제, 조기 중단 기능 등을 추가하여 속도와 성능을 모두 잡았다. 캐글과 같은 데이터 경진대회에서 "일단 XGBoost부터 쓰고 본다"는 말이 있을 정도라고 한다.

```python
from xgboost import XGBClassifier

# XGBoost 모델 생성 및 조기 중단 설정
xgbc = XGBClassifier(n_estimators=1000, early_stopping_rounds=10, eval_metric='logloss')
xgbc.fit(X_train, y_train, eval_set=[(X_test, y_test)], verbose=False)

print(f"XGBoost Accuracy: {xgbc.score(X_test, y_test)}")
```

> **결과**: 역시 `98.2%`! 랜덤 포레스트와 동일한 최고 성능을 보여주었다. 조기 중단 기능 덕분에 불필요한 학습을 막아 효율적이었다.

#### **다. LightGBM**
XGBoost보다 더 빠르고 가벼운 모델을 목표로 개발되었다. 리프 중심 트리 분할 방식을 사용해 대용량 데이터 처리에서 특히 강점을 보인다.

---

### ✨ 오늘의 회고

오늘은 **앙상블 기법**을 공부하며 모델 간 집단지성의 힘을 직접 확인했다. **Voting, Bagging, Boosting** 이라는 세 가지 큰 축을 중심으로, 각 기법이 어떻게 모델의 성능을 끌어올리는지 배울 수 있었다. 특히 단일 모델의 성능에만 매달리는 것이 아니라, 여러 모델을 현명하게 조합하는 **'아키텍처' 설계**가 얼마나 중요한지 깨달았다.

랜덤 포레스트의 안정성과 XGBoost의 강력함이 특히 인상 깊었다. 왜 많은 현업 데이터 사이언티스트들이 이 모델들을 사랑하는지 알 것 같았다.! 😄
