---
layout: post
title:  "🕸️ PyTorch로 밑바닥부터 CNN 쌓아보기 (Fashion MNIST 실습)"
date:   2025-09-05 10:00:00 +0900
categories: [WOORI_FISA, Deep_Learning]
tags: [cnn, pytorch, deeplearning, fashionmnist, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링']
---

<br>

## 🚀 PyTorch로 CNN 모델 단계별로 정복하기

오늘은 마치 레고 블록을 하나씩 쌓아 멋진 성을 만들듯, 간단한 신경망에서 시작해 완전한 CNN 모델을 단계별로 구축하는 과정을 진행했다. Fashion MNIST 데이터셋을 가지고, 각 부품(레이어)이 모델 성능에 어떤 영향을 미치는지 직접 확인해보는 시간이었다.

> #### 🧐 "CNN, 왜 이미지 처리에 강력할까?"
> 이미지는 픽셀의 위치와 관계, 즉 **공간적 특징**이 매우 중요하다. CNN은 **컨볼루션(Convolution)** 과 **풀링(Pooling)** 이라는 특별한 연산으로 이 공간 정보를 똑똑하게 유지하며 학습하기 때문에 이미지 인식에서 뛰어난 성능을 발휘한다.

---

### 🛠️ CNN의 핵심 재료: 컨볼루션과 풀링

- **컨볼루션(Convolution)**: 이미지 위를 슬라이딩하는 **필터(커널)** 를 통해 윤곽선, 질감 같은 고유한 특징을 추출한다. 이 결과물을 **특징 맵(Feature Map)** 이라고 부른다.
- **풀링(Pooling)**: 특징 맵의 크기를 줄여(Sub-sampling) 중요한 정보만 남긴다. **맥스 풀링(Max Pooling)** 은 특정 영역에서 가장 중요한 특징(가장 큰 값)만 뽑아내 모델을 더 강인하게 만든다.

<img src="https://miro.medium.com/v2/resize:fit:1400/0*qREkleedQfWNO6ee.gif" width="500">

---

### ✨ 모델 빌드업 과정: 바닥부터 천천히

Fashion MNIST 데이터셋으로 모델의 성능이 어떻게 진화하는지 단계별로 따라가 보았다.

#### **1️⃣ Step 1: 기본 DNN 모델 (Baseline)**
- 가장 먼저, 이미지를 1차원으로 길게 펼쳐서 일반적인 완전 연결 신경망(DNN)으로 학습시켰다.
- **결과**: Test Accuracy `88.66%`. 나쁘지 않은 시작이다!

#### **2️⃣ Step 2: 컨볼루션(CNN) 레이어 추가**
- DNN 모델 앞에 컨볼루션 레이어를 추가하여 이미지의 공간 정보를 활용하도록 했다.
- **결과**: Test Accuracy `88.37%`. 정확도는 비슷했지만, 학습 과정이 더 안정화되었다.

#### **3️⃣ Step 3: 맥스풀링(Max Pooling)으로 성능 점프!**
- 컨볼루션 레이어 뒤에 맥스 풀링을 추가해 특징을 압축하고 연산 효율을 높였다.
- **결과**: Test Accuracy `91.22%`. 성능이 눈에 띄게 향상되었다! 역시 CNN의 핵심은 풀링에 있는 것 같다.

#### **4️⃣ Step 4: 과적합 방지 & 안정화 (Dropout, Batch Norm)**
- 모델이 훈련 데이터에만 너무 익숙해지는 과적합을 막기 위해 **드롭아웃(Dropout)** 과 **배치 정규화(Batch Normalization)** 를 추가했다. 이들은 모델을 더 안정적이고 일반화 성능이 좋게 만들어준다.

```python
# 최종 모델의 일부
class CNN(nn.Module):
  def __init__(self):
    super(CNN, self).__init__()
    self.layer1 = nn.Sequential(
        nn.Conv2d(1, 32, kernel_size=3, padding=1),
        nn.BatchNorm2d(32), # 배치 정규화
        nn.ReLU(),
        nn.MaxPool2d(2),    # 맥스 풀링
        nn.Dropout(0.3)     # 드롭아웃
    )
    self.layer2 = nn.Sequential(
        nn.Conv2d(32, 64, kernel_size=3, padding=1),
        nn.BatchNorm2d(64),
        nn.ReLU(),
        nn.MaxPool2d(2),
        nn.Dropout(0.3)
    )
    self.fc1 = nn.Linear(64 * 7 * 7, 10) # 완전 연결 레이어

  def forward(self, x):
    x = self.layer1(x)
    x = self.layer2(x)
    x = x.view(x.size(0), -1) # flatten
    x = self.fc1(x)
    return x
```

- **최종 결과**: Test Accuracy **`~93%`**. 모든 부품을 조립하니 훨씬 견고하고 강력한 모델이 완성되었다. 👍

---

### ✨ 오늘의 회고

단순한 DNN 모델에서 시작해 컨볼루션, 풀링, 드롭아웃, 배치 정규화 같은 개념들을 하나씩 추가하며 모델의 성능이 눈에 띄게 좋아지는 것을 직접 확인하니 정말 흥미로웠다. 각 레이어가 어떤 역할을 하는지, 왜 필요한지를 몸소 체감할 수 있는 최고의 실습이었다.

특히 맥스 풀링을 추가했을 때 성능이 크게 오르는 것을 보고, 특징을 잘 압축하고 강조하는 것이 얼마나 중요한지 깨달았다. 다음에는 오늘 만든 모델을 기반으로 더 복잡한 데이터셋에 도전하거나, VGG나 ResNet 같은 유명한 CNN 아키텍처를 분석해봐야겠다. 😄
