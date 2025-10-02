---
layout: post
title: "[BOJ] 사전 순 최대 공통 부분 수열"
date: 2025-09-18 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [greedy, dp, lcs]
---

<br>

### 💻 문제: 사전 순 최대 공통 부분 수열

- **문제 링크:** (문제 출처를 찾지 못했습니다)
- **알고리즘 분류:** 그리디, 동적 프로그래밍

## 문제 소개 🧐

두 수열 A와 B가 주어졌을 때, 공통으로 나타나는 부분 수열 중에서 사전 순으로 가장 나중에 오는 것을 찾는 문제다. '사전 순으로 나중'이라는 것은 앞자리부터 비교했을 때 더 큰 수가 오는 것을 의미한다.

### 입력 📝

- 첫 줄: 수열 A의 길이 N (1 ≤ N ≤ 100)
- 둘째 줄: 수열 A의 원소들 (1 ≤ Ai ≤ 100)
- 셋째 줄: 수열 B의 길이 M (1 ≤ M ≤ 100)
- 넷째 줄: 수열 B의 원소들 (1 ≤ Bi ≤ 100)

### 출력 📤

- 사전 순으로 가장 나중인 공통 부분 수열의 크기 K와 그 원소들을 출력한다.

---

### 풀이: 두 가지 접근법과 깨달음 🚀

#### Idea 1: LCS 기반 접근

처음에는 '최대 공통 부분 수열(LCS)'이라는 키워드에 꽂혀서 DP로 접근했다. LCS를 구하는 표준적인 알고리즘을 사용하여 가장 긴 공통 부분 수열을 먼저 찾고, 그 안에서 사전 순으로 가장 나중인 것을 찾으려고 시도했다.

#### Code 1

```python
import sys
from collections import deque
from itertools import combinations

N = int(sys.stdin.readline().rstrip())
A = deque(map(int, sys.stdin.readline().split()))
A.appendleft('')

M = int(sys.stdin.readline().rstrip())
B = deque(map(int, sys.stdin.readline().split()))
B.appendleft('')

def get_lcs_length():
    global A, B, N, M
    lcs = [[0 for _ in range(M+1)] for _ in range(N+1)]

    for i in range(1, N+1):
        for j in range(1, M+1):
            if A[i] == B[j]:
                lcs[i][j] = lcs[i-1][j-1] + 1
            else:
                lcs[i][j] = max(lcs[i-1][j], lcs[i][j-1])

    return lcs

def get_lcs_list(lcs):
    global A, B, N, M
    result = deque()

    i, j = N, M
    while i > 0 and j > 0:
        if lcs[i][j] == lcs[i-1][j]:
            i -= 1
        elif lcs[i][j] == lcs[i][j-1]:
            j -= 1
        else:
            result.appendleft(A[i])
            i -= 1
            j -= 1
    return list(result)

def get_last_dict(lcs_list):
    idx = 0
    result = []
    while idx < len(lcs_list):
        for i in range(idx, len(lcs_list)):
            if i == idx:
                result.append(lcs_list[i])
            elif result[-1] < lcs_list[i]:
                result.pop()
                result.append(lcs_list[i])
                idx = i

        idx += 1
    return result

lcs = get_lcs_length()

lcs_list = get_lcs_list(lcs)
if not lcs_list:
    print(0)
    exit()

answer = get_last_dict(lcs_list)
print(len(answer))
print(*answer)
```

#### Result 1

- **결과: 실패 😭**
- 이 접근은 '가장 긴' 공통 부분 수열을 찾는 데는 유효하지만, '사전 순으로 가장 나중인' 수열을 찾는 것과는 목표가 다르다. 예를 들어, `{1, 9}`와 `{8, 9}`는 둘 다 공통 부분 수열일 수 있지만, 사전 순으로는 `{8, 9}`가 더 나중이다. LCS는 길이에만 초점을 맞추므로 이 문제를 해결할 수 없었다.

---

#### Idea 2: 그리디 알고리즘

'사전 순으로 가장 나중'이라는 조건에 집중하여 다시 생각했다. 가장 앞에 오는 수가 클수록 좋으므로, 현재 두 수열에서 만들 수 있는 공통 원소 중 가장 큰 값을 먼저 선택하는 것이 최적이라는 결론에 도달했다.

1.  두 수열 `arr1`, `arr2`가 모두 비어있지 않은 동안 반복한다.
2.  `arr1`과 `arr2` 각각에서 가장 큰 값을 찾는다.
3.  두 최댓값이 같다면, 이 값이 현재 시점에서 선택할 수 있는 최선의 수다. 결과에 추가하고, 두 수열에서 이 값이 나타난 위치 다음부터의 부분 수열을 새로운 `arr1`, `arr2`로 삼는다.
4.  두 최댓값이 다르다면, 더 작은 최댓값을 가진 수열에서는 그 값이 현재 최선이 될 수 없으므로 해당 원소를 버린다.

이 과정을 반복하면 자연스럽게 사전 순으로 가장 나중인 공통 부분 수열이 만들어진다.

#### Code 2

```python
import sys

N = int(sys.stdin.readline().rstrip())
A = list(map(int, sys.stdin.readline().split()))

M = int(sys.stdin.readline().rstrip())
B = list(map(int, sys.stdin.readline().split()))

def solve(arr1: list, arr2: list):
    result = []

    while arr1 and arr2:
        max_arr1 = max(arr1)
        max_arr1_idx = arr1.index(max_arr1)

        max_arr2 = max(arr2)
        max_arr2_idx = arr2.index(max_arr2)

        if max_arr1 == max_arr2:
            result.append(max_arr1)
            arr1 = arr1[max_arr1_idx+1:]
            arr2 = arr2[max_arr2_idx+1:]
        elif max_arr1 > max_arr2:
            arr1.pop(max_arr1_idx)
        else:
            arr2.pop(max_arr2_idx)
    
    return result
    
answer = solve(A, B)
print(len(answer))
print(*answer)
```

#### Result 2

- **결과: 성공!** 🎉
- 그리디한 접근법이 문제의 핵심을 정확히 꿰뚫었다. 각 단계에서 최선의 선택(가장 큰 공통 원소)을 하는 것이 전체의 최적해로 이어졌다.

---

## 배운 점 및 개선할 부분 🤔

- **문제의 본질 파악하기:** '최대 공통 부분 수열'이라는 단어에 매몰되어 DP로만 접근하려 했던 것이 첫 번째 패착이었다. 문제의 진짜 요구사항은 '길이'가 아니라 '사전 순서'임을 명확히 인지하는 것이 중요했다.
- **그리디 알고리즘의 힘:** 어떤 기준으로 '가장 좋은' 것을 선택해야 하는지 명확할 때, 그리디 알고리즘은 매우 강력하고 직관적인 해법이 될 수 있다. '사전 순으로 가장 나중'이라는 조건은 '앞에 오는 숫자가 클수록 좋다'는 명확한 기준을 제시했다.
- **고정관념 탈피:** LCS = DP라는 고정관념에서 벗어나 문제의 조건을 다시 한번 곱씹어보는 자세가 필요함을 느꼈다.

> 익숙한 이름에 속지 말고, 문제의 진짜 얼굴을 보자.

<div style="text-align: center">⁂</div>
