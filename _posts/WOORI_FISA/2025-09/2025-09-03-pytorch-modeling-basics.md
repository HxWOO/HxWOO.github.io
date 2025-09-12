---
layout: post
title: "PyTorch로 구현하는 핵심 머신러닝 모델 🤖"
date: 2025-09-03 10:00:00 +0900
categories: [WOORI_FISA, Deep_Learning]
tags: [PyTorch, Modeling, LinearRegression, Classification, MNIST]
---

어제 PyTorch의 기초와 Autograd를 익혔으니, 오늘은 직접 모델을 만들어볼 차례다! 선형 회귀부터 시작해서 이진 분류, 그리고 손글씨를 알아보는 다중 분류 모델까지, PyTorch로 주요 머신러닝 모델들을 구현해본 과정을 기록한다.

직접 모델을 설계하고 학습시키는 과정은 언제나 두근거린다. 가보자고! 🚀

---

## 1. 선형 회귀 모델

선형 회귀는 연속적인 값을 예측하는 모델이다. PyTorch의 `nn.Sequential`을 사용해 간단한 다층 퍼셉트론(MLP) 모델을 구성하고, 평균 제곱 오차(MSE)를 손실 함수로 사용해 보스턴 집값 예측을 학습시켰다.

### 핵심 개념
- **모델 구조**: `nn.Linear` 레이어를 쌓아 MLP를 구성했다.
- **활성화 함수**: `nn.ReLU`를 사용해 비선형성을 추가했다.
- **손실 함수**: `nn.MSELoss`를 사용해 예측값과 실제값의 차이를 계산했다.
- **최적화**: `torch.optim.Adam`으로 가중치를 업데이트했다.

### 코드 구현

```python
import torch
import torch.nn as nn
from torch.optim.adam import Adam
from torch.utils.data import Dataset, DataLoader
import pandas as pd

# ... (데이터셋 클래스 및 데이터 준비 코드는 생략) ...

# 모델 정의
model = nn.Sequential(
    nn.Linear(13, 30),
    nn.ReLU(),
    nn.Linear(30, 15),
    nn.ReLU(),
    nn.Linear(15, 8),
    nn.ReLU(),
    nn.Linear(8, 1)
)

# 학습 루프
optimizer = Adam(model.parameters(), lr=0.001)
criterion = nn.MSELoss()

for epoch in range(2000):
    for data, label in dataloader:
        prediction = model(data)
        loss = criterion(prediction, label)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
    if epoch % 100 == 0:
        print(f'Epoch {epoch}: Loss {loss.item():.4f}')
```
학습이 진행될수록 손실이 쭉쭉 줄어드는 것을 보니 뿌듯했다. 🎉

---

## 2. 이진 분류 모델

다음은 두 개의 클래스 중 하나를 예측하는 이진 분류 모델이다. 출력 레이어에 `nn.Sigmoid` 활성화 함수를 붙여 결과를 0과 1 사이의 확률 값으로 만들고, 이진 교차 엔트로피(BCE) 손실 함수를 사용했다.

### 핵심 개념
- **출력 활성화 함수**: `nn.Sigmoid`를 사용해 출력을 확률로 변환했다.
- **손실 함수**: `nn.BCELoss` (Binary Cross-Entropy Loss)를 사용했다.

### 코드 구현

```python
# 모델 정의 (출력층 변경)
binary_model = nn.Sequential(
    nn.Linear(13, 32),
    nn.ReLU(),
    nn.Linear(32, 16),
    nn.ReLU(),
    nn.Linear(16, 1),
    nn.Sigmoid() # 출력을 0과 1 사이로!
)

# ... (데이터 준비 코드 생략) ...

# 학습 루프
optimizer_binary = Adam(binary_model.parameters(), lr=0.001)
criterion_binary = nn.BCELoss() # 이진 분류 손실 함수

# ... (학습 루프 코드 생략) ...
```
회귀 모델의 구조를 약간만 변경해서 분류 모델을 만들 수 있다는 점이 인상 깊었다. 👍

---

## 3. 다중 분류 모델 (Softmax)

세 개 이상의 클래스를 분류할 때는 Softmax 함수가 등장한다. 모델의 최종 출력(logit)을 모든 클래스에 대한 확률 분포로 바꿔주는 역할을 한다.

### 핵심 개념
- **Softmax 함수**: 모든 출력의 합이 1이 되도록 만들어, 각 클래스에 속할 확률을 나타낸다.
- **손실 함수**: `nn.CrossEntropyLoss`를 사용한다. **주의할 점!** 이 손실 함수는 내부적으로 `Softmax`를 포함하고 있어서, 모델의 마지막에 `Softmax` 레이어를 추가하면 안 된다.

---

## 4. MNIST 손글씨 분류

다중 분류의 "Hello, World!"인 MNIST 손글씨 분류를 직접 구현해봤다. 28x28 픽셀의 이미지를 784개의 1차원 벡터로 펼쳐서 모델에 입력하고, 0부터 9까지 10개의 클래스 중 하나로 분류하는 문제다.

### 핵심 개념
- **데이터셋**: `torchvision.datasets.MNIST`로 데이터를 정말 쉽게 불러왔다.
- **데이터 전처리**: `transforms.ToTensor()`로 이미지를 텐서로 바꾸고 정규화까지 한 번에 해결했다.
- **모델 구조**: 입력은 784, 출력은 10개의 노드를 가지도록 설계했다.

### 코드 구현

```python
from torchvision.datasets.mnist import MNIST
from torchvision.transforms import ToTensor

# ... (데이터 로드 코드 생략) ...

# 모델 정의
class MNIST_Classifier(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(28 * 28, 128)
        self.fc2 = nn.Linear(128, 64)
        self.output = nn.Linear(64, 10)

    def forward(self, x):
        x = x.view(x.size(0), -1) # 2D 이미지를 1D 벡터로 펼침
        x = torch.relu(self.fc1(x))
        x = torch.relu(self.fc2(x))
        x = self.output(x) # CrossEntropyLoss가 Softmax를 포함하므로 그대로 출력
        return x

# ... (학습 및 평가 코드 생략) ...

# print(f'Test Accuracy: {100 * correct / total:.2f}%')
```
결과는... 테스트 정확도 약 95% 달성! 직접 만든 모델이 손글씨를 알아맞히는 걸 보니 정말 신기했다. 🚀

---

## 느낀점

- **느낀점**: `nn.Sequential`과 `nn.Module`을 사용하니 모델 구조를 레고처럼 쉽게 조립할 수 있었다. 각 문제(회귀, 분류)에 맞는 손실 함수(`MSELoss`, `BCELoss`, `CrossEntropyLoss`)와 출력층 활성화 함수(`Sigmoid`)를 선택하는 것이 중요하다는 점을 다시 한번 깨달았다. 특히 `CrossEntropyLoss`가 내부적으로 Softmax를 포함하고 있다는 점은 헷갈리기 쉬우니 꼭 기억해야겠다.
- **어려웠던 점**: 데이터의 형태(shape)를 모델의 각 레이어에 맞게 조정하는 부분이 여전히 조금 헷갈린다. `view`나 `reshape`을 사용할 때 실수가 잦아서 디버깅에 시간이 걸렸다. 😭
- **더 학습할 내용**: 오늘은 기본적인 MLP 모델만 만들었지만, 다음에는 CNN(Convolutional Neural Networks)이나 RNN(Recurrent Neural Networks) 같은 더 복잡한 모델 구조에 도전해보고 싶다. 이미지나 시계열 데이터를 다루는 날이 기다려진다! 🔥
