---
layout: post
title: "[BOJ] 1166 - 선물"
date: 2025-09-21 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [binary search, parametric search]
---

<br>

### 💻 문제: 선물 스페셜 저지

- **알고리즘 분류:** 이분 탐색(Binary Search), 매개 변수 탐색(Parametric Search)

## 문제 소개 🧐

N개의 정육면체 모양 작은 박스(A × A × A)를 큰 직육면체 박스(L × W × H)에 모두 넣으려고 한다. 작은 박스의 변은 큰 박스의 변과 평행해야 할 때, 가능한 A의 최댓값을 찾는 문제다.

| 시간 제한 | 메모리 제한 | 제출 | 정답 | 맞힌 사람 | 정답 비율 |
| --- | --- | --- | --- | --- | --- |
| 2 초 | 128 MB | 9319 | 2445 | 1696 | 24.222% |

### 입력 📝

- 첫째 줄에 네 정수 N, L, W, H가 주어진다. (1 ≤ N, L, W, H ≤ 1,000,000,000)

### 출력 📤

- 가능한 A의 최댓값을 출력한다. 절대/상대 오차는 10-9까지 허용한다.

---

### 풀이 1: 비율 기반 접근과 오차 문제 😭

#### Idea

처음에는 L, W, H의 비율을 이용해 문제를 풀어보려 했다. 전체 부피나 길이 합에 대한 각 변의 비율을 구하고, 그 비율에 따라 박스가 몇 개 들어갈지 계산하면 되지 않을까 생각했다. `L * W * H`를 직접 계산하면 자료형의 범위를 초과할 수 있으므로, 변의 길이를 비교하는 방식으로 접근했다.

하지만 이 방법은 변의 길이 비율이 극단적일 경우 부동소수점 연산에서 큰 오차를 발생시킬 수 있다는 약점이 있었다.

#### Code

```python
import sys
import math

MAX = 1000000000

N, L, H, W = map(int, sys.stdin.readline().split())

total_sum = sum([L, H, W])

total_sum

answer = 0

l_rate = L / total_sum
h_rate = H / total_sum
w_rate = W / total_sum

# N개의 정사각형, 한변이 answer 길이.

idx = 1
cnts = []
while True:
    l_cnt = math.floor(idx * l_rate)
    h_cnt = math.floor(idx * h_rate)
    w_cnt = math.floor(idx * w_rate)

    if l_cnt * h_cnt * w_cnt >= N:
        cnts.append(l_cnt)
        cnts.append(h_cnt)
        cnts.append(w_cnt)
        break
    idx +=1

answer = min(L/l_cnt, H/h_cnt, W/w_cnt)

print(f"{answer:}")
```

#### Result

- **결과: 25% 시간 초과** 😭
- 이 접근은 비율 계산 시 발생하는 부동소수점 오차 문제와 더불어, `idx`를 1씩 증가시키며 찾는 방식이 비효율적이어서 시간 초과를 유발했다. 최적의 해를 찾기에는 너무 느린 방법이었다.

---

### 풀이 2: 이분 탐색으로 최적해 찾기 💪

#### Idea 2

첫 시도의 문제점을 통해, 이 문제는 '가능한 A의 최댓값'을 찾는 최적화 문제이며, A의 값에 따라 'N개의 박스를 담을 수 있는가?'라는 결정 문제로 바꿀 수 있음을 깨달았다. 이는 전형적인 **매개 변수 탐색(Parametric Search)** 문제다.

-   A의 가능한 범위는 `0`부터 `min(L, W, H)`까지다.
-   이 범위 내에서 이분 탐색을 수행하며 중간값 `mid`를 A의 후보로 삼는다.
-   `mid`를 한 변의 길이로 할 때, `(L // mid) * (W // mid) * (H // mid)`개의 박스를 담을 수 있다.
-   이 값이 `N`보다 크거나 같으면, `mid`는 가능한 답이므로 더 큰 A값을 찾아보기 위해 탐색 범위를 오른쪽(`left = mid`)으로 옮긴다.
-   값이 `N`보다 작으면, `mid`가 너무 크다는 의미이므로 탐색 범위를 왼쪽(`right = mid`)으로 옮긴다.
-   이 과정을 충분히 반복하여 오차 범위 내의 최적해를 찾는다.

#### Code

```python
import sys
import math

MAX = 1000000000

N, L, H, W = map(int, sys.stdin.readline().split())

left = 0.0
right = min(L,H,W)
answer = 0.0
for _ in range(1000):
    mid = (left + right) / 2
    if mid == 0: # mid가 0이 되면 ZeroDivisionError 발생
        left = mid
        continue
    cnt = (int(L // mid) * int(H // mid) * int(W // mid))

    if cnt >= N:
        left = mid
        answer = mid

    else:
        right = mid

print(f"{answer:.10f}")
```

#### Result

- **결과: 성공!** 🎉
- 이분 탐색을 통해 매우 넓은 탐색 범위를 효율적으로 줄여나가며 정답을 찾을 수 있었다. 100번 정도의 반복으로도 충분히 10^-9 오차 범위 내의 답을 구할 수 있다.

---

## 배운 점 및 개선할 부분 🤔

-   **최적화 문제를 결정 문제로:** '최대/최소값을 구하라'는 최적화 문제는 종종 '어떤 값 X가 조건을 만족하는가?'를 묻는 결정 문제로 변환하여 이분 탐색으로 풀 수 있다. 이 문제 역시 'A의 최댓값'을 'A가 X일 때 N개의 상자를 담을 수 있는가?'라는 문제로 바꾸어 해결했다.
-   **부동소수점 이분 탐색:** 정수뿐만 아니라 실수 범위에서도 이분 탐색은 유효하다. 정답의 오차 범위가 주어졌을 때, 충분한 횟수(보통 100회)만큼 반복하면 정밀한 근사치를 얻을 수 있다.
-   **탐색 범위와 예외 처리:** 이분 탐색에서 `left`, `right`의 초기값 설정은 매우 중요하다. 또한 `mid`가 0이 되어 `ZeroDivisionError`를 발생시키는 경우와 같은 예외 상황을 항상 고려해야 한다.

> 최적화 문제를 결정 문제로 바꾸는 매개 변수 탐색, 잊지 말자!

<div style="text-align: center">⁂</div>
