---
layout: post
title: "[BOJ] 10830 - 행렬 제곱"
date: 2025-09-17 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [divide-and-conquer, math, linear-algebra]
---

<br>

### 💻 백준 10830번: 행렬 제곱

- **문제 링크:** [https://www.acmicpc.net/problem/10830](https://www.acmicpc.net/problem/10830)
- **알고리즘 분류:** 분할 정복을 이용한 거듭제곱, 수학

## 문제 소개 🧐

크기가 N*N인 행렬 A와 거듭제곱할 횟수 B가 주어질 때, A의 B제곱을 구하는 문제다. B가 최대 1000억까지 가능하므로, 단순 반복으로는 해결할 수 없고 효율적인 거듭제곱 알고리즘이 필요하다.

### 입력 📝

- 첫째 줄: 행렬의 크기 N (2 ≤ N ≤ 5)과 지수 B (1 ≤ B ≤ 100,000,000,000)
- 둘째 줄부터 N개의 줄: 행렬 A의 원소 (0 이상 1,000 이하의 정수)

### 출력 📤

- 행렬 A를 B제곱한 결과의 각 원소를 1,000으로 나눈 나머지를 출력한다.

---

### 풀이: 분할 정복을 이용한 행렬 거듭제곱 🚀

#### Idea 1: 단순 반복

가장 먼저 생각할 수 있는 방법은 B번 만큼 행렬 곱셈을 반복하는 것이다. 행렬의 크기 N이 5로 매우 작기 때문에 혹시 가능하지 않을까 생각했다.

#### Code 1

```python
import sys

n, b = map(int, sys.stdin.readline().split())
mat = [list(map(int, sys.stdin.readline().split())) for _ in range(n)]

def pow_mat(matrix:list[list], n, b):

    new_mat = [row[:] for row in matrix]
    for _ in range(b-1):
        next_mat = [[0 for _ in range(n)] for _ in range(n)]
        for i in range(n):
            for j in range(n):
                for k in range(n):
                    next_mat[i][j] += new_mat[i][k] * matrix[k][j]
                next_mat[i][j] %= 1000
        new_mat = next_mat
    
    return new_mat

new_mat = pow_mat(mat, n, b)
for row in new_mat:
    print(*row)
```

#### Result 1

- **결과: 시간 초과 😭**
- B가 최대 1000억이므로, 시간 복잡도가 O(N³ * B)가 되어 당연히 시간 초과가 발생했다.

---

#### Idea 2: 분할 정복 적용

일반적인 거듭제곱 수를 구할 때 사용하는 분할 정복(Exponentiation by Squaring) 원리를 행렬 곱셈에 적용하기로 했다.

- `A^B`는 `A^(B/2) * A^(B/2)`와 같다.
- 만약 B가 홀수라면, `A^(B/2) * A^(B/2) * A`가 된다.

이 원리를 재귀적으로 적용하면 B번의 곱셈을 logB번으로 줄일 수 있다.

1.  **재귀 기본 단계:** B가 1이면 행렬 A를 그대로 반환한다.
2.  **분할:** `pow_mat(A, B/2)`를 재귀적으로 호출하여 `A^(B/2)`를 구한다.
3.  **정복:** 2에서 구한 결과를 자기 자신과 곱하여 `A^B` (B가 짝수일 때) 또는 `A^(B-1)` (B가 홀수일 때)를 계산한다.
4.  **조정:** B가 홀수이면, 3의 결과에 A를 한 번 더 곱해준다.

#### Code 2

```python
import sys

sys.setrecursionlimit(30000000)

n, b = map(int, sys.stdin.readline().split())
mat = [list(map(lambda x: int(x) % 1000, sys.stdin.readline().split())) for _ in range(n)]

def mul_mat(matrix1:list[list], matrix2:list[list], n):
    result = [[0 for _ in range(n)] for _ in range(n)]
    for i in range(n):
        for j in range(n):
            for k in range(n):
                result[i][j] += matrix1[i][k] * matrix2[k][j]
            result[i][j] %= 1000
    return result

def pow_mat(matrix:list[list], n, b):
    if b == 1:
        return matrix

    # 기본 거듭제곱 로직
    result = pow_mat(matrix, n, b // 2)
    result = mul_mat(result, result, n)

    # 만약 홀수면 곱하기 기본 행렬 추가
    if b % 2 == 1:
        result = mul_mat(result, matrix, n)
    
    return result

new_mat = pow_mat(mat, n, b)
for row in new_mat:
    print(*row)
```

#### Result 2

- **결과: 성공!** 🎉
- 시간 복잡도가 O(N³ * logB)로 크게 줄어들어 시간 내에 문제를 해결할 수 있었다.

---

## 배운 점 및 개선할 부분 🤔

- **거듭제곱의 분할 정복:** 지수가 매우 클 경우, 분할 정복을 이용한 거듭제곱은 필수적인 알고리즘이다. 이 원리가 일반 숫자뿐만 아니라 행렬 곱셈에도 동일하게 적용될 수 있음을 다시 한번 확인했다.
- **모듈러 연산의 위치:** 곱셈과 덧셈 연산이 누적될 때 수가 매우 커질 수 있으므로, 중간 계산 과정마다 모듈러 연산(`% 1000`)을 적용하여 오버플로우를 방지하고 계산량을 줄이는 것이 중요하다.
- **재귀 깊이:** Python에서 재귀를 사용할 때는 `sys.setrecursionlimit()`을 이용해 최대 재귀 깊이를 늘려주어야 `RecursionError`를 피할 수 있다. 이 문제에서는 B가 크지만 재귀 깊이는 logB에 비례하므로 사실 기본 제한으로도 충분했을 수 있다.

> B가 1000억? '이걸 어떻게 다 곱하지?' 라는 생각이 들면, 분할 정복을 떠올리자.

<div style="text-align: center">⁂</div>
