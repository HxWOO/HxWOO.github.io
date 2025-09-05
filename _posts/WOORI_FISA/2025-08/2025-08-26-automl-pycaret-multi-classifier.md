---
layout: post
title:  "🤖 AutoML, 코딩의 종말? PyCaret으로 모델링 완전 정복"
date:   2025-08-26 10:00:00 +0900
categories: [Woori FISA, Machine Learning]
tags: [AutoML, PyCaret, 로우코드, MLOps, 데이터분석, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## :carrot: "이걸 다 언제 코딩하지?"에 대한 해답, PyCaret

이전 실습에서 일일히 머신러닝 코드를 써가며 분류 모델을 만들었던 기억, 아직도 생생하다. 그런데 이번에는 AutoML(자동화 머신러닝) 라이브러리인 `PyCaret`을 사용해봤다. 단 몇 줄의 코드로 전체 머신러닝 워크플로우를 자동화하는 경험은 정말 충격 그 자체였다. 🤯

> ### 💡 로우코드(Low-code) 라이브러리의 힘!
> PyCaret은 Scikit-learn, XGBoost 같은 여러 라이브러리를 쉽게 쓸 수 있게 감싸서, 데이터 전처리부터 모델 학습, 튜닝, 배포까지 모든 과정을 자동화해준다. 덕분에 나는 모델링 자체에 더 집중할 수 있었다.

---

### 🚀 PyCaret 핵심 워크플로우: 딱 5단계만 기억해!

PyCaret의 작업은 놀라울 정도로 간단한 5단계로 요약된다.

**Setup ➡️ Compare Models ➡️ Analyze Model ➡️ Prediction ➡️ Save Model**

이 흐름만 따라가면 누구나 전문가 수준의 모델을 만들 수 있다는 게 정말 매력적이었다.

#### 1️⃣ 실험 준비: `setup()`
모든 작업의 시작. `setup` 함수 하나로 모든 게 준비된다. 데이터 분석, 결측치 처리, 인코딩, 데이터 분할까지 알아서 다 해주니 시간을 엄청나게 아낄 수 있었다.

```python
from pycaret.classification import *
s = setup(data, target = 'species', session_id = 123)
```

#### 2️⃣ 모델 비교: `compare_models()`
PyCaret의 꽃! 🌸 단 한 줄의 코드로 수십 개 모델을 한 번에 학습하고, 교차 검증 성능을 표로 쫙 보여준다. 어떤 모델이 내 데이터에 가장 적합한지 순식간에 파악할 수 있었다.

```python
best = compare_models()
```

#### 3️⃣ 모델 생성 및 튜닝: `create_model()` & `tune_model()`
가장 좋은 모델이나 특정 모델을 골라 더 깊게 파고들 수 있다. `tune_model()`은 RandomizedSearchCV를 기반으로 최적의 하이퍼파라미터를 찾아 모델 성능을 극대화해준다.

```python
dt = create_model('dt')
tuned_dt = tune_model(dt)
```

#### 4️⃣ 모델 분석 및 예측: `plot_model()` & `predict_model()`
모델을 만들고 끝이 아니다. 혼동 행렬, 피처 중요도 등 15가지가 넘는 플롯을 그려 모델을 다각도로 분석할 수 있다. SHAP을 이용한 모델 해석까지 가능하다니, 정말 강력했다.

```python
plot_model(best, plot = 'confusion_matrix')
interpret_model(lightgbm, plot = 'summary')
```

#### 5️⃣ 모델 저장 및 배포: `save_model()` & `create_api()`
잘 만든 모델은 전처리 과정까지 포함된 전체 파이프라인으로 저장하고, FastAPI 코드와 Dockerfile까지 자동으로 생성해준다. MLOps의 시작을 이렇게 쉽게 할 수 있다니..! 🐳

```python
save_model(best, 'my_best_pipeline')
create_api(tuned_dt, api_name = 'my_iris_api')
create_docker('my_iris_api')
```

---

### ✨ 오늘의 회고

PyCaret을 사용한 경험은 **'혁신'** 그 자체였다. 이전 실습에서 수백 줄에 걸쳐 했던 데이터 전처리, 모델 비교, 평가, 튜닝 과정을 단 몇 줄의 코드로 압축할 수 있었다.

특히 `compare_models()`로 모든 모델의 성능을 한눈에 비교하고 시작하는 접근법은 시간을 극적으로 단축시켜 주었다. 모델 성능 분석부터 API, Dockerfile 생성까지 지원하는 것을 보며, PyCaret이 단순히 빠른 실험뿐만 아니라 **프로덕션 환경까지 고려한 매우 실용적인 도구**라는 것을 느낄 수 있었다.

앞으로 머신러닝 프로젝트를 시작할 때, PyCaret으로 빠르게 프로토타입을 만들고 가장 유망한 모델을 탐색하는 것이 나의 표준 워크플로우가 될 것 같다. 🚀
