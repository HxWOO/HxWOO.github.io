---
layout: post
title: "[BOJ] 11779 - 최소비용 구하기 2"
date: 2025-09-16 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [dijkstra, shortest-path, graph-traversal]
---

<br>

### 💻 백준 11779번: 최소비용 구하기 2

- **문제 링크:** [https://www.acmicpc.net/problem/11779](https://www.acmicpc.net/problem/11779)
- **알고리즘 분류:** 다익스트라, 그래프 이론

## 문제 소개 🧐

n개의 도시와 m개의 버스가 주어질 때, 특정 도시 A에서 B까지 가는 데 드는 최소 비용과 그 경로를 함께 출력하는 문제다. 일반적인 최단 경로 문제와 달리, 경로 자체도 복원해야 한다는 점이 특징이다.

### 입력 📝

- 첫째 줄: 도시의 개수 n (1 ≤ n ≤ 1,000)
- 둘째 줄: 버스의 개수 m (1 ≤ m ≤ 100,000)
- 셋째 줄부터 m개의 줄: 버스 정보 (출발 도시, 도착 도시, 비용)
- 마지막 줄: 구하고자 하는 출발 도시와 도착 도시

### 출력 📤

- 첫째 줄: 최소 비용
- 둘째 줄: 경로에 포함된 도시의 개수
- 셋째 줄: 최소 비용 경로 (출발 도시부터 순서대로)

---

### 풀이: 다익스트라와 경로 역추적 🗺️

#### Idea

가중치가 있는 그래프에서 한 지점에서 다른 지점까지의 최단 경로를 구하는 문제이므로 다익스트라(Dijkstra) 알고리즘을 사용하는 것이 자연스럽다.

다만, 이 문제는 최소 비용뿐만 아니라 경로까지 출력해야 한다. 따라서 다익스트라 알고리즘을 실행하면서 최단 경로를 구성하는 직전 노드들을 별도의 배열에 기록해두어야 한다.

1.  **최단 거리 계산:** 다익스트라 알고리즘으로 출발점에서 모든 점까지의 최단 거리를 계산한다.
2.  **경로 복원을 위한 배열:** `prev_path`와 같은 배열을 만들어, 최단 거리가 갱신될 때마다 `prev_path[현재_노드] = 직전_노드` 형태로 경로를 저장한다.
3.  **경로 역추적:** 알고리즘 종료 후, 도착점에서부터 `prev_path`를 따라가며 출발점이 나올 때까지 경로를 역으로 추적한다. 이 결과를 `deque`의 `appendleft`를 이용해 저장하면 올바른 순서의 경로를 쉽게 얻을 수 있다.

#### Code

```python
import sys
import heapq
from collections import deque

n = int(sys.stdin.readline().rstrip())
m = int(sys.stdin.readline().rstrip())

graph = [[] for _ in range(n+1)]

for _ in range(m):
    v, w, c = map(int, sys.stdin.readline().split())
    graph[v].append((c, w))
    
def dijstra(start, end, board):
    global n

    q = []
    q.append((0, start, 0))

    shortest_d = [float('inf')] * (n+1)
    shortest_d[start] = 0
    prev_path = [float('inf')] * (n+1)
    prev_path[start] = 0

    while q:
        cost, v, prev_v = heapq.heappop(q)

        if cost > shortest_d[v]:
            continue
        if v == end:
            break

        for dc, nv in board[v]:
            nc = cost + dc

            if nc < shortest_d[nv]:
                shortest_d[nv] = nc
                prev_path[nv] = v
                heapq.heappush(q, (nc, nv, v))

    return prev_path, shortest_d

a, b = map(int, sys.stdin.readline().split())
shortest_path, s_d_li = dijstra(a, b, graph)

print(s_d_li[b])

answer_path = deque()
while b != a:
    answer_path.appendleft(b)
    b = shortest_path[b]
answer_path.appendleft(a)

print(len(answer_path))
print(*answer_path)
```

#### Result

- **결과: 성공!** 🎉
- 다익스트라 알고리즘의 기본 구조에 직전 노드를 저장하는 배열 하나만 추가함으로써 경로 복원 문제까지 해결할 수 있었다.

---

## 배운 점 및 개선할 부분 🤔

- **다익스트라와 경로 복원:** 최단 경로 문제에서 실제 경로를 출력해야 할 경우, 거리 갱신 시 직전 노드를 함께 저장하는 것이 핵심이다. `dist[next] = dist[curr] + cost`가 일어날 때, `parent[next] = curr`와 같이 기록하면 된다.
- **경로 역추적 방법:** 도착지부터 시작해 부모 노드를 계속 따라가면 출발지까지의 경로가 역순으로 나온다. 이를 스택에 넣고 빼거나, `deque`의 `appendleft`를 사용하면 정방향 경로를 쉽게 만들 수 있다.
- **다익스트라의 종료 조건:** 우선순위 큐를 사용하는 다익스트라에서는 큐에서 뽑은 노드가 도착지라고 해서 바로 종료하면 안 되는 경우가 있다. 하지만 이 문제처럼 가중치가 모두 양수이고, 우선순위 큐가 항상 가장 비용이 적은 노드를 먼저 꺼내는 특성상, 도착지를 처음 방문했을 때의 비용이 곧 최단 비용임이 보장되므로 `if v == end: break` 최적화가 가능하다.

> 최단 경로를 구하는 것과 그 경로를 '보여주는' 것은 다른 문제다. 경로 복원을 위한 추가적인 데이터 저장을 잊지 말자.

<div style="text-align: center">⁂</div>
