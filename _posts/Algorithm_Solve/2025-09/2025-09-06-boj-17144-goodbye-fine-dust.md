---
layout: post
title:  "[백준] 17144: 미세먼지 안녕!"
date:   2025-09-06 10:00:00 +0900
categories: Algorithm_Solve
tags: [algorithm, simulation, implementation]
---

<br>

### 💻 백준 17144번: 미세먼지 안녕!

- **문제 링크:** [https://www.acmicpc.net/problem/17144](https://www.acmicpc.net/problem/17144)
- **알고리즘 분류:** 시뮬레이션, 구현

## 문제 소개 🧐

R×C 격자판에 미세먼지와 공기청정기가 있다. 1초 동안 다음 두 가지 일이 순서대로 일어난다.

1.  **미세먼지 확산:**
    - 모든 칸에서 동시에 인접한 네 방향으로 확산된다.
    - 확산되는 양은 `A[r,c] / 5` 이다.
    - 확산 후 원래 칸에 남는 양은 `A[r,c] - (A[r,c] / 5) * (확산된 방향 수)` 이다.
    - 공기청정기가 있거나 칸이 없으면 확산되지 않는다.

2.  **공기청정기 작동:**
    - 위쪽 공기청정기는 반시계방향, 아래쪽은 시계방향으로 바람이 순환한다.
    - 바람의 방향대로 미세먼지가 한 칸씩 이동한다.
    - 공기청정기로 들어간 미세먼지는 정화된다.

T초가 지난 후 방에 남아있는 미세먼지의 총 양을 구하는 문제다.

### 입력 📝

- 첫째 줄: R, C, T (6 ≤ R, C ≤ 50, 1 ≤ T ≤ 1,000)
- 둘째 줄부터 R개 줄: 방의 상태. `-1`은 공기청정기, 나머지는 미세먼지 양.

### 출력 📤

- T초 후 남아있는 미세먼지의 양을 출력한다.

---

### 나의 풀이 ✨

#### Idea

시뮬레이션 문제로, 문제에서 요구하는 두 가지 과정을 T초 동안 반복해야 한다.

1.  **미세먼지 확산 함수:** 확산은 모든 칸에서 '동시에' 일어나므로, 현재 상태를 기준으로 각 칸의 변화량을 별도의 2차원 배열에 기록했다가 모든 계산이 끝난 후 한 번에 원래 배열에 적용해야 한다.
2.  **공기청정기 작동 함수:** 공기청정기 바람의 경로를 따라가며 미세먼지를 한 칸씩 이동시킨다. 위쪽과 아래쪽의 순환 방향이 다르므로 나누어 처리한다.

이 두 함수를 T번 반복 실행한 후, 방에 남은 미세먼지의 총합을 계산한다.

#### Code

```python
import sys

R, C, T = map(int, sys.stdin.readline().split())

room = [list(map(int, sys.stdin.readline().split())) for _ in range(R)]
moves = [(1, 0), (0, 1), (-1, 0), (0, -1)]  # 하 우 상 좌

conditioner = []
for i in range(R):
    if room[i][0] == -1:
        conditioner.append((i,0))

def spread_air(room):
    global R, C
    global conditioner

    spread_map = [[0 for _ in range(C)] for _ in range(R)]

    for i in range(R):
        for j in range(C):
            if room[i][j] > 0:
                d_munji = room[i][j] // 5
                count = 0

                for dr, dc in moves:
                    nr, nc = (i + dr), (j + dc)
                    
                    if (0 <= nr < R) and (0 <= nc < C) and room[nr][nc] != -1:
                        spread_map[nr][nc] += d_munji
                        count += 1
                
                room[i][j] -= d_munji * count
    
    for i in range(R):
        for j in range(C):
            room[i][j] += spread_map[i][j]

def clean_air(room):
    global R, C
    global conditioner

    # 위쪽 공기청정기 (반시계)
    up_r, up_c = conditioner[0]
    # 1. 오른쪽으로
    prev = 0
    for c in range(up_c + 1, C):
        room[up_r][c], prev = prev, room[up_r][c]
    # 2. 위로
    for r in range(up_r - 1, -1, -1):
        room[r][C-1], prev = prev, room[r][C-1]
    # 3. 왼쪽으로
    for c in range(C - 2, -1, -1):
        room[0][c], prev = prev, room[0][c]
    # 4. 아래로
    for r in range(1, up_r):
        room[r][0], prev = prev, room[r][0]

    # 아래쪽 공기청정기 (시계)
    down_r, down_c = conditioner[1]
    # 1. 오른쪽으로
    prev = 0
    for c in range(down_c + 1, C):
        room[down_r][c], prev = prev, room[down_r][c]
    # 2. 아래로
    for r in range(down_r + 1, R):
        room[r][C-1], prev = prev, room[r][C-1]
    # 3. 왼쪽으로
    for c in range(C - 2, -1, -1):
        room[R-1][c], prev = prev, room[R-1][c]
    # 4. 위로
    for r in range(R - 2, down_r, -1):
        room[r][0], prev = prev, room[r][0]


for _ in range(T):
    spread_air(room)
    clean_air(room)

answer = 0
for row in room:
    answer += sum(row)

print(answer + 2)
```

#### Result

- **결과: 성공!** 🎉

---

## 마무리 🤔

- **동시성 처리:** 미세먼지 확산과 같이 여러 요소가 '동시에' 변하는 시뮬레이션에서는, 변화량을 즉시 반영하지 않고 별도의 공간에 기록했다가 한 번에 업데이트하는 것이 핵심이다.
- **구현의 정확성:** 공기청정기 바람의 순환처럼 정해진 경로를 따라 상태를 변경하는 문제는, 경계를 정확히 설정하고 순서대로 차근차근 구현하는 것이 중요하다.

> 문제의 요구사항을 정확히 이해하고 단계별로 나누어 구현하는 능력이 중요한 시뮬레이션 문제였다. 특히 확산 로직과 순환 로직을 분리하여 생각한 것이 도움이 되었다.

<div style="text-align: center">⁂</div>
