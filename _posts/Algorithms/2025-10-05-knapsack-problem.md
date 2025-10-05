---
layout: post
title:  "[Algorithm] 냅색(Knapsack) 문제: DP와 그리디의 대표 주자"
date:   2025-10-05 00:00:00 +0900
categories: Algorithms
tags: [knapsack, algorithm, dynamic_programming, greedy, python]
---

# 🎒 냅색(Knapsack) 문제: DP와 그리디의 대표 주자

---

## 🤔 1. 냅색(Knapsack) 문제란?

**냅색(Knapsack) 문제**는 제한된 자원으로 최대의 가치를 얻는 방법을 찾는 대표적인 **조합 최적화(Combinatorial Optimization)** 문제다.

상황은 간단하다.

> 당신은 도둑이다. 용량이 정해진 배낭(knapsack)을 메고 보석상에 들어왔다. 눈앞에는 각각 다른 무게와 가치를 지닌 보석들이 있다. 배낭이 터지지 않게 하면서, 가장 비싼 값어치의 보석들을 훔쳐 나오려면 어떻게 해야 할까?

이 문제는 제약 조건에 따라 크게 두 가지로 나뉜다.

---

## ⚙️ 2. 0/1 냅색 문제 (0/1 Knapsack Problem)

가장 고전적인 형태의 냅색 문제다.

-   **제약 조건:** 각 보석을 **통째로 가져가거나(1), 아예 가져가지 않거나(0)** 둘 중 하나만 선택해야 한다. 보석을 쪼갤 수 없다.

### 접근법: 동적 계획법 (Dynamic Programming)

이 문제는 작은 문제의 해를 조합해 큰 문제의 해를 찾는 **동적 계획법(Dynamic Programming)** 으로 해결하는 것이 정석이다. 이는 모든 정점 쌍의 최단 경로를 구하는 **[플로이드-워셜 알고리즘](https://hxwoo.github.io/algorithms/2025/09/04/floyd-warshall-all-pairs-shortest-path.html)** 과 같은 원리를 공유한다.

-   **핵심 아이디어:** `dp[i][w]`를 "i번째 보석까지 고려했고, 현재 배낭의 용량이 w일 때의 최대 가치"라고 정의한다.
-   **점화식:**
    -   `i`번째 보석을 담지 않는 경우: `dp[i-1][w]`
    -   `i`번째 보석을 담는 경우: `dp[i-1][w - weight[i]] + value[i]`
    -   `dp[i][w] = max(위 두 경우)`

### 💻 파이썬 코드 예시 (0/1 Knapsack)

```python
def knapsack_01(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        weight = weights[i-1]
        value = values[i-1]
        for w in range(1, capacity + 1):
            if weight <= w:
                # 현재 보석을 담을 수 있다면,
                # (안 담는 경우) vs (담는 경우) 중 더 가치 있는 쪽을 선택
                dp[i][w] = max(dp[i-1][w], dp[i-1][w - weight] + value)
            else:
                # 못 담으면 이전 상태 그대로
                dp[i][w] = dp[i-1][w]
    return dp[n][capacity]

# 예시
weights = [2, 3, 4, 5]
values = [3, 4, 5, 6]
capacity = 5
print(f"0/1 Knapsack 최대 가치: {knapsack_01(weights, values, capacity)}") # 7
```

---

## 💰 3. 분할 가능 냅색 문제 (Fractional Knapsack Problem)

0/1 냅색 문제보다 제약이 완화된 버전이다.

-   **제약 조건:** 보석을 **가루처럼 쪼개서** 일부만 가져갈 수 있다.

### 접근법: 탐욕 알고리즘 (Greedy Algorithm)

이 문제는 매 순간 가장 최선의 선택을 하는 **탐욕 알고리즘(Greedy Algorithm)** 으로 최적해를 찾을 수 있다. 이는 최단 경로를 찾을 때 항상 가장 가까운 정점부터 방문하는 **[다익스트라 알고리즘](https://hxwoo.github.io/algorithms/2025/08/12/dijkstra-algorithm-shortest-path.html)** 과 유사한 접근 방식이다.

-   **핵심 아이디어:** 모든 보석에 대해 **'무게 대비 가치' (가성비)** 를 계산한다.
-   **전략:**
    1.  가성비가 가장 높은 보석부터 배낭에 담는다.
    2.  만약 보석을 통째로 담을 수 없다면, 배낭의 남은 용량만큼 잘라서 담는다.
    3.  이 과정을 위해 보석들을 가성비 순으로 **[정렬](https://hxwoo.github.io/algorithms/2025/08/04/python-sort-and-timsort.html)** 하는 과정이 필수적이다.

### 💻 파이썬 코드 예시 (Fractional Knapsack)

```python
def fractional_knapsack(weights, values, capacity):
    n = len(weights)
    # (가성비, 가치, 무게) 튜플 리스트 만들기
    items = []
    for i in range(n):
        items.append((values[i] / weights[i], values[i], weights[i]))
    
    # 가성비 높은 순으로 내림차순 정렬
    items.sort(reverse=True)
    
    total_value = 0
    
    for ratio, value, weight in items:
        if capacity >= weight:
            # 보석을 통째로 담을 수 있는 경우
            capacity -= weight
            total_value += value
        else:
            # 쪼개서 담아야 하는 경우
            total_value += capacity * ratio
            break # 배낭이 꽉 찼으므로 종료
            
    return total_value

# 예시
weights = [10, 20, 30]
values = [60, 100, 120]
capacity = 50
print(f"Fractional Knapsack 최대 가치: {fractional_knapsack(weights, values, capacity)}") # 240.0
```

---

## 📊 4. 비교 요약

| 구분 | 0/1 냅색 문제 | 분할 가능 냅색 문제 |
| :--- | :--- | :--- |
| **아이템 분할** | ❌ 불가능 | ✅ 가능 |
| **주요 해법** | 동적 계획법 (DP) | 탐욕 알고리즘 (Greedy) |
| **시간 복잡도** | `O(N * W)` | `O(N log N)` (정렬) |

(N: 아이템 개수, W: 배낭 용량)

---


## 🧩 5. 심화: 냅색 문제의 다른 변형들

0/1, 분할 가능 냅색 외에도 다양한 변형 문제들이 존재한다.

### 1. 무제한 냅색 문제 (Unbounded Knapsack Problem)
- **규칙:** 모든 아이템을 **원하는 만큼 여러 번** 담을 수 있다. (각 종류별로 재고가 무한대)
- **해법:** 0/1 냅색과 마찬가지로 DP를 사용하지만, 점화식이 약간 다르다. 1차원 DP 배열로 최적화하여 풀 수 있다.
  - `dp[w] = max(dp[w], dp[w - weight[i]] + value[i])`

### 2. 다차원 냅색 문제 (Multidimensional Knapsack Problem)
- **규칙:** 배낭의 제약 조건이 무게 하나가 아니라 **무게, 부피 등 여러 개**다.
- **해법:** DP 상태에 차원이 추가된다. `dp[w1][w2]...` 와 같이 상태 공간이 커져 훨씬 복잡한 문제가 된다. (NP-Hard)

### 3. 다중 냅색 문제 (Multiple Knapsack Problem)
- **규칙:** 용량이 다른 **여러 개의 배낭**이 있고, 아이템들을 이 배낭들에 나누어 담아야 한다.
- **해법:** 훨씬 복잡한 DP나 다른 근사 알고리즘이 필요하다.

이처럼 냅색 문제는 어떤 제약이 추가되느냐에 따라 다양한 형태로 발전할 수 있다.

---

## ✅ 6. 마무리

냅색 문제는 이름은 같지만 제약 조건에 따라 풀이법이 DP와 그리디로 완전히 달라지는 점이 매우 흥미로웠다. 특히 0/1 냅색 문제는 DP의 원리를 이해하는 데 가장 좋은 예제 중 하나였다. 제한된 자원(시간, 돈, 공간)으로 최대의 효율을 뽑아내야 하는 현실의 많은 문제들이 냅색 문제로 모델링될 수 있다는 점에서 꼭 마스터해야 할 중요한 주제라고 생각한다.

> 👰‍♀️ 오빠는 무인도에 딱 세개만 가져갈 수 있다면? 어떤걸 가져갈꺼야?
 👨‍💼 나는 그리디 알고리즘에 따라 가치 대비 무게가 적은 물건으로 골라갈거야
