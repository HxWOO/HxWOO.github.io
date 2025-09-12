---
layout: post
title:  "⛑️ Isolation Forest와 SMOTE로 사기 거래 탐지기 성능 끌어올리기!"
date:   2025-09-01 10:00:00 +0900
categories: [WOORI_FISA, Machine_Learning]
tags: [IsolationForest, SMOTE, PyCaret, RandomForest, AnomalyDetection, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## ⛑️ 이상치를 찾아라! Isolation Forest 첫 만남

오늘은 비지도 학습 기반의 이상치 탐지 알고리즘, **Isolation Forest**를 공부했다. 마치 '월리를 찾아라'에서 수많은 사람 중 혼자 튀는 월리를 찾아내듯, 데이터 속에서 홀로 동떨어진 이상치를 빠르게 찾아내는 멋진 녀석이다.

> ### 🌳 "이상치는 정상 데이터보다 고립시키기 쉽다!"
> Isolation Forest의 핵심 아이디어다. 정상 데이터는 빽빽한 숲처럼 모여 있어 하나의 나무(데이터)를 분리하기 어렵지만, 이상치는 숲에서 멀리 떨어진 외톨이 나무 같아서 금방 분리할 수 있다는 원리다.

---

### 😭 하지만 현실은 쉽지 않았다: 클래스 불균형 문제

신용카드 사기 거래 데이터를 다뤄보니, Isolation Forest만으로는 한계가 명확했다. 전체 거래 28,462건 중 사기 거래는 단 30건! 😭 이런 극심한 **클래스 불균형** 상태에서는 모델이 대부분의 정상 거래만 잘 맞춰도 정확도가 높게 나와, 실제로는 사기 거래를 거의 탐지하지 못하는 문제가 발생했다.

### ✨ 구세주 등장: SMOTE 오버샘플링

이때 배운 것이 바로 **SMOTE(Synthetic Minority Over-sampling Technique)** 다. 소수 클래스(사기 거래) 데이터를 기반으로 원본과 비슷하지만 약간 다른 합성 데이터를 만들어내어 데이터의 균형을 맞춰주는 마법 같은 기법이다. SMOTE를 적용하니, 30건에 불과했던 사기 거래 데이터가 정상 거래와 동일한 28,432건으로 늘어났다.

```python
# SMOTE 적용으로 데이터 균형 맞추기
from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42)
X_resampled, y_resampled = smote.fit_resample(X, y)

print(y_resampled.value_counts())
# 0    28432
# 1    28432
# Name: Class, dtype: int64
```

---

### 🚀 AutoML(PyCaret)로 최강 모델 찾기

데이터 준비가 끝난 후, **PyCaret**으로 여러 모델의 성능을 자동으로 비교해봤다. `fix_imbalance=True` 옵션으로 데이터 불균형까지 손쉽게 처리하며 돌려본 결과, **Random Forest(`rf`)** 모델이 가장 뛰어난 성능을 보여주었다. 👍

결과를 바탕으로 SMOTE로 증강된 데이터에 Random Forest를 직접 적용해보니, 사기 거래를 탐지하는 성능(Recall)이 매우 높게 나타났다. 드디어 제대로 된 탐지기를 만든 기분이었다.

```python
# SMOTE + Random Forest 모델 학습 및 평가
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

# ... (데이터 분할) ...

rf_clf = RandomForestClassifier(n_estimators=1000, random_state=42)
rf_clf.fit(X_train, y_train)
y_pred = rf_clf.predict(X_test)

print(classification_report(y_test, y_pred))
```

> **높은 재현율(Recall) 달성!** 실제 사기 거래의 대부분을 놓치지 않고 탐지해냈다는 의미다. 이상 탐지 모델의 핵심 목표를 성공적으로 달성했다! 🥳

---

### ✨ 오늘의 회고

단순히 비지도 학습 모델 하나만 사용하는 것에서 그치지 않고, 데이터의 특성(클래스 불균형)을 파악하고 SMOTE와 같은 기법으로 데이터를 보완한 뒤, AutoML을 통해 최적의 지도 학습 모델을 찾아내는 과정이 정말 흥미로웠다. 😄

결국 좋은 모델을 만드는 것은 단순히 알고리즘을 아는 것을 넘어, **데이터를 깊이 이해하고 올바른 전략을 세우는 것**이 핵심이라는 것을 다시 한번 깨달았다. 다음에는 캐글의 불균형 데이터셋을 대상으로도 이상 탐지를 진행해 봐야겠다.
