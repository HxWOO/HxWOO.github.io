---
layout: post
title:  "📈PyTorch 시계열 예측 정복하기: DNN, CNN, RNN, LSTM"
date:   2025-09-08 10:00:00 +0900
categories: [WOORI_FISA, Deep_Learning]
tags: [PyTorch, Time_Series, DNN, CNN, RNN, LSTM, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 📈 PyTorch로 시계열 예측의 세계에 빠져들다

미래를 예측하는 것은 언제나 흥미로운 일이다. 주식 가격의 등락, 내일의 날씨, 혹은 다음 분기 매출까지. 이런 예측의 중심에 바로 **시계열 데이터**가 있다. 오늘은 PyTorch를 이용해 과거의 데이터로 미래를 엿보는 다양한 딥러닝 모델들을 탐험하며, 시계열 예측의 기초부터 차근차근 정복해나간 과정을 기록했다.

> #### 💡 "과거는 미래의 거울이다."
> 시계열 데이터는 시간에 따라 기록된 데이터로, 그 안에는 트렌드, 계절성, 자기 상관 등 미래를 예측할 수 있는 중요한 단서들이 숨어있다. 이 단서들을 어떻게 찾아내고 활용하는지가 예측 모델의 성패를 좌우한다.

---

### 🕓 시계열 데이터, 무엇을 담고 있을까?

시계열 데이터는 크게 4가지 공통 특징을 가진다. 이들을 이해하는 것이 분석의 첫걸음이다.

- **트렌드 (Trend):** 데이터가 장기적으로 상승하거나 하강하는 방향성.
- **계절성 (Seasonality):** 특정 주기로 반복되는 패턴 (e.g., 월별, 계절별).
- **자기 상관 (Autocorrelation):** 이전 시점의 데이터가 현재 또는 미래 데이터에 영향을 미치는 현상.
- **잡음 (Noise):** 예측 불가능한 임의의 변동.

이런 특징들을 코드로 직접 만들어보며 시계열 데이터와 친해지는 시간을 가졌다.

```python
# 시계열 데이터 생성을 위한 함수 정의
import numpy as np
import matplotlib.pyplot as plt

def trend(time, slope=0):
    return slope * time

def seasonal_pattern(season_time):
    return np.where(season_time < 0.4,
                    np.cos(season_time * 2 * np.pi),
                    1 / np.exp(3 * season_time))

def seasonality(time, period, amplitude=1, phase=0):
    season_time = ((time + phase) % period) / period
    return amplitude * seasonal_pattern(season_time)

def noise(time, noise_level=1, seed=None):
    rnd = np.random.RandomState(seed)
    return rnd.randn(len(time)) * noise_level

# 4년치 시계열 데이터 생성
time = np.arange(4 * 365 + 1, dtype="float32")
baseline = 10
amplitude = 15
noise_level = 6
series = baseline + trend(time, 0.05) + seasonality(time, period=365, amplitude=amplitude)
series += noise(time, noise_level, seed=42)
```

---

### 🛠️ 딥러닝 모델별 예측 도전기

#### 1. 윈도우 데이터셋: 예측을 위한 재료 손질

딥러닝 모델에 시계열 데이터를 넣기 위해선 '윈도우' 단위로 잘라주는 작업이 필요하다. 특정 기간(window)의 데이터를 입력(X)으로, 바로 다음 시점의 데이터를 정답(y)으로 만드는 것이다. PyTorch의 `Dataset`과 `DataLoader`를 활용해 이 과정을 효율적으로 처리했다.

```python
from torch.utils.data import Dataset, DataLoader

class WindowedDataset(Dataset):
    def __init__(self, data, window_size, shift):
        self.data = data
        self.window_size = window_size
        self.shift = shift
        self.windows = [data[i:i+window_size+1] for i in range(0, len(data)-window_size, shift)]

    def __len__(self):
        return len(self.windows)

    def __getitem__(self, idx):
        window = self.windows[idx]
        return window[:-1], window[-1:]
```

#### 2. DNN (Deep Neural Network) 모델

가장 기본적인 `Linear` 레이어를 쌓아 만든 DNN 모델로 첫 예측을 시도했다. 결과는 MSE 42.65, MAE 5.20. 생각보다 성능이 좋지 않아 아쉬웠다. 🤔

#### 3. 1D CNN (Convolutional Neural Network) 모델

`Conv1d` 레이어를 사용해 시계열의 지역적 패턴을 포착하는 CNN 모델을 만들었다. 이미지 처리에만 쓰이는 줄 알았던 CNN을 시계열에 적용하니 신기했다. 하지만 성능은 MSE 44.18, MAE 5.32로 DNN과 비슷했다.

#### 4. RNN (Recurrent Neural Network) 모델

순차 데이터 처리에 강점을 가진 RNN에 큰 기대를 걸었다. 이전 스텝의 출력을 다음 스텝의 입력으로 사용하는 **'되먹임 구조'** 덕분에 시계열의 '흐름'을 학습할 수 있었다. `yfinance`로 실제 AAPL 주가 데이터를 가져와 예측을 진행하니 훨씬 흥미로웠다. 📈

```python
# RNN 모델 정의
class RNN(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, seq_size):
        super(RNN, self).__init__()
        self.rnn = nn.RNN(input_size, hidden_size, num_layers, batch_first=True)
        self.fc1 = nn.Linear(seq_size * hidden_size, 64)
        self.output = nn.Linear(64, 1)
        self.relu = nn.ReLU()

    def forward(self, x, h0=None):
        # ... (forward pass logic)
        return torch.flatten(x)
```

#### 5. LSTM (Long Short-Term Memory) 모델

RNN의 장기 의존성 문제를 해결한 LSTM은 역시 강력했다. '셀 상태(Cell State)'와 3개의 게이트(망각, 입력, 출력)를 통해 정보의 흐름을 정교하게 제어하는 구조가 인상 깊었다. RNN보다 복잡했지만, 그만큼 장기적인 패턴을 잘 학습해내는 것을 확인할 수 있었다.

> LSTM의 핵심은 **정보를 효과적으로 기억하고 잊는 능력**에 있다. 덕분에 멀리 떨어진 과거의 데이터가 현재 예측에 미치는 영향을 포착할 수 있다.

```python
# LSTM 모델 정의
class LSTM(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, seq_size):
        super(LSTM, self).__init__()
        self.lstm = nn.LSTM(input_size=input_size,
                            hidden_size=hidden_size,
                            num_layers=num_layers,
                            batch_first=True)
        self.fc1 = nn.Linear(in_features=seq_size * hidden_size, out_features=64)
        self.fc2 = nn.Linear(in_features=64, out_features=1)
        self.relu = nn.ReLU()

    def forward(self, x, h0, c0):
        x, (hn, cn) = self.lstm(x, (h0, c0))
        x = torch.reshape(x, (x.shape[0], -1))
        x = self.relu(self.fc1(x))
        x = self.fc2(x)
        return x
```

---

### ✨ 오늘의 회고

오늘은 시계열 데이터의 특징부터 시작해 DNN, CNN, RNN, 그리고 LSTM에 이르기까지 다양한 딥러닝 모델로 예측을 수행해봤다. 각 모델의 구조와 장단점을 코드로 직접 구현하며 비교해보니 이론으로만 배우던 것보다 훨씬 명확하게 이해할 수 있었다. 특히 RNN과 LSTM이 왜 시계열 데이터에 적합한지, 그 구조적 차이가 성능에 어떤 영향을 미치는지 체감할 수 있는 좋은 경험이었다.

다음에는 Transformer 기반의 모델(e.g., `Informer`, `Autoformer`)을 사용해 시계열 예측에 도전해보고 싶다. LSTM을 뛰어넘는 성능을 보여준다고 하니, 얼마나 더 정확한 예측이 가능할지 기대된다. 😄
