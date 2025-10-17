---
layout: post
title: "[BOJ] 1082 - 방 번호"
date: 2025-10-07 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [greedy, dp]
---

<br>

### 💻 문제: 방 번호

- **문제 링크:** [https://www.acmicpc.net/problem/1082](https://www.acmicpc.net/problem/1082)
- **알고리즘 분류:** 그리디(Greedy), 다이나믹 프로그래밍(DP)

## 문제 소개 🧐

0부터 N-1까지의 숫자를 각각 다른 가격으로 구매할 수 있다. 주어진 예산 M원 내에서 만들 수 있는 가장 큰 방 번호를 찾는 문제다. 단, 0을 제외한 방 번호는 0으로 시작할 수 없다.

| 시간 제한 | 메모리 제한 | 제출 | 정답 | 맞힌 사람 | 정답 비율 |
| --- | --- | --- | --- | --- | --- |
| 2 초 | 128 MB | 8763 | 2975 | 2338 | 34.302% |

### 입력 📝

- 첫째 줄에 N이 주어진다. (1 ≤ N ≤ 10)
- 둘째 줄에는 각 숫자의 가격 P0, ..., PN-1이 주어진다. (1 ≤ Pi ≤ 50)
- 마지막 줄에는 예산 M이 주어진다. (1 ≤ M ≤ 50)

### 출력 📤

- 최대 M원을 사용해서 만들 수 있는 가장 큰 방 번호를 출력한다.

---

### Idea 1: 자리수 최대로 채우고 교체하기

#### Idea

가장 큰 수를 만들기 위한 그리디 접근법.

1.  **자리수 최대화:** 가장 저렴한 숫자를 구매하여 방 번호의 자리수를 최대한으로 늘린다.
2.  **앞자리부터 교체:** 남은 예산과 기존 숫자를 팔아서 생긴 예산을 합쳐, 가장 앞자리(가장 높은 자리수)부터 더 큰 숫자로 교체할 수 있는지 확인하고 교체한다.
3.  **0 처리:** 만약 가장 앞자리가 0이 되면, 0이 아닌 다른 숫자로 교체할 수 있도록 처리한다.

#### Code (1차 시도)

```python
import sys
from collections import deque

N = int(sys.stdin.readline().rstrip())
prices = list(map(int, sys.stdin.readline().split()))
M = int(sys.stdin.readline().rstrip())

if N == 1:
    print(0)
    exit()

price_num_dic = {}
for i in range(N):
    if price_num_dic.get(prices[i]):
        price_num_dic[prices[i]].append(i)
    else:
        price_num_dic[prices[i]] = [i]

# 1. 일단 제일 저렴한 수로 배열을 만듦 (자리수 최대)
prices.sort()
sorted_prices = deque()
sorted_prices.appendleft(prices.pop())
for i in range(N-1):
    price = prices.pop()
    if price != sorted_prices[0]:
        sorted_prices.appendleft(price)

answer = deque()
min_cost = sorted_prices[0]
cost = 0
while True:
    cost += min_cost
    if M < cost:
        cost -= min_cost
        break
    answer.append(max(price_num_dic[min_cost]))
    
if len(sorted_prices) == 1:
    print(*answer, sep='')
    exit()

# 만약 첫 자리가 0 이면 숫자가 아니니까, 첫자리를 손봐줌
available_cost = M - cost
if answer[0] == 0 and len(answer) != 1:
    # 0이 아닌 숫자를 살 수 있을때까지, 0을 없애고, 비용을 확보해줌
    while answer and available_cost < sorted_prices[1]:
        answer.pop()
        available_cost += sorted_prices[0]

    new_num = answer[0]
    new_cost = 0
    # 사용가능한 비용으로 쓸 수 있는 가장 큰 값을 찾아줌
    for i in range(1, N):
        if sorted_prices[i] <= available_cost:
            if new_num < max(price_num_dic[sorted_prices[i]]):
                new_num = max(price_num_dic[sorted_prices[i]])
                new_cost = sorted_prices[i]
    answer.appendleft(new_num)
    available_cost -= new_cost

# 남은 사용가능한 비용으로 자릿수를 바꾸지 않는 선에서 수를 더 키울 수 있으면 키워줌
idx = 1
while len(sorted_prices) != 1 and available_cost >= (sorted_prices[1] - sorted_prices[0]):
    available_cost += sorted_prices[0]
    new_num = answer[idx]
    new_cost = 0
    for i in range(1, N):
        if sorted_prices[i] <= available_cost:
            if new_num < max(price_num_dic[sorted_prices[i]]):
                new_num = max(price_num_dic[sorted_prices[i]])
                new_cost = sorted_prices[i]
    answer[idx] = new_num
    available_cost -= new_cost

print(*answer, sep='')
```

#### Result

-   **결과: 인덱스 에러** 😭
-   **문제점:** 로직이 지나치게 복잡하다. 가장 싼 값으로 모두 채운 뒤 교체하는 방식은 여러 예외 케이스(0 처리, 비용 계산 등)를 고려해야 하므로 코드가 길어지고 오류 발생 가능성이 높다. 특히 `deque`에서 원소를 제거하고 비용을 다시 계산하는 과정에서 인덱스 관련 문제가 발생한 것으로 보인다.

---

### Idea 2: 자리수 고정 후 앞자리부터 결정하기 ✨

#### Idea

접근 방식을 바꿔, 자리수를 먼저 확정하고 가장 큰 수부터 차례대로 결정한다.

1.  **최대 자리수 결정:** 먼저 만들 수 있는 방 번호의 최대 길이를 계산한다. 가장 싼 숫자로만 구성했을 때의 길이가 기준이 되지만, 맨 앞자리가 0이 될 수 없는 경우를 고려하여 유효한 최대 길이를 찾는다.
2.  **앞자리부터 그리디 선택:** 결정된 길이만큼 반복하면서, 각 자리에 올 수 있는 가장 큰 숫자(N-1부터 0까지)를 선택한다.
3.  **유효성 검사:** 특정 숫자를 현재 자리에 놓았을 때, 남은 예산으로 나머지 자리들을 모두 채울 수 있는지(`(남은 길이) * (최소 비용)`)를 확인한다. 이 조건이 만족되면 해당 숫자를 선택하고 다음 자리로 넘어간다.

이 방식은 각 자리를 결정할 때마다 항상 최선의(가장 큰 숫자를 넣는) 선택을 하면서도, 그 선택이 최종적으로 유효한 번호를 만들 수 있도록 보장한다.

#### Code (2차 시도)

```python
import sys
from collections import deque

# 입력 처리
N = int(sys.stdin.readline().rstrip())
prices = list(map(int, sys.stdin.readline().split()))
M = int(sys.stdin.readline().rstrip())

# 숫자 가격-번호 매핑 (같은 가격 여러 개일 수 있음)
price_num_dic = {}
for i in range(N):
    price_num_dic.setdefault(prices[i], []).append(i)

# 가장 싼 가격
min_cost = min(prices)

# 아무것도 살 수 없는 경우
if M < min_cost:
    print(0)
    exit()

# 가능한 최대 자릿수
max_length = M // min_cost
if max_length == 0:
    print(0)
    exit()

# 가능한 길이 중 유효한 최대 길이 찾기
# (맨 앞자리가 0이 아닌 숫자로 만들 수 있는지)
valid_length = 0
for l in range(max_length, 0, -1):
    if l == 1:
        feasible = any(prices[d] <= M for d in range(N))
    else:
        feasible = any(prices[d] + (l - 1) * min_cost <= M for d in range(1, N))
    if feasible:
        valid_length = l
        break

if valid_length == 0:
    print(0)
    exit()

# 정답 수를 deque으로 저장
answer = deque()
remain = M

for i in range(valid_length):
    # 자리별 숫자 선택 (큰 숫자부터)
    for d in range(N - 1, -1, -1):
        # 첫 자리가 0이면 안 됨 (단, 길이 1인 경우 허용)
        if i == 0 and d == 0 and valid_length > 1:
            continue
        cost = prices[d]
        # 이 숫자를 선택했을 때 남은 자리들을 최소비용으로 채울 수 있는가?
        if remain - cost >= (valid_length - i - 1) * min_cost:
            answer.append(d)
            remain -= cost
            break

print(*answer, sep='')
```

#### Result

-   **결과: 성공!** 🎉
-   첫 번째 접근법보다 훨씬 간결하고 명확하다. 최대 길이를 먼저 계산하고, 높은 자리수부터 가장 큰 숫자를 탐욕적으로 선택하는 전략이 유효했다. 각 단계에서 남은 자리를 채울 수 있는지 확인하는 조건이 핵심이다.

---

## 배운 점 및 개선할 부분 🤔

-   **그리디 문제의 핵심 원칙:** "가장 큰 수"를 만드는 문제에서는 보통 두 가지 그리디 전략을 고려할 수 있다.
    1.  **자리수를 최우선으로:** 숫자의 크기는 자리수가 많을수록 커진다. 따라서 먼저 최대 자리수를 확보하는 것이 중요하다.
    2.  **높은 자리수부터 결정:** 자리수가 같다면, 가장 왼쪽(가장 높은 자리)에 큰 숫자가 올수록 전체 수가 커진다.
-   **접근법의 차이:** 1차 시도는 '일단 채우고 수정'하는 방식이었고, 2차 시도는 '조건을 확인하며 처음부터 제대로 채우는' 방식이었다. 후자가 더 직접적이고 간단한 그리디 해법으로 이어졌다.
-   **유효성 검사의 중요성:** 그리디 선택을 할 때, 그 선택이 미래의 선택에 영향을 주어 전체 해를 구하지 못하게 되는 경우는 아닌지 항상 검토해야 한다. 2차 시도에서 `remain - cost >= (valid_length - i - 1) * min_cost` 부분이 바로 그 유효성 검사 역할을 한다.

> 복잡한 수정 로직 대신, 처음부터 올바른 값을 구성해나가는 방식이 더 효과적일 때가 많다. 이 문제는 그 전형적인 예시이다.

<div style="text-align: center">⁂</div>
