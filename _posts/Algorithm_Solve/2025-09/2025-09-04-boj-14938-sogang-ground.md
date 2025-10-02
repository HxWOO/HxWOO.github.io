---
layout: post
title:  "[BOJ] 14938번: 서강그라운드"
date:   2025-09-04 10:00:00 +0900
categories: [Algorithm_Solve]
tags: [PS, BOJ, Floyd-Warshall, Shortest Path]
---

<br>

### 💻 백준 14938번: 서강그라운드

- **문제 링크:** [https://www.acmicpc.net/problem/14938](https://www.acmicpc.net/problem/14938)
- **알고리즘 분류:** 플로이드-워셜, 최단 경로

## 문제 소개 🧐

예은이는 요즘 가장 인기가 있는 게임 서강그라운드를 즐기고 있다. 서강그라운드는 여러 지역중 하나의 지역에 낙하산을 타고 낙하하여, 그 지역에 떨어져 있는 아이템들을 이용해 서바이벌을 하는 게임이다.

각 지역은 일정한 길이의 길로 다른 지역과 연결되어 있고 이 길은 양방향 통행이 가능하다. 예은이는 낙하한 지역을 중심으로 수색 범위 이내의 모든 지역의 아이템을 습득 가능하다고 할 때, 예은이가 얻을 수 있는 아이템의 최대 개수를 구하는 문제다.

### 입력 📝

- 첫째 줄에는 지역의 개수 n (1 ≤ n ≤ 100)과 예은이의 수색범위 m (1 ≤ m ≤ 15), 길의 개수 r (1 ≤ r ≤ 100)이 주어진다.
- 둘째 줄에는 n개의 숫자가 차례대로 각 구역에 있는 아이템의 수 t (1 ≤ t ≤ 30)를 알려준다.
- 세 번째 줄부터 r+2번째 줄 까지 길 양 끝에 존재하는 지역의 번호 a, b, 그리고 길의 길이 l (1 ≤ l ≤ 15)가 주어진다.

### 출력 📤

- 예은이가 얻을 수 있는 최대 아이템 개수를 출력한다.

### 제한 ❌

- 시간 제한: 1초
- 메모리 제한: 128MB

---

### 나의 풀이 ✨

#### Idea
어느 지역에 떨어져야 가장 많은 아이템을 얻을 수 있는지 찾아야 하는 문제다. 이를 위해서는 각 지역에 떨어졌을 때를 모두 시뮬레이션 해보고, 그 중 가장 많은 아이템을 얻는 경우를 찾아야 한다.

특정 지역에 떨어졌을 때 얻을 수 있는 아이템의 총합을 구하려면, 해당 지역으로부터 다른 모든 지역까지의 최단 거리를 알아야 한다. 그 후, 최단 거리가 수색 범위 `m` 이내인 지역들의 아이템을 모두 더하면 된다.

모든 지역 쌍 간의 최단 거리를 구해야 하므로, **플로이드-워셜(Floyd-Warshall)** 알고리즘을 사용하는 것이 가장 직관적이고 효과적이라고 판단했다.

**풀이 단계:**
1.  `n x n` 크기의 2차원 배열 `graph`를 생성하고, 모든 값을 무한(infinity)으로 초기화한다. 자기 자신으로 가는 경로는 0으로 설정한다.
2.  입력으로 주어지는 길 정보를 `graph`에 반영한다. 양방향 통행이 가능하므로 `graph[a][b]`와 `graph[b][a]`에 모두 길의 길이 `l`을 저장한다.
3.  플로이드-워셜 알고리즘을 실행하여 모든 지역 쌍 간의 최단 거리를 계산한다.
4.  모든 지역을 순회하며 각 지역을 시작점으로 가정한다.
5.  각 시작점 `i`에 대해, `i`로부터의 최단 거리가 수색 범위 `m` 이하인 모든 지역 `j`를 찾아 해당 지역의 아이템 수를 모두 더한다.
6.  각 시작점마다 계산된 아이템의 총합 중 최댓값을 찾아 최종 결과로 출력한다.

#### Code

```python
import sys

n, m, r = map(int, sys.stdin.readline().split())
t = list(map(int, sys.stdin.readline().split()))

graph = [[float('inf') for _ in range(n)] for _ in range(n)]

for i in range(r):
    a, b, l = map(int, sys.stdin.readline().split())
    graph[a-1][b-1] = min(graph[a-1][b-1], l)
    graph[b-1][a-1] = min(graph[b-1][a-1], l)

for i in range(n):
    graph[i][i] = 0

def floyd_warshall(n, graph):
    dist = [row[:] for row in graph]

    for k in range(n):
        for i in range(n):
            for j in range(n):
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
    return dist

shortest_path = floyd_warshall(n, graph)
answer = 0
for path in shortest_path:
    itm_cnt = 0
    for i in range(n):
        if path[i] <= m:
            itm_cnt += t[i]
    answer = max(answer, itm_cnt)

print(answer)
```

#### Result

- **결과: 성공!** 🎉

---

## 마무리 🤔

- **알고리즘 선택:** 모든 지점에서 다른 모든 지점으로의 최단 거리를 구해야 하는 문제 상황에서 플로이드-워셜 알고리즘을 떠올리는 것은 자연스러운 흐름이었다. 지역의 개수 `n`이 100으로 크지 않았기 때문에 O(n³)의 시간 복잡도를 가진 플로이드-워셜로도 충분히 시간 내에 해결할 수 있었다.
- **다익스트라 vs 플로이드-워셜:** 만약 `n`이 더 컸다면, 모든 노드를 시작점으로 하여 다익스트라 알고리즘을 `n`번 실행하는 방식(O(N * E log N))도 고려해볼 수 있었을 것이다. 하지만 이 문제에서는 플로이드-워셜이 구현이 더 간편하고 효과적이었다.

> 모든 쌍 최단 경로(All-Pairs Shortest Path) 문제의 전형적인 풀이법을 연습할 수 있는 좋은 문제였다.

<div style="text-align: center">⁂</div>
