---
layout: post
title: "[BOJ] 2206 - 벽 부수고 이동하기"
date: 2025-09-11 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [bfs, graph, shortest-path, implementation]
---

<br>

### 💻 백준 2206번: 벽 부수고 이동하기

- **문제 링크:** [https://www.acmicpc.net/problem/2206](https://www.acmicpc.net/problem/2206)
- **알고리즘 분류:** BFS, 그래프 이론, 최단 경로

## 문제 소개 🧐

N×M 크기의 맵이 주어졌다. 0은 이동할 수 있는 곳, 1은 벽이다. (1, 1)에서 (N, M)까지 최단 경로로 이동해야 한다.

이때, 특별한 규칙이 하나 있다. 이동하는 도중 벽을 **최대 한 개까지** 부수고 이동할 수 있다는 점이다. 시작하는 칸과 끝나는 칸도 경로에 포함해서 센다.

맵이 주어졌을 때, 최단 경로의 길이를 구하는 문제다.

### 입력 📝

첫째 줄에 N(1 ≤ N ≤ 1,000), M(1 ≤ M ≤ 1,000)이 주어진다. 다음 N개의 줄에 M개의 숫자로 맵이 주어진다. (1, 1)과 (N, M)은 항상 0이라고 가정한다.

### 출력 📤

첫째 줄에 최단 거리를 출력한다. 불가능할 때는 -1을 출력한다.

### 제한 ❌

- 시간 제한: 2초
- 메모리 제한: 192MB

---

### 첫 번째 시도: 3차원 방문 배열을 활용한 BFS ✨

#### Idea

최단 경로 문제이므로 BFS를 사용하는 것이 자연스럽다고 생각했다. 하지만 이 문제는 '벽을 한 번 부술 수 있다'는 추가 조건이 있었다.

이 조건을 처리하기 위해 방문 배열을 3차원으로 확장하는 아이디어를 떠올렸다. `visited[row][col][broken_status]`와 같이, `broken_status` (0: 벽 안 부숨, 1: 벽 부숨)를 추가하여 각 상태별 최단 거리를 기록하기로 했다.

큐에 `(행, 열, 벽 파괴 여부)`를 튜플로 저장하여 BFS를 진행했다.

1.  현재 위치에서 상하좌우로 이동할 때, 다음 칸이 맵 범위 내에 있는지 확인한다.
2.  **벽을 부순 적이 없는 경우 (`broken_status` = 0):**
    -   다음 칸이 길(0)이면, 벽을 부수지 않은 상태 그대로 이동한다.
    -   다음 칸이 벽(1)이면, 벽을 부수고 `broken_status`를 1로 바꿔 이동한다.
3.  **벽을 부순 적이 있는 경우 (`broken_status` = 1):**
    -   다음 칸이 길(0)인 경우에만 이동할 수 있다.

각 경우에 대해 방문하지 않았던 상태라면, 거리를 갱신하고 큐에 추가했다.

#### Code

```python
import sys
from collections import deque

N, M = map(int, sys.stdin.readline().split())
board = [list(map(int, sys.stdin.readline().rstrip())) for _ in range(N)]
moves = [(0, 1), (1, 0), (0, -1), (-1, 0)]

def bfs():
    # visited[r][c][0]: 벽 안 부수고 (r,c)까지 오는 최단거리
    # visited[r][c][1]: 벽 한 번 부수고 (r,c)까지 오는 최단거리
    visited = [[[0] * 2 for _ in range(M)] for _ in range(N)]
    q = deque([(0, 0, 0)]) # r, c, broken_status
    visited[0][0][0] = 1

    while q:
        r, c, broken = q.popleft()

        if r == N - 1 and c == M - 1:
            return visited[r][c][broken]

        for dr, dc in moves:
            nr, nc = r + dr, c + dc

            if 0 <= nr < N and 0 <= nc < M:
                # 다음 칸이 길(0)이고, 아직 방문 안 한 경우
                if board[nr][nc] == 0 and visited[nr][nc][broken] == 0:
                    visited[nr][nc][broken] = visited[r][c][broken] + 1
                    q.append((nr, nc, broken))
                # 다음 칸이 벽(1)이고, 아직 벽을 부순 적이 없는 경우
                elif board[nr][nc] == 1 and broken == 0:
                    visited[nr][nc][1] = visited[r][c][0] + 1
                    q.append((nr, nc, 1))
    return -1

print(bfs())

```

#### Result

- **결과: 성공!** 🎉
- 3차원 방문 배열을 활용한 BFS로 상태를 관리하는 것이 핵심이었다. 벽을 부쉈을 때와 부수지 않았을 때의 경로를 독립적으로 탐색함으로써 정답을 찾을 수 있었다.

---

## 배운 점 및 개선할 부분 🤔

- **상태의 확장:** 단순한 2차원 좌표 방문 여부만이 아니라, '벽을 파괴했는가'라는 추가적인 '상태'를 함께 관리해야 하는 문제였다. 이처럼 기존 BFS/DFS에서 상태를 확장하여 더 복잡한 문제를 해결하는 테크닉을 익힐 수 있었다.
- **BFS의 응용:** 가중치가 없는 그래프(또는 모든 가중치가 동일한 그래프)에서 최단 경로를 찾을 때 BFS가 얼마나 강력한지 다시 한번 느꼈다. 이 문제 역시 BFS의 원리를 응용해 해결할 수 있었다.

> 문제의 제약 조건(벽을 한 번 부술 수 있음)을 어떻게 '상태'에 녹여낼지 고민하는 과정이 중요했다.

<div style="text-align: center">⁂</div>
