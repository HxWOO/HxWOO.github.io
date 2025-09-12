---
layout: post
title:  "🚗💨 자동차 연비 예측 DNN 회귀 모델링 실습"
date:   2025-09-04 10:00:00 +0900
categories: [WOORI_FISA, Deep_Learning]
tags: [PyTorch, DNN, Regression, Feature Engineering, Normalization, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🚗 오늘의 목표: 자동차 연비 예측 모델 만들기

오늘은 1970-80년대의 클래식한 자동차 데이터를 가지고 연비(MPG)를 예측하는 DNN 회귀 모델을 PyTorch로 만들어보았다. 데이터 전처리부터 특성 공학, 모델링, 그리고 평가까지 전체 파이프라인을 경험해보는 알찬 실습이었다. 😄

> ### ✨ 핵심 정리
> **#데이터전처리 #특성공학 #정규화 #PyTorch #DNN #회귀모델 #규제**

---

### 🛠️ 한눈에 보는 과정

| 단계 | 내용 | 핵심 기술/개념 |
| --- | --- | --- |
| **1. 데이터 준비** | UCI 서버에서 데이터를 불러와 `pandas`로 확인했다. | `pandas.read_csv` |
| **2. 전처리** | `Horsepower` 열의 결측치를 깔끔하게 제거했다. | `dropna()` |
| **3. 특성 공학** | `Model Year`는 그룹으로 묶고, `Origin`은 원-핫 인코딩을 시도했다. | `torch.bucketize`, `one_hot` |
| **4. 정규화** | `StandardScaler`로 수치 데이터들의 스케일을 맞췄다. 안 하면 학습이 불안정해진다. | `StandardScaler` |
| **5. 모델링 및 학습** | PyTorch로 간단한 DNN 회귀 모델을 만들고 열심히 학습시켰다. 🤓 | `torch.nn.Sequential`, `MSELoss`, `SGD` |
| **6. 성능 평가 및 규제** | 학습된 모델의 성능을 MSE와 MAE로 평가하고, 과적합을 막기 위한 L1/L2 규제도 살펴보았다. | `L1Loss`, `weight_decay` |

---

### 📜 삽질과 배움의 기록

#### 1. 데이터 준비 및 전처리

- **데이터 로드**: `pd.read_csv`로 데이터를 불러왔다. 공백으로 구분된 파일이라 `sep=" "` 옵션을 줬다.
- **결측치 처리**: `Horsepower` 열에 `?`가 섞여있었다. `na_values='?'`로 `NaN` 변환 후 `dropna()`로 날려버렸다. 속 시원! 👍
- **데이터 분할**: `train_test_split`으로 훈련용과 테스트용 데이터를 나눴다. 기본 중의 기본이다.

```python
# 데이터 로드 및 전처리
import pandas as pd
import torch
from sklearn.model_selection import train_test_split

url = 'http://archive.ics.uci.edu/ml/machine-learning-databases/auto-mpg/auto-mpg.data'
column_names = ['MPG', 'Cylinders', 'Displacement', 'Horsepower', 'Weight',
                'Acceleration', 'Model Year', 'Origin']
df = pd.read_csv(url, names=column_names, na_values="?", comment='\t', sep=" ", skipinitialspace=True)
df = df.dropna()

# 데이터 분할
X_train_df, X_test_df, y_train_df, y_test_df = train_test_split(df.iloc[:, 1:], df['MPG'], test_size=0.2, random_state=121)
```

#### 2. 데이터 정규화 (Normalization)

> **정규화는 선택이 아닌 필수!** ✨
> 특성마다 값의 범위가 너무 달라서(`Weight` vs `Cylinders`) 정규화는 필수였다. 안 그러면 모델이 큰 값에만 휘둘릴 수 있다. `StandardScaler`를 써서 간단히 해결했다.

```python
from sklearn.preprocessing import StandardScaler

numeric_columns = ['Cylinders', 'Displacement', 'Horsepower', 'Weight', 'Acceleration']
scaler = StandardScaler()

# 훈련 데이터 기준으로 스케일러 학습 및 적용
X_train_df[numeric_columns] = scaler.fit_transform(X_train_df[numeric_columns])
# 테스트 데이터는 학습된 스케일러로 변환만!
X_test_df[numeric_columns] = scaler.transform(X_test_df[numeric_columns])
```

#### 3. 특성 공학 (Feature Engineering)

- **`Model Year`**: 연도를 몇 개 그룹으로 묶었다 (`torch.bucketize`). 너무 많은 연도를 그대로 쓰기보다 그룹으로 만드는 게 낫다고 판단했다.
- **`Origin`**: 제조 국가를 원-핫 인코딩(`one_hot`)했다. 모델이 국가를 서열로 오해하면 안 되니까!

```python
from torch.nn.functional import one_hot

# Model Year 버킷화
boundaries = torch.tensor([73, 76, 79])
X_train_df['Model Year Bucketed'] = torch.bucketize(torch.tensor(X_train_df['Model Year'].values), boundaries, right=True)
X_test_df['Model Year Bucketed'] = torch.bucketize(torch.tensor(X_test_df['Model Year'].values), boundaries, right=True)

# Origin 원-핫 인코딩 및 텐서 결합
train_origin_encoded = one_hot(torch.tensor(X_train_df['Origin'].values) % 3)
test_origin_encoded = one_hot(torch.tensor(X_test_df['Origin'].values) % 3)

x_train = torch.cat([torch.tensor(X_train_df[numeric_columns + ['Model Year Bucketed']].values), train_origin_encoded], 1).float()
x_test = torch.cat([torch.tensor(X_test_df[numeric_columns + ['Model Year Bucketed']].values), test_origin_encoded], 1).float()

y_train = torch.FloatTensor(y_train_df.values)
y_test = torch.FloatTensor(y_test_df.values)
```

#### 4. DNN 모델 구축 및 학습

- **모델 설계**: `torch.nn.Sequential`로 간단한 DNN 모델을 만들었다. 은닉층 2개에 활성화 함수는 ReLU!
- **손실 함수와 옵티마이저**: 회귀 문제라 손실 함수는 `MSELoss`, 옵티마이저는 클래식한 `SGD`를 썼다.
- **학습 시작!**: `DataLoader`로 데이터를 조금씩 모델에 주면서 200 에포크 동안 학습시켰다. 손실이 쭉쭉 떨어지는 걸 보니 뿌듯했다. 🎉

```python
from torch.utils.data import DataLoader, TensorDataset
import torch.nn as nn

# 데이터 로더 생성
train_ds = TensorDataset(x_train, y_train)
train_dl = DataLoader(train_ds, batch_size=8, shuffle=True)

# 모델 정의
hidden_units = [8, 4]
input_size = x_train.shape[1]
all_layers = []
for hidden_unit in hidden_units:
    layer = nn.Linear(input_size, hidden_unit)
    all_layers.append(layer)
    all_layers.append(nn.ReLU())
    input_size = hidden_unit
all_layers.append(nn.Linear(hidden_units[-1], 1))
model = nn.Sequential(*all_layers)

# 손실 함수 및 옵티마이저
loss_fn = nn.MSELoss()
optimizer = torch.optim.SGD(model.parameters(), lr=0.001)

# 모델 학습
torch.manual_seed(1)
num_epochs = 200
for epoch in range(num_epochs):
    loss_train = 0
    for X, y in train_dl:
        pred = model(X).squeeze()
        loss = loss_fn(pred, y)
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()
        loss_train += loss.item()
    if epoch % 20 == 0:
        print(f'Epoch {epoch}, Loss {loss_train/len(train_dl):.4f}')
```

#### 5. 모델 성능 평가 및 규제

- **성능 평가**: 학습이 끝난 모델을 테스트 데이터로 평가했다. MSE와 MAE 값을 확인하니, 나쁘지 않은 결과가 나왔다.
- **과적합 방지**: 모델이 훈련 데이터에만 너무 익숙해지는 걸 막기 위해 **규제(Regularization)**라는 기법이 있다.
    - **L1 (Lasso)**: 불필요한 특성의 가중치를 0으로 만들어 버린다.
    - **L2 (Ridge)**: 가중치들이 너무 커지지 않게 전반적으로 누르는 느낌. `SGD` 옵티마이저의 `weight_decay`로 쉽게 적용할 수 있다.

```python
# 모델 평가
model.eval()
with torch.no_grad():
    pred = model(x_test).squeeze()
    test_mse = loss_fn(pred, y_test)
    test_mae = nn.L1Loss()(pred, y_test)
    print(f'Test MSE: {test_mse.item():.4f}') # 결과 확인!
    print(f'Test MAE: {test_mae.item():.4f}')

# L2 규제 적용 예시
optimizer_l2 = torch.optim.SGD(model.parameters(), lr=0.001, weight_decay=0.5)
```

---

### ✨ 오늘의 회고

데이터 전처리부터 모델링, 평가까지 전체 파이프라인을 경험해본 좋은 실습이었다. 특히 **데이터 정규화**의 중요성을 다시 한번 느꼈다. PyTorch로 직접 모델을 짜보니 딥러닝 모델의 구조가 더 명확하게 이해됐고, 훈련 손실과 테스트 손실을 비교하며 모델의 일반화 성능을 가늠해볼 수 있었고, **규제**의 필요성도 체감했다.

다음에는 하이퍼파라미터 튜닝으로 모델 성능을 더 끌어올려 봐야겠다. 
