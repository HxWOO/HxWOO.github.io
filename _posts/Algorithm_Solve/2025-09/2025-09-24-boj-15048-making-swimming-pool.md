---
layout: post
title: "[BOJ] 15048 - 수영장 만들기"
date: 2025-09-24 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [bfs, dfs, simulation, implementation]
---

<br>

### 💻 문제: 수영장 만들기

- **문제 링크:** [https://www.acmicpc.net/problem/15048](https://www.acmicpc.net/problem/15048)
- **알고리즘 분류:** 너비 우선 탐색(BFS), 깊이 우선 탐색(DFS), 시뮬레이션(Simulation), 구현(Implementation)

## 문제 소개 🧐

N x M 크기의 땅에 각기 다른 높이의 직육면체가 있을 때, 물이 밖으로 새지 않도록 가둘 수 있는 물의 총량을 계산하는 문제다. 물은 항상 높이가 낮은 곳으로만 흐르며, 가장자리에 닿으면 밖으로 빠져나간다.

| 시간 제한 | 메모리 제한 | 제출 | 정답 | 맞힌 사람 | 정답 비율 |
| --- | --- | --- | --- | --- | --- |
| 2 초 | 128 MB | 6340 | 2662 | 1795 | 40.058% |

### 입력 📝

- 첫째 줄에 땅의 세로 크기 N과 가로 크기 M이 주어진다. (N, M ≤ 50)
- 둘째 줄부터 N개의 줄에 걸쳐 각 칸의 높이가 1에서 9 사이의 숫자로 주어진다.

### 출력 📤

- 수영장에 채울 수 있는 물의 최대 양을 출력한다.

---

### 첫 번째 시도: 영역 탐색 후 벽 높이 계산 😭

#### Idea

1.  아직 물이 채워지지 않은 내부 칸(`1`부터 `N-2`, `1`부터 `M-2`)에서 시작한다.
2.  DFS를 통해 시작 칸보다 높이가 낮은 인접 칸들을 탐색하여 하나의 '웅덩이'가 될 수 있는 영역(`water_set`)을 찾는다.
3.  만약 탐색 중 가장자리에 닿으면 물이 새는 영역이므로 `None`을 반환한다.
4.  가장자리에 닿지 않고 영역이 확정되면, 그 영역을 둘러싼 '벽'들 중 가장 낮은 높이(`min_wall`)를 찾는다.
5.  웅덩이 영역의 각 칸에 대해 `(min_wall - 현재 칸 높이)`를 계산하여 총량을 더한다.

#### Code

```python
import sys

N, M = map(int, sys.stdin.readline().split())

lands = [list(map(int, list(sys.stdin.readline().rstrip()))) for _ in range(N)]
moves = [(1, 0), (0, 1), (-1, 0), (0, -1)]  # 하 우 상 좌

def dfs(start :tuple):
    """
    start 랑 같은 영역을 공유하는 애들을 구함
    dfs인데 visited 조건 + 현재 높이랑 비교
    0, N-1 행 / 0, M-1 행 닿으면 물 못 채움
    """
    global N, M
    global lands

    stack = []
    visited = [[False for _ in range(M)] for _ in range(N)]

    stack.append(start)
    output = set()
    start_height = lands[start[0]][start[1]]
    
    while stack:
        r, c = stack.pop()
        output.add((r,c))

        for dr, dc in moves:
            nr = r + dr
            nc = c + dc
            if 0 <= nr < N and 0 <= nc < M and not visited[nr][nc] and lands[nr][nc] <= start_height:
                # 0, N-1 행 / 0, M-1 행 닿으면 물 못 채움
                if nr == 0 or nr == (N-1) or nc == 0 or nc == (M-1):
                    return None
                stack.append((nr,nc))
                output.add((nr, nc))
                visited[nr][nc] = True

    return output

def get_wall_height(water_area, is_water):
    """
    특정 영역을 감싸는 지형중 가장 낮은 높이를 구하는 함수
    """
    global N, M
    global lands
    
    for r, c in water_area:
        is_water[r][c] = True
    
    min_wall = float('inf')
    for r,c in water_area:
        for dr, dc in moves:
            nr = r + dr
            nc = c + dc
            if 0 <= nr < N and 0 <= nc < M and not is_water[nr][nc]:
                min_wall = min(lands[nr][nc], min_wall)
    
    return min_wall

is_water = [[False for _ in range(M)] for _ in range(N)]
answer = 0
for i in range(1, N-1):
    for j in range(1, M-1):
        # 수영장 영역 구함
        water_set = None
        if not is_water[i][j]:
            water_set = dfs((i,j))

        # 수영장 영역이 있다면
        if water_set:
            # 둘러싼 벽중 가장 낮은 벽의 높이를 구해서
            wall_height = get_wall_height(water_set, is_water)
            
            # 수영장 영역 지형 높이를 뺀걸 더함
            answer += sum(list(map(lambda x: wall_height - lands[x[0]][x[1]], water_set)))

print(answer)
```

#### Result

- **결과: 실패** 😭
- **문제점:** 이 로직은 한 번에 하나의 웅덩이를 완벽하게 식별하지 못한다. 예를 들어, 더 낮은 웅덩이가 더 높은 웅덩이에 포함될 수 있는데, 이를 따로 계산하면서 중복 계산이 발생하거나 잘못된 벽 높이를 참조하는 문제가 생긴다.
- **반례:**
  ```
  4 5
  99999
  97679
  99899
  99999
  ```
  위 예시에서 `(1, 1)`의 높이 7에서 시작하면 `(1, 2)`의 6을 포함하는 웅덩이를 계산하고, 나중에 `(2, 2)`의 8에서 시작하면 또 다른 웅덩이를 계산하게 되어 로직이 꼬인다.

---

### 두 번째 시도: 높이 기반 시뮬레이션 ✨

#### Idea

땅의 높이가 1~9로 매우 낮다는 점에 착안했다. 물이 1부터 9까지 차례로 차오른다고 생각하고 시뮬레이션하는 접근법이다.

1.  물의 높이(`lv`)를 1부터 8까지 1씩 증가시키며 반복한다.
2.  각 높이 `lv`에서, 현재 땅의 높이가 `lv`인 모든 칸을 찾는다.
3.  해당 칸에서부터 DFS/BFS를 시작하여, 같은 높이(`lv`)를 가진 인접한 영역을 모두 찾는다.
4.  탐색 중 영역이 가장자리에 닿으면, 그 영역은 물이 빠져나가는 곳이므로 물을 채울 수 없다 (`is_absorbed = True`).
5.  탐색이 끝난 후,
    -   `is_absorbed`가 `True`이면 물이 새므로 아무것도 하지 않는다.
    -   `is_absorbed`가 `False`이면, 해당 영역은 물을 가둘 수 있으므로 영역의 칸 수만큼(`len(area)`) 정답에 더한다.
6.  탐색했던 영역의 모든 칸의 높이를 1 증가시킨다. (`lands[r][c] += 1`) 이는 다음 높이의 물을 채울 때 이 칸들이 바닥 역할을 하도록 만들기 위함이다.

#### Code

```python
import sys

N, M = map(int, sys.stdin.readline().split())

lands = [list(map(int, list(sys.stdin.readline().rstrip()))) for _ in range(N)]
moves = [(1, 0), (0, 1), (-1, 0), (0, -1)]  # 하 우 상 좌

def dfs(start :tuple, level, visited):
    """
    start 랑 같은 영역인 애들을 구함
    높이도 같음
    0, N-1 행 / 0, M-1 행 닿으면 물 못 채움
    """
    global N, M
    global lands

    stack = []
    stack.append(start)
    # 물이 빠져나가는지 확인하는 flag
    is_absorbed = False

    # 시작부터 가장자리일수 있으니깐 확인
    if start[0] == 0 or start[0] == N-1 or start[1] == 0 or start[1] == M-1:
        is_absorbed = True
    visited[start[0]][start[1]] = True
    area = []
    
    while stack:
        r, c = stack.pop()
        area.append((r, c))

        for dr, dc in moves:
            nr = r + dr
            nc = c + dc
            if 0 <= nr < N and 0 <= nc < M and not visited[nr][nc] and lands[nr][nc] == level:
                if nr == 0 or nr == N-1 or nc == 0 or nc == M-1:
                    is_absorbed = True
                stack.append((nr,nc))
                visited[nr][nc] = True

    return is_absorbed, area

answer = 0
for lv in range(1, 9): # 높이가 1일때부터 확인
    visited = [[False for _ in range(M)] for _ in range(N)]
    for i in range(N):
        for j in range(M):
            if lands[i][j] == lv and not visited[i][j]:
                is_absorbed, area = dfs((i,j), lv, visited)
                for r, c in area: # 일단 방문한 곳의 높이를 채워주고
                    lands[r][c] += 1
                if not is_absorbed:  # 만약 물을 채울 수 있는 영역이면 그만큼 답에 더함
                    answer += len(area)
    del visited  # 메모리 관리 차원에서 delete

print(answer)
```

#### Result

- **결과: 성공!** 🎉
- 이 접근법은 각 높이 레벨에서 물이 채워질 수 있는지를 독립적으로 판단하므로, 첫 번째 아이디어의 복잡한 '웅덩이' 식별 문제를 피할 수 있다.

---

## 배운 점 및 개선할 부분 🤔

-   **관점의 전환:** '웅덩이를 찾고 벽을 계산'하는 복잡한 방식에서 '물을 한 층씩 채워보는' 단순한 시뮬레이션으로 관점을 바꾸니 문제가 쉽게 풀렸다. 문제의 제약 조건(특히 높이 범위)을 활용하는 것이 중요하다.
-   **시뮬레이션의 힘:** 복잡한 상호작용이 있는 문제에서, 상태를 단계별로 변화시키는 시뮬레이션은 논리를 명확하게 하고 실수를 줄이는 강력한 도구가 될 수 있다.
-   **상태 업데이트의 중요성:** 두 번째 풀이에서 `lands[r][c] += 1` 부분이 핵심이다. 물이 채워진 칸은 더 이상 바닥이 아니므로 높이를 1 올려 다음 레벨의 시뮬레이션에 영향을 주도록 상태를 업데이트해야 한다. 이 부분이 없다면 같은 칸에 물이 중복으로 채워지는 오류가 발생한다.

> 복잡한 지형 문제도 때로는 가장 낮은 곳부터 차근차근 채워나가면 해결의 실마리가 보인다.

<div style="text-align: center">⁂</div>
