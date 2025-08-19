---
layout: post
title:  "🤖 머신러닝 기초, 통계부터 딥러닝까지 코드로 정복하기!"
date:   2025-08-19 10:00:00 +0900
categories: [AI Engineering, Woori FISA]
tags: ['Python', 'Pandas', 'Statistics', 'ML', 'DeepLearning', '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🚀 오늘의 항해: 머신러닝의 바다로 떠나다

오늘은 인공지능 엔지니어의 길을 걷기 위해 가장 먼저 만나야 할 두 거인, **통계**와 **머신러닝**을 만났다. 단순히 눈으로만 읽는 공부가 아니라, 파이썬 코드로 직접 데이터를 주무르고, 통계의 원리를 파헤치며 딥러닝의 가장 작은 단위인 퍼셉트론까지 직접 만들어보는 경험을 했다. 😄

> ### :guard: "데이터를 이해하지 못하면, 모델은 그저 허상일 뿐이다."
> 모델을 만드는 것은 어쩌면 쉬울지도 모른다. 하지만 그 모델이 왜 그렇게 작동하는지, 어떤 데이터를 먹고 자랐는지 이해하지 못한다면, 작은 변화에도 쉽게 무너지는 모래성을 쌓는 것과 같다는 생각이 들었다.

---

### 🐍 기본 라이브러리 준비

모든 코드는 **환경설정**부터 시작한다! 데이터 분석과 머신러닝에 필요한 기본 라이브러리를 가져오는 것으로 학습을 시작했다.

```python
import warnings # 경고 메시지 무시
warnings.filterwarnings('ignore')

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# numpy 출력 옵션 변경 (지수표기법 방지)
np.set_printoptions(suppress=True)
```

---

### 📊 기술통계: 데이터의 민낯 들여다보기

수집한 데이터를 요약, 묘사, 설명하는 기술통계는 데이터의 전반적인 특징을 파악하는 데 사용된다.

#### 1. 중심에 대한 통계 (feat. 도미 데이터 :fish:)

데이터의 중심 경향을 나타내는 값들입니다. 실제 도미 데이터를 `pandas`의 DataFrame으로 만들어 각종 통계량을 직접 구해보았다.

**- 데이터 준비**
```python
# 도미 데이터 (길이, 무게)
bream_length = [25.4, 26.3, 26.5, 29.0, 29.0, 29.7, 29.7, 30.0, 30.0, 30.7, 31.0, 31.0, 31.5, 32.0, 32.0, 32.0, 33.0, 33.0, 33.5, 33.5, 34.0, 34.0, 34.5, 35.0, 35.0, 35.0, 35.0, 36.0, 36.0, 37.0, 38.5, 38.5, 39.5, 41.0, 41.0]
bream_weight = [242.0, 290.0, 340.0, 363.0, 430.0, 450.0, 500.0, 390.0, 450.0, 500.0, 475.0, 500.0, 500.0, 340.0, 600.0, 600.0, 700.0, 700.0, 610.0, 650.0, 575.0, 685.0, 620.0, 680.0, 700.0, 725.0, 720.0, 714.0, 850.0, 1000.0, 920.0, 955.0, 925.0, 975.0, 950.0]

df = pd.DataFrame(zip(bream_length, bream_weight), columns=['length', 'weight'])
```

**- 평균, 중앙값, 최빈값 계산**

`describe()` 함수로 기본적인 통계량을 한 번에 확인하고, `value_counts()`로 최빈값을 구했다.

```python
# 기본 통계량 (평균-mean, 중앙값-50%)
df.describe()

# 최빈값 (가장 많이 등장하는 값)
df.length.value_counts().head(1)
```

| 구분 | 설명 |
| :--- | :--- |
| **평균 (Mean)** | 모든 값을 더해 개수로 나눈 값. `describe()`의 `mean`에 해당 |
| **중앙값 (Median)** | 데이터를 정렬했을 때 중앙에 위치하는 값. `describe()`의 `50%`에 해당 |
| **최빈값 (Mode)** | 가장 자주 나타나는 값. `value_counts()`로 확인 가능 |

#### 2. 산포에 대한 통계

데이터가 중심으로부터 얼마나 흩어져 있는지를 나타낸다.

**- 분산과 표준편차 계산**
```python
# 1. 편차 (Deviation)
mean_ = df.length.mean()
deviation = df.length - mean_

# 2. 변동 (Variation) - 편차의 제곱합
variation = deviation**2

# 3. 분산 (Variance)
variance = sum(variation) / len(variation)

# 4. 표준편차 (Standard Deviation)
std = np.sqrt(variance)
```

#### 3. 정규화와 표준화

데이터의 단위를 맞춰주어(스케일링) 모델이 더 잘 학습할 수 있도록 돕는다.

**- 표준화 (Standardization, Z-score)**

데이터에서 평균을 빼고 표준편차로 나누어, 평균 0, 표준편차 1인 분포로 변환

```python
# 도미 길이(length)를 표준화
z_length = (df.length - df.length.mean()) / np.std(df.length)

# 도미 무게(weight)를 표준화
z_weight = (df.weight - df.weight.mean()) / np.std(df.weight)

# 표준화된 데이터 확인
new_df = pd.DataFrame(zip(z_length, z_weight), columns=['length_z', 'weight_z'])
new_df.describe()
```
> 결과를 보면 `mean`은 0에 가깝고, `std`는 1에 가까운 값으로 변환된 것을 볼 수 있다. 🎉

---

### :brain: 딥러닝: 퍼셉트론으로 맛보기

딥러닝의 기본 단위인 뉴런(퍼셉트론)이 어떻게 학습하는지 간단한 코드로 구현해보았다.

**- 퍼셉트론 학습 과정**

`x=1`이 입력되면 `y=0`을 출력하는 아주 간단한 뉴런

```python
# y = wx + b
w = np.random.uniform(0, 1) # 가중치(w)는 랜덤 값으로 시작
x = 1
y = 0
eta = 0.1  # 학습률 (learning rate)

for i in range(10): # 10번 반복 학습
    output = w * x      # 예측값 계산
    error = y - output  # 실제값과 예측값의 차이(오차) 계산
    
    # 경사 하강법: 오차의 반대 방향으로 가중치를 업데이트
    w = w + (x * error) * eta
    
    print(f"{i+1}회차 가중치: {w:.4f}, 오차: {error:.4f}")

print(f"\n최종 예측값: {w*x:.4f}")
```
학습이 반복될수록 가중치(w)가 점차 갱신되면서 오차(error)가 0에 가까워지고, 최종 예측값이 목표인 0에 수렴하는 것을 확인할 수 있다. :bulb:

---

### ✨ 오늘의 회고

오늘 학습을 통해 데이터와 모델의 관계를 어렴풋이나마 이해할 수 있었다. 특히 간단한 퍼셉트론 코드를 통해, 복잡하게만 느껴졌던 '학습'이라는 과정이 결국 '오차를 줄여나가는 여정'이라는 것을 체감했다. 
앞으로 배울 더 복잡한 모델들도 결국은 이 기본 원리 위에 서 있다는 생각을 하니, 왠지 모를 자신감이 생긴다. 다음에는 더 다양한 머신러닝 모델들을 만나고, 직접 구현해보는 시간을 가질 예정이다.🔥