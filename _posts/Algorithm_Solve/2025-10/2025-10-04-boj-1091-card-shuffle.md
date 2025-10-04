---
layout: post
title: "[BOJ] 1091 - 카드 섞기"
date: 2025-10-04 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [implementation, simulation]
---

<br>

### 💻 문제: 카드 섞기

- **문제 링크:** [https://www.acmicpc.net/problem/1091](https://www.acmicpc.net/problem/1091)
- **알고리즘 분류:** 구현(Implementation), 시뮬레이션(Simulation)

## 문제 소개 🧐

N개의 카드를 특정 규칙(수열 S)에 따라 반복적으로 섞어서, 각 카드가 지정된 플레이어(수열 P)에게 돌아가도록 하는 최소 섞기 횟수를 찾는 문제다.

| 시간 제한 | 메모리 제한 | 제출 | 정답 | 맞힌 사람 | 정답 비율 |
| --- | --- | --- | --- | --- | --- |
| 2 초 | 128 MB | 3981 | 1902 | 1457 | 51.999% |

### 입력 📝

- 첫째 줄에 N이 주어진다. N은 3보다 크거나 같고, 48보다 작거나 같은 3의 배수이다.
- 둘째 줄에 길이가 N인 수열 P가 주어진다. 수열 P에 있는 수는 0, 1, 2 중 하나이다.
- 셋째 줄에 길이가 N인 수열 S가 주어진다. 수열 S에 있는 수는 모두 N-1보다 작거나 같은 자연수 또는 0이고 중복되지 않는다.

### 출력 📤

- 첫째 줄에 몇 번 섞어야 하는지 출력한다. 만약, 섞어도 섞어도 카드를 해당하는 플레이어에게 줄 수 없다면, -1을 출력한다.

---

### 순환 주기 탐색 후 계산 😭

#### Idea (1차 시도 - 실패)

처음에는 각 카드가 섞일 때 어떤 위치들을 순환하는지 파악하려고 했다.

1.  `get_rotates` 함수로 각 카드 `i`가 셔플을 반복했을 때 거치는 위치들의 순환 리스트를 구한다.
2.  이 순환 리스트에 목표 플레이어 `P[i]`에게 해당하는 위치(`pos % 3 == P[i]`)가 없는 카드가 하나라도 있다면, 목표 달성이 불가능하므로 `-1`을 출력한다.
3.  모든 카드가 목표에 도달할 가능성이 있다면, 0번부터 셔플 횟수(`time`)를 늘려가며 모든 카드가 동시에 목표 조건을 만족하는지 확인한다.

### 첫번째 시도
#### Code (1차 시도)

```python
import sys

N = int(sys.stdin.readline().rstrip())
P = list(map(int, sys.stdin.readline().split()))
S = list(map(int, sys.stdin.readline().split()))

rotates = []

def get_rotates(start):  #O(N)
    """
    섞었을때 나오는 결과의 순환리스트를 찾음
    """
    global N, P, S
    stack = []
    stack.append(start)

    while stack:
        stack.append(S[stack[-1]])

        if stack[-1] == start:
            stack.pop()
            return stack

def shuffle(time, rotates, rotate_lens):  # O(N)
    """
    주어진 덱을 셔플
    """
    global N, P, S
    result = []

    is_target = True
    for i in range(N):
        idx = time % rotate_len_dict[i]
        result.append(rotates[i][idx])
        if rotates[i][idx] != P[i]:
            is_target = False
    
    return result, is_target

# 1. 셔플 결과의 순환 찾음
rotate_len_dict = {}
for i in range(N):  #O(N)
    rotate = get_rotates(i) #O(N)
    rotate = list(map(lambda x: x%3, rotate)) #O(N)
    rotates.append(rotate)
    rotate_len_dict[i] = len(rotate)
#O(2N^2)

# 2. 셔플해서 타겟이 될 수 있는지 판정 O(N^2)
for i in range(N):
    if not (P[i] in rotates[i]):
        print(-1)
        exit()

# 3. 셔플 시뮬레이션 실행
time = -1
while True:
    time += 1
    deck, is_good = shuffle(time, rotates, rotate_len_dict)
    if is_good:
        print(time)
        break
```
#### Result

- **결과: 실패** 😭
- **문제점:** 각 카드의 순환 주기를 계산하고, 최소공배수와 관련된 문제로 풀려고 했었다. 하지만 로직이 복잡하고 시뮬레이션 과정에서 실수가 있었다.

---

### 두 번째 시도: 우직한 시뮬레이션 ✨

#### Idea

복잡한 계산 대신, 문제의 요구사항을 그대로 따라가는 단순한 시뮬레이션으로 접근 방식을 바꿨다.

1.  `card_positions` 배열을 만들어 `i`번 카드의 현재 위치를 추적한다. (`card_positions[i]`는 `i`번 카드의 위치)
2.  초기 카드 위치를 `initial_card_positions`에 저장해둔다. 이는 나중에 셔플이 순환하여 초기 상태로 돌아왔는지 확인하여 불가능한 경우를 판정하는 데 사용된다.
3.  `while` 루프를 돌며 다음을 반복한다:
    a. 현재 카드 배치(`card_positions`)가 목표(`P`)를 만족하는지 확인한다. `i`번 카드의 위치는 `card_positions[i]`이고, 이 카드는 `card_positions[i] % 3`번 플레이어에게 간다. 이 값이 모든 `i`에 대해 `P[i]`와 일치하는지 검사한다.
    b. 일치하면 현재까지의 셔플 횟수(`count`)를 출력하고 종료한다.
    c. 일치하지 않으면 셔플 횟수를 1 증가시키고, `S` 규칙에 따라 카드를 섞는다. 즉, `i`번 카드의 다음 위치는 `S[card_positions[i]]`가 된다.
    d. 셔플 후 카드 배치가 초기 상태(`initial_card_positions`)와 같아지면, 이는 사이클이 발생했으며 목표 상태에 도달할 수 없음을 의미한다. `-1`을 출력하고 종료한다.

#### Code (2차 시도)

```python
import sys

N = int(sys.stdin.readline())
P = list(map(int, sys.stdin.readline().split()))
S = list(map(int, sys.stdin.readline().split()))

# card_positions[i]는 i번 카드의 현재 위치를 저장
# 초기 상태: i번 카드는 i번 위치에 있음
card_positions = list(range(N))

# 초기 상태를 저장하여 사이클을 통한 불가능 판정
initial_card_positions = list(range(N))

count = 0
while True:
    # 현재 card_positions 상태가 목표를 만족하는지 확인
    # "i번 카드가 P[i]번 플레이어에게 가는가?"
    # i번 카드의 위치는 card_positions[i]이고, 이 위치의 카드는 card_positions[i] % 3 플레이어에게 간다.
    is_target = True
    for i in range(N):
        if card_positions[i] % 3 != P[i]:
            is_target = False
            break
    
    if is_target:
        print(count)
        sys.exit(0)

    # 셔플 횟수 증가 및 셔플 진행
    count += 1
    
    # 다음 카드 위치 계산
    next_card_positions = [0] * N
    for i in range(N):
        # i번 카드는 현재 card_positions[i] 위치에 있으므로,
        # 다음 위치는 S[card_positions[i]]가 된다.
        next_card_positions[i] = S[card_positions[i]]
    
    card_positions = next_card_positions

    # 초기 상태로 돌아왔다면, 목표 상태에 도달 불가능
    if card_positions == initial_card_positions:
        print(-1)
        sys.exit(0)
```

#### Result

- **결과: 성공!** 🎉
- 두 번째 접근 방식인 직접 시뮬레이션으로 문제를 해결할 수 있었다.

---

## 배운 점 및 개선할 부분 🤔

-   **시뮬레이션의 힘:** 복잡한 수학적, 조합적 접근이 떠오르더라도 문제의 제약 조건(N ≤ 48)이 충분히 작다면, 때로는 문제의 동작을 그대로 코드로 옮기는 단순한 시뮬레이션이 더 효과적이고 직관적인 해결책이 될 수 있다.
-   **상태 추적의 중요성:** 이 문제의 핵심은 '어떤 카드가 어디에 있는가'를 정확히 추적하는 것이다. 1차원 배열 `card_positions`를 사용해 각 카드의 위치를 관리한 것이 주효했다. `card_positions[i] = j`는 'i번 카드가 j번 위치에 있다'는 의미로, 이 관계를 헷갈리지 않는 것이 중요하다.
-   **불가능 판정:** 셔플을 반복하다 보면 언젠가는 초기 상태로 돌아온다. 만약 목표 상태를 거치지 않고 초기 상태로 돌아왔다면, 그 이후로는 같은 과정이 반복될 뿐이므로 목표에 도달하는 것은 불가능하다. 초기 상태를 저장해두고 매 셔플마다 비교하는 것은 시뮬레이션 문제에서 루프를 탈출하는 중요한 테크닉이다.
-   **첫 실패의 교훈:** 첫 번째 접근은 각 카드를 독립적으로 보고 순환 주기를 계산하려 했다. 하지만 이 문제는 모든 카드가 *동시에* 올바른 플레이어에게 가야 하는 조건이므로, 전체 카드 덱의 상태를 하나로 보고 시뮬레이션하는 것이 더 적합했다.

> 때로는 가장 정직한 방법이 가장 빠른 길이다. 복잡한 생각보다 단순한 시뮬레이션이 답일 때가 있다.

<div style="text-align: center">⁂</div>
