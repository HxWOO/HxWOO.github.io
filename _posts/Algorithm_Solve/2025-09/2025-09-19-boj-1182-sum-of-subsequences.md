---
layout: post
title: "[BOJ] 1182 - 부분수열의 합"
date: 2025-09-19 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [bruteforce, combinations, itertools]
---

<br>

### 💻 문제: 1182 - 부분수열의 합

- **문제 링크:** [https://www.acmicpc.net/problem/1182](https://www.acmicpc.net/problem/1182)
- **알고리즘 분류:** 브루트포스, 백트래킹

## 문제 소개 🧐

N개의 정수로 이루어진 수열에서, 크기가 양수인 부분수열 중 그 원소들의 합이 S가 되는 경우의 수를 찾는 문제다.

| 시간 제한 | 메모리 제한 | 제출 | 정답 | 맞힌 사람 | 정답 비율 |
| --- | --- | --- | --- | --- | --- |
| 2 초 | 256 MB | 108624 | 49335 | 32138 | 43.416% |

### 입력 📝

- 첫째 줄: 정수의 개수 N과 합 S (1 ≤ N ≤ 20, |S| ≤ 1,000,000)
- 둘째 줄: N개의 정수

### 출력 📤

- 합이 S가 되는 부분수열의 개수를 출력한다.

---

### 풀이: 모든 조합을 탐색하는 브루트포스 💪

#### Idea: `itertools.combinations` 활용

N의 크기가 최대 20으로 비교적 작다는 점에서 모든 부분수열을 탐색하는 브루트포스 접근이 가능하다고 판단했다.

1.  Python의 `itertools` 라이브러리에 있는 `combinations` 함수를 사용한다.
2.  `for` 루프를 통해 1부터 N까지의 모든 길이에 대해 가능한 모든 부분수열 조합을 생성한다.
3.  각 조합의 합을 계산하여, 그 합이 S와 일치하는 경우의 수를 센다.

이 방법은 모든 경우의 수를 직접 확인하므로 간단하고 직관적이다.

#### Code

```python
import sys
from itertools import combinations

N, S = map(int, sys.stdin.readline().split())

num_list = list(map(int, sys.stdin.readline().split()))

sum_dict = {}

for i in range(1, N):
    com_list = list(combinations(num_list, i))
    for li in com_list:
        total_num = sum(li)
        if not sum_dict.get(total_num):
            sum_dict[total_num] = 1
        else:
            sum_dict[total_num] += 1
    

offset = 0
if S == sum(num_list):
    offset += 1

if sum_dict.get(S):
    print(sum_dict.get(S) + offset)
else:
    print(offset)
```

#### Result

- **결과: 성공!** 🎉
- 모든 조합을 탐색하는 방법으로 문제를 해결할 수 있었다.

---

## 배운 점 및 개선할 부분 🤔

- **문제의 제약 조건 확인:** N이 20으로 작았기 때문에, 2^20 (약 100만) 정도의 모든 부분수열을 탐색하는 것이 시간 내에 가능하다는 것을 알 수 있었다. 제약 조건을 보고 풀이 접근법을 선택하는 것이 중요하다.
- **`itertools`의 강력함:** `combinations`와 같은 내장 라이브러리를 사용하면 복잡한 조합 로직을 직접 구현하지 않고도 간결하게 코드를 작성할 수 있다.
- **코드 개선점:** 현재 코드는 크기 N인 부분수열(전체 수열)을 별도로 처리하고 있다. `for i in range(1, N+1):` 과 같이 반복 범위를 수정하면 `offset` 변수 없이 더 일관되고 간결한 코드를 작성할 수 있을 것이다. 이런 작은 리팩토링이 코드의 가독성을 높인다.

> 작은 N은 완전 탐색을 의심해보자.

<div style="text-align: center">⁂</div>
