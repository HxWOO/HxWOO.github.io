---
layout: post
title:  "🩺 머신러닝 모델 건강검진: 전처리와 성능 평가 톺아보기"
date:   2025-08-25 10:00:00 +0900
categories: [Woori FISA, Machine Learning]
tags: [전처리, 성능평가, 교차검증, 과대적합, Scikit-learn, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## :bread: 모델, 혹시 암기만 하고 있니?

열심히 만든 머신러닝 모델이 학습 데이터에서는 100점인데, 막상 실전에 투입하니 엉뚱한 답을 내놓는다면 어떨까? 마치 시험 범위만 달달 외우고 응용 문제는 하나도 풀지 못하는 학생 같았다. 이런 현상을 **과대적합(Overfitting)** 이라고 한다. 반대로 너무 공부를 안 해서 패턴조차 파악하지 못하면 **과소적합(Underfitting)** 이다.

> #### 💡 "모델의 진짜 실력은 새로운 데이터에서 나타난다!"
> 모델이 학습 데이터에만 치우치지 않고 일반적인 성능을 갖도록 하는 것, 바로 **교차 검증**과 **데이터 전처리**가 그 해답이었다. 오늘 내가 모델의 건강 상태를 꼼꼼히 진단하고 컨디션을 최상으로 끌어올렸던 방법을 공유해본다.

---

### 🚀 신뢰도를 높이는 훈련법: 교차 검증

데이터를 여러 조각으로 나눠서, 모델을 여러 번 테스트하고 평균 성능을 평가하는 아주 스마트한 방법이었다. 특히 **K-폴드 교차 검증**은 데이터를 K개의 덩어리(폴드)로 나누고, 하나씩 돌아가며 테스트용으로 사용해서 데이터가 부족해도 신뢰도 높은 검증을 가능하게 했다.

```python
# K-Fold로 교차 검증을 직접 구현해봤다.
from sklearn.model_selection import KFold
from sklearn.metrics import accuracy_score
import numpy as np

# 예시 데이터와 모델이 있다고 가정했다.
# X, y, dt_clf

kfold = KFold(n_splits=5)
cv_accuracy = []

for train_index, test_index in kfold.split(X):
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]

    dt_clf.fit(X_train, y_train)
    pred = dt_clf.predict(X_test)

    accuracy = np.round(accuracy_score(y_test, pred), 4)
    cv_accuracy.append(accuracy)

print(f'K-Fold 평균 정확도: {np.mean(cv_accuracy):.4f} 성공! 🎉')
```

> 데이터를 알차게 활용해서 모델의 일반화 성능을 측정할 수 있었다. 이제 내 모델은 더 이상 우물 안 개구리가 아니다!:frog:

---

### ✨ 데이터 성형수술: 전처리(Preprocessing)

**"Garbage in, Garbage out."** 이라는 말이 있다. 좋은 모델은 좋은 데이터에서 나오는 법. 모델이 잘 이해할 수 있도록 데이터를 깔끔하게 다듬어주는 전처리 기법들을 알아봤다.

#### 1. 레이블 인코딩 (Label Encoding)
`'TV'`, `'냉장고'` 같은 문자열 데이터를 숫자로 바꿔주지만, 숫자 크기에 의미가 부여될 수 있어 주의가 필요했다. (e.g., `TV(0) < 냉장고(1)`)

```python
from sklearn.preprocessing import LabelEncoder

items = ['TV', '냉장고', '전자레인지', '컴퓨터', '선풍기', '선풍기', '믹서', '믹서']
encoder = LabelEncoder()
labels = encoder.fit_transform(items)
# 인코딩 변환값: [0 1 4 5 3 3 2 2]
```

#### 2. 원-핫 인코딩 (One-Hot Encoding)
위 단점을 보완하기 위해 각 카테고리를 새로운 피처(열)로 만든다. 해당하면 1, 아니면 0으로 표시하는 것이다. 피처가 너무 많아지는 단점은 감수해야 했다.

```python
from sklearn.preprocessing import OneHotEncoder

# 2차원 배열로 변환해야 했다.
items_2d = np.array(items).reshape(-1, 1)
oh_encoder = OneHotEncoder()
oh_labels = oh_encoder.fit_transform(items_2d)
# print(oh_labels.toarray())
```

#### 3. 피처 스케일링 (Feature Scaling)
키(cm)와 몸무게(kg)처럼 단위가 다른 피처들의 값 범위를 맞춰주는 작업이다.

| 구분 | **StandardScaler** | **MinMaxScaler** |
| :--- | :--- | :--- |
| **개념** | 평균 0, 분산 1의 정규분포 형태로 변환 | 모든 값을 0과 1 사이로 조정 |
| **특징** | 이상치에 상대적으로 덜 민감 | 이상치에 민감하게 반응할 수 있음 |

```python
# 붓꽃 데이터로 스케일링을 직접 해보니 데이터 분포가 바뀌는 게 신기했다.
from sklearn.preprocessing import StandardScaler, MinMaxScaler

# iris 데이터가 로드되어 있다고 가정했다.
std_scaler = StandardScaler()
iris_scaled_std = std_scaler.fit_transform(iris.data)

minmax_scaler = MinMaxScaler()
iris_scaled_minmax = minmax_scaler.fit_transform(iris.data)
```

---

### 📊 모델 성적표 제대로 읽기: 성능 측정 지표

정확도(Accuracy)만 믿으면 큰코다칠 수 있다. 특히 데이터가 불균형할 때 그렇다. 모델의 진짜 실력을 다각도로 평가하는 지표들을 알아봤다.

-   **오차 행렬 (Confusion Matrix)**: 모델이 뭘 맞추고 뭘 헷갈리는지 한눈에 보여주는 표다. 모든 지표의 어머니!
-   **정밀도 (Precision)**: "모델이 `정답`이라고 예측한 것 중 진짜 `정답` 비율"이다. (Positive 예측 성능)
-   **재현율 (Recall)**: "실제 `정답` 중 모델이 `정답`이라고 맞춘 비율"이다. (놓치면 안 되는 것을 찾아내는 능력)
-   **F1 스코어 (F1 Score)**: 정밀도와 재현율의 조화 평균. 둘 다 중요할 때 빛을 발한다.
-   **ROC 곡선과 AUC**: 모델이 정답과 오답을 얼마나 잘 구분하는지 보여주는 곡선과 그 아래 면적. AUC가 1에 가까울수록 완벽한 모델이다.

> 암 환자 예측 모델에서, 실제 환자(True)를 놓치지 않는 것(Recall)이 중요할까, 아니면 일반인(False)을 환자로 잘못 예측하지 않는 것(Precision)이 중요할까? 이처럼 상황에 맞는 지표 선택이 필수라는 걸 느꼈다.

---

### ✨ 오늘의 회고

오늘 학습으로 모델의 성능을 제대로 측정하고 개선하는 기본기를 다졌다. 과대적합을 피하기 위한 교차 검증의 중요성과 데이터 특성에 맞는 전처리 기법을 적용하는 법도 배웠다.

특히 **정확도**라는 하나의 숫자 뒤에 숨겨진 모델의 진짜 성능을 **오차 행렬, 정밀도, 재현율, AUC** 등 다양한 지표로 다각적으로 평가해야 한다는 점이 가장 중요하게 다가왔다. 앞으로 모델을 만들 때, 이 건강검진표들을 꼼꼼히 확인하며 더 신뢰도 높은 AI를 만들어 나가야겠다고 다짐했다. 😄
