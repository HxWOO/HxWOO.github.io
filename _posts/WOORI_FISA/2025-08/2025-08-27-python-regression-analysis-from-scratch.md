---
layout: post
title:  "📈 파이썬으로 회귀 분석 정복하기: 기초부터 AutoML까지"
date:   2025-08-27 19:30:00 +0900
categories: [WOORI FISA, Machine_Learning]
tags: [python, scikit-learn, regression, automl, pycaret, 회귀분석, 머신러닝, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🔮 데이터로 미래를 예측하는 수정 구슬

만약 우리에게 미래를 예측하는 수정 구슬이 있다면 어떨까? 주식 가격, 내년도 매출, 혹은 부동산 시세까지 예측할 수 있다면 말이다. 머신러닝에서 **회귀(Regression) 분석**이 바로 그 수정 구슬과 같은 역할을 한다. 과거 데이터의 패턴을 학습해 연속적인 미래 값을 예측하는 강력한 도구이다.

오늘은 파이썬을 이용해 이 회귀 분석의 세계를 탐험해 보았다. 단순한 개념 정리부터 시작해, 모델의 성능을 평가하고, 단계별로 개선해나가며 최종적으로 AutoML을 통해 최적의 모델을 찾아가는 여정을 생생하게 담아보았다.

> #### ✨ "회귀는 선을 긋는 것이 아니라, 데이터의 목소리를 듣는 과정이다."
> 모델의 성능을 나타내는 숫자 뒤에 숨겨진 의미를 이해하고, 더 나은 예측을 위해 모델을 담금질하는 과정이야말로 회귀 분석의 진정한 묘미라고 할 수 있다.

---

### 🧐 회귀 분석, 핵심 개념 파고들기

회귀 분석을 제대로 활용하려면 몇 가지 핵심 개념과 평가 지표를 알아야 한다.

| 구분 | **단순 선형 회귀** | **다중 선형 회귀** |
| :--- | :--- | :--- |
| **개념** | 하나의 독립변수로 종속변수를 예측 | 여러 개의 독립변수로 종속변수를 예측 |
| **수식** | `y = wx + b` | `y = w1x1 + w2x2 + ... + b` |

모델을 만들었다면, 그 성능을 제대로 평가하는 것이 중요하다. 회귀 모델에서는 주로 아래 지표들이 사용된다.

> **📝 주요 회귀 평가 지표**
> - **R² (결정계수)**: 모델이 데이터를 얼마나 잘 설명하는지를 나타내는 지표. 1에 가까울수록 좋다.
> - **MSE (평균 제곱 오차)**: 오차의 제곱에 대한 평균. 작을수록 좋다.
> - **RMSE (평균 제곱근 오차)**: MSE에 루트를 씌운 값. 실제 값과 유사한 단위를 가져 해석이 용이하다.
> - **MAE (평균 절대 오차)**: 오차의 절대값에 대한 평균. 이상치에 덜 민감하다.

이 지표들을 이해해야 우리 모델이 얼마나 똑똑한지, 혹은 어디가 부족한지 알 수 있다.

---

### 🏠 캘리포니아 주택 가격 예측 실습

실제 데이터를 가지고 모델을 만들어보았다. 90년대 캘리포니아 주택 가격 데이터를 사용해, 여러 기법으로 모델 성능을 개선해나가는 과정을 단계별로 진행했다.

#### **1단계: Baseline 모델 - 첫걸음 떼기** 🚶

아무런 전처리 없이 가장 기본적인 `LinearRegression` 모델을 적용해 보았다.

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import root_mean_squared_error
from sklearn.datasets import fetch_california_housing
import pandas as pd

# 데이터 로드 및 분할
housing = fetch_california_housing()
X = pd.DataFrame(housing.data, columns=housing.feature_names)
y = pd.Series(housing.target, name='MedHouseVal')
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=121)

# 모델 학습 및 평가
li_model = LinearRegression()
li_model.fit(X_train, y_train)
y_pred = li_model.predict(X_test)

print(f"R2 Score: {li_model.score(X_test, y_test)}")
print(f"RMSE: {root_mean_squared_error(y_test, y_pred)}")
```

> **결과**: R² 점수는 `0.62`, RMSE는 `0.70`이 나왔다. 나쁘진 않지만, 개선의 여지가 많아 보였다. 🤔

#### **2단계: Scaling & Feature Selection - 모델 다이어트** 🏃

모델 성능을 높이기 위해 스케일링을 적용하고, `SequentialFeatureSelector`로 중요한 특성만 골라내 보았다. 하지만 `LinearRegression` 모델에서는 스케일링 효과가 미미했고, 특성을 너무 적게 선택하니 오히려 성능이 떨어졌다. 실패 😭

> 특성을 무작정 줄이는 것이 능사는 아니라는 교훈을 얻었다. 데이터의 특성을 이해하는 것이 먼저였다.

#### **3단계: 규제 모델 (Ridge, Lasso) - 과적합 방지턱 넘기** 🚧

모델이 훈련 데이터에만 과하게 최적화되는 과적합을 막기 위해 규제가 있는 Ridge, Lasso 모델을 사용했다. `RandomizedSearchCV`로 최적의 `alpha` 값을 찾아 적용했다.

```python
from sklearn.linear_model import Ridge, Lasso
from sklearn.model_selection import RandomizedSearchCV
from scipy import stats

param_dist = {"alpha": stats.reciprocal(1e-1, 1e1)}

# Ridge 모델 최적화
ridge = Ridge()
ri_model_search = RandomizedSearchCV(ridge, param_distributions=param_dist, n_iter=70, scoring="neg_root_mean_squared_error", cv=5, random_state=42)
ri_model_search.fit(X_train, y_train)
ri_model = ri_model_search.best_estimator_
y_pred_ridge = ri_model.predict(X_test)

print(f"Ridge RMSE: {root_mean_squared_error(y_test, y_pred_ridge)}")
```

> **결과**: Ridge 모델이 Baseline과 비슷하거나 약간 더 나은 성능을 보였다. 규제를 통해 모델이 더 안정화된 느낌이다.

#### **4단계: AutoML - 최종 병기 등장** 🤖

수동 튜닝의 한계를 넘어, `PyCaret`과 `Optuna` 같은 AutoML 라이브러리를 사용해 최적의 모델과 하이퍼파라미터를 탐색했다.

```python
# PyCaret으로 여러 모델 비교
from pycaret.regression import *
s = setup(pd.concat([X, y], axis=1), target='MedHouseVal', session_id=123, preprocess=False, verbose=False)
best_model = compare_models()
print(best_model)
```

> `PyCaret`은 단 몇 줄의 코드로 수많은 모델을 테스트하고 가장 좋은 모델을 추천해주었다. `CatBoost`나 `ExtraTreesRegressor`가 좋은 성능을 보여주었다. 이런 자동화 도구 덕분에 모델 탐색 시간을 크게 줄일 수 있었다.

---

### ✨ 오늘의 회고

오늘은 회귀 분석의 이론부터 시작해, 실제 데이터로 모델을 만들고 점진적으로 성능을 개선하는 전 과정을 경험했다. 단순히 모델을 만들고 끝내는 것이 아니라, 왜 성능이 잘 나오지 않는지 고민하고, **스케일링, 특성 선택, 규제** 등 다양한 방법을 시도하는 과정이 정말 재미있었다.

특히 AutoML의 강력함을 체감할 수 있었다. 물론 기본 원리를 이해하는 것이 중요하지만, 이런 자동화 도구를 잘 활용하면 훨씬 더 효율적으로 좋은 모델을 찾을 수 있겠다는 확신이 들었다. 다음에는 모델의 **하이퍼파라미터**를 `Optuna`로 더 깊게 튜닝해보며 성능을 극한까지 끌어올리는 작업을 해볼 계획이다. 😄
