---
layout: post
title:  "[Algorithm] N-Queen 문제와 알고리즘 완벽 정리"
date:   2025-08-27 00:00:00 +0900
categories: Computer_Science
tags: [n-queen, backtracking, algorithm, python]
---

# 👑 N-Queen 문제와 알고리즘 회고

---

## 🧐 1. N-Queen 문제란?

**N-Queen 문제**는 알고리즘 공부할 때 꼭 한 번씩은 풀어보게 되는 고전적인 문제였다.

- 체스판 크기: `N × N`  
- 조건: `N`개의 퀸(Queen)을 서로 공격할 수 없게 놓아야 한다.
- 퀸의 공격 범위:
  - 같은 행(Row)
  - 같은 열(Column)
  - 같은 대각선(Diagonal)  

한마디로, **같은 행, 열, 대각선에 퀸이 하나만 있도록** 배치하는 방법을 찾는 문제다.

---

## 🧠 2. 어떻게 풀었나? 접근 방법 회고

### 2.1 완전 탐색 (Brute Force)
- 그냥 모든 경우를 다 해보는 방법. 각 행마다 N개의 선택지가 있으니 `N^N`이다.
- `N=8`만 해도 `16,777,216`... 이건 아니다 싶었다. 😭
- 너무 비효율적이라 바로 포기했다.

### 2.2 백트래킹 (Backtracking)
- 이 문제의 정석은 백트래킹이었다. ✅
- 한 행씩 퀸을 놓아보다가, 조건에 안 맞으면 바로 그 길은 버리고 다른 길을 찾는다. (가지치기!)
- 덕분에 모든 경우를 다 보지 않고, 안 될 것 같은 길은 미리 포기할 수 있었다.

이론상 복잡도는 `N!`이지만, 가지치기 덕분에 실제로는 훨씬 빨랐다.

---

## 📜 3. 기본 알고리즘 (재귀적 백트래킹)

가장 기본 N-Queen 해법이다.

```python
def solve_n_queens(N):
    board = [-1] * N   # board[i] = i번째 행에 있는 퀸의 열 위치
    results = []

    def is_safe(row, col):
        # 이전에 놓았던 퀸들과 충돌하는지 확인했다.
        for r in range(row):
            c = board[r]
            # 같은 열이거나 대각선에 있으면 실패!
            if c == col or abs(c - col) == abs(r - row):
                return False
        return True

    def backtrack(row):
        if row == N:
            # 끝까지 다 놓았으면 성공!
            results.append(board[:])
            return
        
        for col in range(N):
            if is_safe(row, col):
                board[row] = col
                backtrack(row + 1) # 다음 행으로 이동
                board[row] = -1  # 원래대로 돌려놓기 (백트래킹)

    backtrack(0)
    return results
```

`N=4`로 실행해보니 결과는 잘 나왔다.
```
[1, 3, 0, 2]
[2, 0, 3, 1]
```
배열을 그림으로 풀면 이렇다
```
=== [1, 3, 0, 2] ===
. Q . .
. . . Q
Q . . .
. . Q .

=== [2, 0, 3, 1] ===
. . Q .
Q . . .
. . . Q
. Q . .
```

---

## ✨ 5. set을 활용한 최적화

`is_safe` 함수가 매번 O(N)의 시간이 걸리는 게 마음에 걸렸다.
**집합(set)** 자료구조를 써서 O(1) 만에 충돌 검사를 하도록 최적화했다.

```python
def solve_n_queens_optimized(N):
    results = []
    board = [-1] * N

    cols = set()   # 사용 중인 열
    diag1 = set()  # ↘ 대각선 (row+col)
    diag2 = set()  # ↗ 대각선 (row-col)

    def backtrack(row):
        if row == N:
            results.append(board[:])
            return
        
        for col in range(N):
            # set으로 충돌 여부를 O(1)만에 확인!
            if col in cols or (row + col) in diag1 or (row - col) in diag2:
                continue
            
            board[row] = col
            cols.add(col)
            diag1.add(row + col)
            diag2.add(row - col)

            backtrack(row + 1)

            # 백트래킹: 상태 원상복구
            board[row] = -1
            cols.remove(col)
            diag1.remove(row + col)
            diag2.remove(row - col)

    backtrack(0)
    return results
```

이 방법으로 대각선 충돌 여부를 O(1) 만에 판단할 수 있다.

---

## 📊 7. 시간 복잡도 분석

- 최악의 경우: **O(N!)**
- 하지만 백트래킹의 가지치기 덕분에 실제 시간은 훨씬 적게 걸렸다.
- `N=8`일 때 해답 92개를 0.1초도 안 돼서 찾는 걸 보고 신기했다.

---

## 📝 8. 요약

- N-Queen은 **백트래킹**의 아주 좋은 예제였다.
- 핵심은 **가지치기(pruning)** 와 **상태 관리**라는 것을 깨달았다.
- 구현 방법도 여러 가지가 있었다.
  - 기본 재귀
  - `set` 최적화
  - 반복문(스택)

---

## 🚀 9. N-Queen, 그 이상의 문제들

N-Queen 문제를 풀고 나니, 퍼즐을 '응용할 수 있겠다' 라는 걸 알게 됐다.

-   **제약 충족 문제 (CSP, Constraint Satisfaction Problems)**
    -   알고 보니 N-Queen은 '제약 충족 문제'라는 더 큰 개념의 대표적인 예시였다.
    -   정해진 제약 조건(서로 공격 불가)을 만족하는 해를 찾는다는 점에서, 스도쿠 퍼즐이나 강의 시간표 짜기, 자원 할당 문제 등 현실의 많은 문제와 닮아있었다.

-   **VLSI 회로 설계**
    -   놀랍게도 반도체 칩 설계 같은 첨단 분야에서도 비슷한 원리가 사용된다고 한다. 😲
    -   좁은 공간에 부품들을 서로 간섭 없이 효율적으로 배치하는 문제가 N-Queen과 본질적으로 같다는 점이 흥미로웠다.

-   **병렬 컴퓨팅**
    -   N이 아주 커지면 혼자서는 풀기 벅차다.
    -   이런 큰 문제를 여러 컴퓨터가 나누어 푸는 병렬 알고리즘을 테스트하고 최적화하는 데 N-Queen이 좋은 벤치마크가 된다고 한다.

단순한 퍼즐인 줄 알았는데, 생각보다 훨씬 깊이 있고 다양한 분야에 연결되어 있어서 놀랐다. 역시 기본기가 중요하다는 걸 다시 한번 느꼈다.

---

## 📚 10. 참고

- `N=8` 해답 개수: **92개**
- `N=10` 해답 개수: **724개**
- `N=14` 해답 개수: **365,596개**
