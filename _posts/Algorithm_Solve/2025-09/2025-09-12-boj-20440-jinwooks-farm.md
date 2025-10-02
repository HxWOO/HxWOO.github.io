---
layout: post
title: "[BOJ] 20440 - 진욱이의 농장"
date: 2025-09-12 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [brute-force, prefix-sum, binary-search, dp, implementation]
---

<br>

### 💻 백준 20440번: 진욱이의 농장

- **문제 링크:** [https://www.acmicpc.net/problem/20440](https://www.acmicpc.net/problem/20440)
- **알고리즘 분류:** 브루트포스, 누적 합, 이분 탐색, DP

## 문제 소개 🧐

N×N 크기의 농장이 있고, 처음에는 모두 0번 과일이 심어져 있다. M번 씨앗을 뿌리는데, (X, Y)를 왼쪽 위 모서리로 하고 변의 길이가 L인 정사각형 구역에 F번 과일 씨앗을 덮어씌운다.

이 농장에서 **과일 종류가 최대 두 종류**이고, **0번 과일은 포함되지 않는** 정사각형 모양의 구역을 잘라내려고 할 때, 얻을 수 있는 가장 넓은 농장의 넓이를 구하는 문제다.

### 입력 📝

- 첫째 줄: 농장 크기 N (1 ≤ N ≤ 1,000), 씨앗 뿌린 횟수 M (1 ≤ M ≤ 50)
- 둘째 줄부터 M개 줄: X, Y, L, F (씨앗 정보)

### 출력 📤

준규가 가져갈 수 있는 가장 넓은 정사각형의 넓이를 출력한다.

### 제한 ❌

- 시간 제한: 2초
- 메모리 제한: 128MB

---

### 첫 번째 시도: 브루트포스 냅다 박기 🔥

#### Idea

처음에는 문제가 잘 이해되지 않아, 가장 단순한 방법부터 시도해보기로 했다. 가능한 가장 큰 정사각형부터 크기를 1씩 줄여가면서, 농장의 모든 위치를 시작점으로 하여 해당 정사각형이 조건을 만족하는지 확인하는 완전 탐색 방법이다.

- 정사각형의 한 변의 길이를 `L`이라고 할 때, `L`을 `N`부터 `1`까지 감소시킨다.
- 모든 시작점 `(i, j)`에 대해 `L x L` 크기의 정사각형을 검사한다.
- 정사각형 내부에 0번 과일이 있거나, 과일 종류가 3종류 이상이 되면 조건을 만족하지 않는다.
- 조건을 만족하는 첫 번째 `L`을 찾으면, 그 넓이 `L*L`을 출력하고 즉시 종료한다.

#### Code

```python
import sys
from collections import Counter

N, M = map(int, sys.stdin.readline().split())

field = [[0 for _ in range(N)] for _ in range(N)]

for _ in range(M):
    x, y, l, f = map(int, sys.stdin.readline().split())
    for i in range(x, x+l):
        for j in range(y, y+l):
            field[i][j] = f

for l in range(N, 0, -1):  # 길이 줄여가며 비교
    for i in range(0, N-l+1):  # 비교 시작 행
        for j in range(0, N-l+1):  # 비교 시작 열 
            can_give = True
            seeds = set()
            for k in range(i, i+l):
                # 씨앗 셋에 그 행에 있는 씨앗 종류를 union
                seeds.update(field[k][j:j+l])
                if 0 in seeds or len(seeds) > 2:
                    can_give = False
                    break
            
            if can_give:
                print(l*l)
                exit()
```

#### Result

- **결과: 시간 초과** 😭
- 당연한 결과였다. 이 알고리즘의 시간 복잡도는 대략 `O(N^4)`에 육박한다. (길이 `L` * 시작행 `i` * 시작열 `j` * 내부순회 `L*L`의 일부). N이 1,000일 때 이런 알고리즘은 나도 처음 짜보는 것 같다.

---

### 두 번째 시도: 이분 탐색과 누적 합으로 개선하기 🚀

#### Idea

시간 복잡도를 줄이기 위해, 문제의 구조를 바꿔보기로 했다. 정답(정사각형의 넓이)을 직접 찾는 대신, **"한 변의 길이가 L인 조건을 만족하는 정사각형이 존재하는가?"** 라는 결정 문제로 바꾸고 **이분 탐색**을 적용하는 것이다.

- `1`부터 `N`까지의 길이를 대상으로 이분 탐색을 수행한다.
- `valid(L)` 함수는 길이가 `L`인 정사각형이 존재하는지 판별한다.
- `valid(L)` 함수 내부에서 0번 과일의 존재 여부를 빠르게 확인하기 위해 **2D 누적 합**을 사용했다. 특정 구역의 0의 개수를 `O(1)`에 알 수 있다.

하지만 과일 종류가 2개 이하인지 확인하는 부분은 여전히 모든 칸을 순회해야 했다.

#### Code

```python
# ... (씨 뿌리기 및 누적 합 배열 생성 부분은 위와 유사) ...

# 한 변 길이 l 정사각형이 조건을 만족하는지 검사
def valid(l):
    limit = N - l + 1
    for i in range(limit):
        for j in range(limit):
            x2, y2 = i + l - 1, j + l - 1

            # 0이 1개라도 있으면 불가능 (누적 합으로 O(1)에 체크)
            if count_zero(i, j, x2, y2) > 0:
                continue

            # 씨앗 종류 판정 (여전히 O(l*l)이 걸림)
            kinds = set()
            can_sell = True
            for x in range(i, i + l):
                for y in range(j, j + l):
                    kinds.add(field[x][y])
                    if len(kinds) > 2:
                        can_sell = False
                        break
                if not can_sell:
                    break
            
            if can_sell:
                return True
    return False

# 이분 탐색 실행
lo, hi = 1, N
best = 0
while lo <= hi:
    mid = (lo + hi) // 2
    if valid(mid):
        best = mid
        lo = mid + 1
    else:
        hi = mid - 1

print(best * best)
```

#### Result

- **결과: 또 시간 초과...** 😭
- 아까보단 나아졌지만, `valid` 함수 내부에서 과일 종류를 체크하는 부분이 `O(L^2)`의 복잡도를 가져 전체적으로 `O(N^2 * L^2 * logN)` 수준의 시간 복잡도가 발생했다. 이것이 파이썬의 한계인가 하는 생각도 잠시 들었다.

---

## 배운 점 및 개선할 부분 🤔

두 번의 시도가 모두 시간 초과된 근본적인 원인은, 정사각형 내의 **'과일 종류'**를 확인하는 과정이 너무 오래 걸렸기 때문이었다.

**핵심은 과일 종류(F)가 7개로 매우 적다는 점을 활용하는 것이었다.**

### 정답 풀이 아이디어

1.  먼저, 준규가 가져갈 수 있는 과일 종류의 **모든 조합**을 구한다. (최대 2종류)
    -   한 종류만 있는 경우: `{1}, {2}, ..., {7}` (7가지)
    -   두 종류가 있는 경우: `{1, 2}, {1, 3}, ..., {6, 7}` (21가지)
    -   총 28가지의 조합이 나온다. 이는 상수 시간으로 취급할 수 있다.
2.  각 조합에 대해, **해당 과일들로만 구성된 가장 큰 정사각형**을 찾는다.
3.  '특정 종류의 과일들로만 구성된 가장 큰 정사각형'을 찾는 것은 `O(N^2)` **DP**로 해결할 수 있는 유명한 문제다.
    -   `dp[i][j]` = `(i, j)`를 우측 하단 꼭짓점으로 하는, 조건을 만족하는 가장 큰 정사각형의 한 변의 길이
    -   `field[i][j]`가 해당 과일 조합에 속한다면, `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1` 점화식을 적용할 수 있다.

이 접근법의 전체 시간 복잡도는 `O(F^2 * N^2)`이 되어, 시간 내에 충분히 문제를 해결할 수 있다. 과일 종류가 적다는 제한 조건을 어떻게 활용할지가 문제 해결의 열쇠였다.

> 결국, 문제의 핵심 제약 조건을 파악하고 그에 맞는 알고리즘(DP)을 설계하는 능력이 중요하다는 것을 다시 한번 깨달았다.

<div style="text-align: center">⁂</div>
