---
layout: post
title: "[BOJ] 1049 - 기타줄"
date: 2025-09-23 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [greedy, math]
---

<br>

### 💻 문제: 기타줄

- **문제 링크:** [https://www.acmicpc.net/problem/1049](https://www.acmicpc.net/problem/1049)
- **알고리즘 분류:** 그리디 알고리즘(Greedy), 수학(Math)

## 문제 소개 🧐

기타 줄이 N개 끊어졌을 때, M개 브랜드의 패키지(6줄) 가격과 낱개 가격을 보고 N개 이상의 기타줄을 사는 데 필요한 최소 비용을 계산하는 문제다.

| 시간 제한 | 메모리 제한 | 제출 | 정답 | 맞힌 사람 | 정답 비율 |
| --- | --- | --- | --- | --- | --- |
| 2 초 | 128 MB | 37707 | 16142 | 14039 | 42.790% |

### 입력 📝

- 첫째 줄에 끊어진 기타줄의 개수 N (1 ≤ N ≤ 100)과 브랜드 개수 M (1 ≤ M ≤ 50)이 주어진다.
- 둘째 줄부터 M개의 줄에 각 브랜드의 패키지 가격과 낱개 가격이 주어진다. (가격 ≤ 1,000)

### 출력 📤

- 기타줄을 N개 이상 사는 데 필요한 최소 비용을 출력한다.

---

### 첫 번째 시도: 복잡한 조건 분기 😭

#### Idea

가장 저렴한 패키지 가격(`min_p`)과 가장 저렴한 낱개 가격(`min_s`)을 먼저 찾았다. 그 후, 낱개를 1개부터 6개까지 살 때, 패키지 가격보다 저렴한 임계점(`affordable_quantity`)을 찾아서 비용을 계산하려 했다.

예를 들어, 낱개 5개까지는 사는 게 패키지보다 저렴하고 6개부터는 패키지가 더 저렴하다면, `affordable_quantity`는 5가 된다. 이 값을 기준으로 패키지 구매 개수와 낱개 구매 개수를 조합해 계산하는 로직을 세웠다.

#### Code

```python
import sys

N, M = map(int, sys.stdin.readline().split())
answer = 0

min_s = float('inf')
min_p = float('inf')
for i in range(M):
    p, s = map(int, sys.stdin.readline().split())
    min_s = min(min_s, s)
    min_p = min(min_p, p)

# 1. 낱개를 affordable_quantity 개 살때까지 패키지 사는거 보다 나은지 확인
affordable_quantity = 0
for i in range(1, 7):
    if min_s * i <= min_p:
        affordable_quantity = i
    else:
        break

# 2-1. 6줄이어도 패키지보다 낱개로만 사는게 나은 경우
if affordable_quantity == 6:
    answer = N * min_s
else:  # 패키지로 6줄 씩 삼
    pack_qt = N // 6
    answer = pack_qt * min_p
    sing_qt = N % 6
    if sing_qt < affordable_quantity:
        answer += sing_qt * min_s
    else:
        answer += min_p

print(answer)
```

#### Result

- **결과: 실패** 😭
- 로직이 너무 복잡했고, 모든 경우를 제대로 처리하지 못했다.

---

### 두 번째 시도: 단순한 경우의 수 비교 ✨

#### Idea

실패 후, 접근법을 단순화하기로 했다. 최소 비용이 될 수 있는 경우는 결국 아래 3가지 시나리오 중 하나라는 것을 깨달았다.

1.  **오직 낱개로만 N개를 구매하는 경우:** `min_s * N`
2.  **오직 패키지로만 N개 이상을 구매하는 경우:** `min_p * (N // 6 + 1)` (남은 줄이 있어도 패키지로 사는 게 더 쌀 수 있다)
3.  **패키지와 낱개를 조합하여 구매하는 경우:** `min_p * (N // 6) + min_s * (N % 6)`

이 세 가지 값을 모두 계산한 뒤, 그중 가장 작은 값을 선택하면 된다. 이 접근법은 모든 가능성을 포함하면서 훨씬 간결하다.

#### Code

```python
import sys

N, M = map(int, sys.stdin.readline().split())
answer = 0

min_s = float('inf')
min_p = float('inf')
for i in range(M):
    p, s = map(int, sys.stdin.readline().split())
    min_s = min(min_s, s)
    min_p = min(min_p, p)

# 아래 3가지 경우중에 가장 저렴한 경우가 있음
print(min(min_s * N, min_p*(N//6 + 1), min_p*(N//6) + min_s*(N%6)))
```

#### Result

- **결과: 성공!** 🎉
- 복잡한 조건문 없이, 가능한 모든 구매 전략의 최종 비용을 직접 비교하는 것이 훨씬 효과적이고 정확했다.

---

## 배운 점 및 개선할 부분 🤔

-   **그리디, 복잡함보다 단순함:** 그리디 알고리즘 문제를 풀 때, 복잡한 조건 분기로 최적의 해를 찾으려고 하기보다, 최적 해가 될 수 있는 몇 가지 단순한 시나리오를 정의하고 그들을 비교하는 것이 더 나은 전략일 수 있다.
-   **핵심 경우의 수 식별:** 이 문제의 핵심은 (1) 전부 낱개, (2) 전부 패키지, (3) 패키지 + 낱개 조합이라는 세 가지 큰 틀을 발견하는 것이었다. 이 틀 안에서 최소값을 찾는 것이 문제 해결의 지름길이었다.
-   **엣지 케이스를 포함하는 일반화:** 두 번째 접근법은 `min_p`와 `min_s * 6`의 대소 관계 같은 엣지 케이스를 따로 고려할 필요 없이, 최종 비용만 비교하면 되므로 자연스럽게 모든 경우를 포괄하게 된다.

> 때로는 가장 단순하고 명료한 접근이 가장 강력한 법이다.

<div style="text-align: center">⁂</div>
