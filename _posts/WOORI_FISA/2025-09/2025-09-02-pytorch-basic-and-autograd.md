---
layout: post
title: "PyTorch 기초와 자동 미분(Autograd) 뿌수기 👊"
date: 2025-09-02 10:00:00 +0900
categories: [WOORI_FISA, Deep_Learning]
tags: [PyTorch, DeepLearning, Autograd, Tensor]
---

<br>

## 👊 PyTorch 기초 뿌수기

오늘은 파이토치(PyTorch)의 가장 기본적인 개념부터 핵심 기능인 자동 미분까지 학습한 내용을 정리해보려고 한다. 딥러닝의 세계에 첫발을 내딛는 느낌이라 설레면서도, 한편으로는 복잡한 개념들에 정신이 아찔했다. 😵‍💫

그래도 역시 직접 코드를 만져보니 이해가 쏙쏙 되는 기분! ✨

### 🤔 딥러닝? 인공 신경망? 

- **딥러닝(Deep Learning)** 은 머신러닝의 한 분야로, **인공 신경망(ANN)** 을 사용해 데이터에서 복잡한 패턴을 학습하는 기술이다. 여러 층(Layer)을 깊게 쌓아서 데이터의 추상적인 특징을 학습하는 것이 핵심이다.
- **인공 신경망(ANN)** 은 인간의 뇌를 모방한 모델로, 예측과 실제 값의 오차를 줄여나가는 방식으로 학습한다. 이 과정에서 **경사 하강법(Gradient Descent)** 이 핵심적인 역할을 했다.

### 🔥 PyTorch와 텐서(Tensor)

**PyTorch**는 Facebook에서 만든 딥러닝 프레임워크다. NumPy와 비슷하지만 GPU를 활용한 빠른 연산이 가능하고, **동적 계산 그래프**를 지원해서 모델을 유연하게 만들 수 있다는 점이 큰 장점이다.

그리고 PyTorch의 모든 것은 **텐서(Tensor)** 로 통한다. 텐서는 다차원 배열로, 모델에 들어가는 데이터, 가중치, 출력 등 모든 것이 텐서로 표현된다.

- **주요 데이터 타입**:
    - `torch.float32`: 신경망 연산의 기본.
    - `torch.int64`: 인덱싱이나 레이블에 사용.
    - `torch.bool`: 마스킹 작업에 유용.

### 🤹 텐서, 가지고 놀아보기! 

텐서는 `torch.tensor()`, `torch.rand()` 등 다양한 방법으로 만들 수 있었다.

```python
import torch
import numpy as np

# 리스트나 NumPy 배열로 만들기
tensor_from_list = torch.tensor([1, 2, 3])
tensor_from_array = torch.from_numpy(np.array([4, 5, 6])) # from_numpy가 더 효율적!

# 특정 모양의 텐서 만들기
zeros_tensor = torch.zeros((2, 3)) # 2x3 크기를 0으로 채우기
rand_tensor = torch.rand((2, 3))   # 0과 1 사이의 랜덤 값으로 채우기
```

형태를 바꾸거나(`reshape`, `squeeze`, `unsqueeze`), 텐서를 나누고 합치는(`chunk`, `split`, `cat`, `stack`) 등 자유자재로 조작하는 재미가 쏠쏠했다. 특히 행렬 곱(`@` 또는 `matmul()`)과 요소별 곱(`*`)의 차이를 명확히 이해하는 것이 중요했다.

### ✨ 마법 같은 자동 미분, `Autograd` 

PyTorch의 가장 강력한 기능 중 하나는 바로 `Autograd`다. `requires_grad=True`로 설정된 텐서에 대한 모든 연산을 추적해서, `.backward()` 한 방이면 알아서 기울기(gradient)를 계산해준다.

경사 하강법을 구현할 때 미분 계산 때문에 골치 아플 일이 없어진 것이다. 이거 완전 마법 아닌가? 🪄

```python
# w에 대한 l의 기울기를 계산해보자!
w = torch.tensor(4.0, requires_grad=True)
x = torch.tensor(3.0, requires_grad=True)

l = (3 * w)**2 + (2 * x)**2 # l = 9w^2 + 4x^2

# 역전파 실행!
l.backward()

# 기울기 확인
print(f"w의 기울기: {w.grad}") # dl/dw = 18w = 72.0
print(f"x의 기울기: {x.grad}") # dl/dx = 8x = 24.0
```
결과가 정확하게 나오는 것을 보고 감탄했다.

### 실습: 경사 하강법으로 깨진 이미지 복원하기

오늘 배운 개념을 총동원해서 재미있는 실습을 진행했다. 특정 함수에 의해 망가진 이미지를 경사 하강법을 이용해 원본으로 복원하는 것이었다.

**전략은 이랬다:**
1.  복원할 이미지와 똑같은 크기의 **무작위 텐서**를 만든다. (이 녀석이 우리의 '후보'!)
2.  이 후보 텐서를 망가뜨리는 함수에 넣어 **가설 이미지**를 생성한다.
3.  **손실(loss)** 을 '가설 이미지'와 '실제 망가진 이미지'의 차이로 정의한다.
4.  손실을 최소화하는 방향으로 후보 텐서를 계속 업데이트한다!

```python
# 복원할 대상인 랜덤 텐서 생성
random_tensor = torch.randn(10000, dtype=torch.float64)
lr = 0.8 # 학습률

for i in range(20000):
    random_tensor.requires_grad_(True)

    # 1. 가설 생성
    hypothesis = weird_function(random_tensor)
    
    # 2. 손실 계산
    loss = torch.dist(hypothesis, broken_image)

    # 3. 기울기 계산
    loss.backward()

    # 4. 텐서 업데이트 (이때는 기울기 추적 X)
    with torch.no_grad():
        random_tensor = random_tensor - lr * random_tensor.grad

    if i % 1000 == 0:
        print(f'{i}번째 학습 Loss: {loss.item()}')
```

학습을 반복할수록 `loss`가 줄어들고, 무작위 텐서가 점점 원본 이미지의 모습을 찾아가는 과정이 정말 신기했다. 최종적으로 복원된 이미지를 봤을 때의 짜릿함이란! 🤩

### ✨ 오늘의 회고

- **느낀점**: 딥러닝의 핵심 원리를 코드로 직접 구현해보니 이론으로만 배우던 것보다 훨씬 깊게 와닿았다. 특히 `Autograd`의 편리함에 크게 감동했다. 앞으로 모델을 만들 때 이 기능을 마음껏 활용할 수 있겠다는 자신감이 생겼다.
- **궁금한 점**: `torch.from_numpy()`와 `torch.tensor()`의 정확한 차이, 그리고 메모리 공유의 장단점이 궁금해졌다. 더 깊이 파봐야겠다.
- **더 학습할 내용**: 오늘은 기초를 다졌으니, 다음에는 `nn.Module`을 상속받아 직접 신경망 모델을 설계하고 학습시키는 과정을 제대로 해보고 싶다. 가자! 🔥
