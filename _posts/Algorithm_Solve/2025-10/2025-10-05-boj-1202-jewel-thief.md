---
layout: post
title: "[BOJ] 1202 - 보석 도둑"
date: 2025-10-05 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [greedy, priority_queue, sorting]
---

<br>

### 💻 문제: 보석 도둑

- **문제 링크:** [https://www.acmicpc.net/problem/1202](https://www.acmicpc.net/problem/1202)
- **알고리즘 분류:** 그리디(Greedy), 우선순위 큐(Priority Queue), 정렬(Sorting)

## 문제 소개 🧐

N개의 보석 각각의 무게와 가격, 그리고 K개의 가방 각각의 최대 수용 무게가 주어졌을 때, 훔칠 수 있는 보석의 최대 가격을 구하는 문제다. 각 가방에는 최대 하나의 보석만 담을 수 있다.

| 시간 제한 | 메모리 제한 | 제출 | 정답 | 맞힌 사람 | 정답 비율 |
| --- | --- | --- | --- | --- | --- |
| 1 초 | 256 MB | 96140 | 24430 | 16934 | 23.546% |

### 입력 📝

- 첫째 줄에 N과 K가 주어진다. (1 ≤ N, K ≤ 300,000)
- 다음 N개 줄에는 각 보석의 정보 Mi와 Vi가 주어진다. (0 ≤ Mi, Vi ≤ 1,000,000)
- 다음 K개 줄에는 가방에 담을 수 있는 최대 무게 Ci가 주어진다. (1 ≤ Ci ≤ 100,000,000)

### 출력 📤

- 첫째 줄에 상덕이가 훔칠 수 있는 보석 가격의 합의 최댓값을 출력한다.

---

### Idea 1: 가격 기준 내림차순 정렬 후 탐색

#### Idea

가장 비싼 보석부터 가방에 담을 수 있는지 확인하는 그리디 접근.

1.  보석을 가격(`v`) 기준으로 내림차순 정렬한다.
2.  가방을 무게(`c`) 기준으로 내림차순 정렬한다.
3.  가장 비싼 보석부터 순회하며, 담을 수 있는 가방 중 가장 무게 제약이 비슷한 (가장 작은) 가방에 넣는다.

#### Code (1차 시도)

```python
import sys

N, K = map(int, sys.stdin.readline().split())

v_w_list = []
for _ in range(N):
    w, v = map(int, sys.stdin.readline().split())
    v_w_list.append((v, w))

bag_w_list = []
for _ in range(K):
    bag_w_list.append(int(sys.stdin.readline().rstrip()))

v_w_list.sort(reverse=True)
bag_w_list.sort(reverse=True)

# 가치가 높은 보석부터 담을 수 있는지 판단
# 최대한 가용량이랑 무게랑 딱 맞는게 이득이니깐 계속 돌긴해야하는데...
answer = 0
for jew in v_w_list:
    idx = 0
    for cap in bag_w_list:
        if cap >= jew[1]:
            break
        idx += 1
    
    if idx < len(bag_w_list):
        answer += jew[0]
        bag_w_list.pop(idx)

print(answer)
```

#### Result

-   **결과: 시간 초과** 😭
-   **문제점:** 보석마다 담을 수 있는 가방을 찾는 과정(`for cap in bag_w_list`)과 `pop(idx)` 연산이 비효율적이다. 최악의 경우 시간 복잡도가 O(N*K)에 가까워 시간 초과가 발생한다.

---

### Idea 2: 이분 탐색으로 가방 찾기

#### Idea

1차 시도의 문제점인 가방 탐색 과정을 이분 탐색으로 최적화하려는 시도.

1.  보석을 가격(`v`) 기준으로 내림차순 정렬한다.
2.  가방을 무게(`c`) 기준으로 오름차순 정렬한다.
3.  각 보석에 대해, 담을 수 있는 가장 작은 가방을 이분 탐색으로 찾는다.

#### Code (2차 시도)

```python
import sys

N, K = map(int, sys.stdin.readline().split())

v_w_list = []
for _ in range(N):
    w, v = map(int, sys.stdin.readline().split())
    v_w_list.append((v, w))

bag_w_list = []
for _ in range(K):
    bag_w_list.append(int(sys.stdin.readline().rstrip()))

v_w_list.sort(reverse=True)
bag_w_list.sort()

# 가치가 높은 보석부터 담을 수 있는지 판단
answer = 0
for jew in v_w_list:
    answer_idx = -1
    if bag_w_list:
        start_idx = 0
        end_idx = len(bag_w_list)
        # 이분 탐색으로 최적의 가방 구하기
        while start_idx < end_idx:
            mid_idx = (start_idx + end_idx) // 2
            if bag_w_list[mid_idx] < jew[1]:
                start_idx = mid_idx + 1
            else:
                end_idx = mid_idx
            answer_idx = max(start_idx, end_idx)
    
    # bag_w_list가 비어있지 않아서 이분탐색을 돌았고, 배낭에 수용가능할때
    if answer_idx != -1 and answer_idx < len(bag_w_list):
        bag_w_list.pop(answer_idx)
        answer += jew[0]

print(answer)
```

#### Result

-   **결과: 시간 초과** 😭
-   **문제점:** 이분 탐색으로 가방을 찾는 시간은 O(log K)로 줄였지만, 찾은 후 `bag_w_list.pop(answer_idx)` 연산이 여전히 O(K)의 시간 복잡도를 가진다. 따라서 전체 시간 복잡도는 O(N*K)로 개선되지 않았다.

---

### Idea 3: 우선순위 큐 활용 ✨

#### Idea

탐색의 관점을 바꾼다. '보석'을 기준으로 가방을 찾는 대신, '가방'을 기준으로 담을 수 있는 보석을 고른다.

1.  보석을 무게(`w`) 기준으로 오름차순 정렬한다.
2.  가방을 수용량(`c`) 기준으로 오름차순 정렬한다.
3.  가벼운 가방부터 순회하며 다음을 반복한다:
    a. 현재 가방의 수용량보다 작거나 같은 무게를 가진 보석들을 모두 우선순위 큐(최대 힙)에 넣는다. (우선순위 큐는 가격을 기준으로 한다)
    b. 우선순위 큐가 비어있지 않다면, 현재까지 담을 수 있는 보석들 중 가장 비싼 보석(`heapq.heappop`)을 꺼내 가격을 더한다.
4.  이 과정을 모든 가방에 대해 반복한다.

이 방식은 각 보석과 가방을 한 번씩만 확인하므로 효율적이다.

#### Code (3차 시도)

```python
import sys
import heapq

N, K = map(int, sys.stdin.readline().split())

w_v_list = []
for _ in range(N):
    w, v = map(int, sys.stdin.readline().split())
    heapq.heappush(w_v_list, (w,v))

bag_w_list = []
for _ in range(K):
    bag_w_list.append(int(sys.stdin.readline().rstrip()))

bag_w_list.sort()

answer = 0
candidate_jew = []
for cap in bag_w_list:
    # 담을 수 있는 보석을 모두 후보에 추가해놓음
    while w_v_list and w_v_list[0][0] <= cap:
        w, v = heapq.heappop(w_v_list)
        heapq.heappush(candidate_jew, -1*v)
    
    # 후보 중 젤 비싼걸 가져감
    if candidate_jew:
        answer += (-1*heapq.heappop(candidate_jew))

print(answer)
```

#### Result

-   **결과: 성공!** 🎉
-   가방을 기준으로, 담을 수 있는 보석들을 후보군으로 만들고 그중 가장 비싼 것을 선택하는 그리디 전략이 유효했다. 우선순위 큐를 통해 '후보 중 가장 비싼 것'을 효율적으로 찾을 수 있었다.

---

## 배운 점 및 개선할 부분 🤔

-   **관점의 전환:** '비싼 보석을 어느 가방에 넣을까?'에서 '가벼운 가방부터 채워나가는데, 어떤 보석을 넣어야 가장 이득일까?'로 관점을 바꾸는 것이 문제 해결의 핵심이었다.
-   **그리디와 우선순위 큐의 조합:** 정렬된 순서로 순회하면서, 특정 시점까지의 후보군 중에서 최적의 값을 선택해야 할 때 우선순위 큐는 매우 강력한 도구이다. 이 문제에서는 '현재 가방에 담을 수 있는 모든 보석'이 후보군이 되고, '가장 비싼 보석'이 최적의 값이 된다.
-   **자료구조의 중요성:** 동일한 로직이라도 어떤 자료구조를 사용하느냐에 따라 시간 복잡도가 크게 달라진다. `list.pop(i)`이 O(N)이라는 점을 간과하여 두 번의 실패를 겪었다. 문제의 제약 조건(N, K ≤ 300,000)을 보고 O(N log N) 또는 O(K log K) 정도의 풀이를 설계해야 함을 예측하는 것이 중요하다.

> 막혔을 땐, 문제에 접근하는 관점 자체를 뒤집어보자. 가방을 기준으로 생각한 것이 신의 한 수였다.

<div style="text-align: center">⁂</div>
