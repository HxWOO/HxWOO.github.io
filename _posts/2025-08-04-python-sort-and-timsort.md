---
layout: post
title:  "[Python] 🐍 파이썬의 정렬과 팀소트(Timsort) 완벽 정리"
date:   2025-08-04 00:00:00 +0900
categories: Computer_Science
tags: [python, sort, timsort, algorithm]
---

# 🐍 Python의 sort()와 팀소트(Timsort) 정리

---

## 🎯 정렬(Sort)이란?

정렬은 **데이터를 일정한 순서(오름차순, 내림차순 등)로 나열하는 작업**입니다.  
정렬된 데이터는 이진 탐색 등 효율적인 알고리즘 활용이 가능해지므로, 컴퓨터 과학에서 매우 중요한 문제입니다.

---

## ⚖️ 정렬의 안정성 (Stability)

-   **안정 정렬 (Stable Sort)** 🟢
    -   동일한 값을 가진 데이터의 **원래 입력 순서가 유지**되는 정렬
    -   (예: 병합정렬, 버블정렬, 삽입정렬)

-   **불안정 정렬 (Unstable Sort)** 🔴
    -   동일한 값의 입력 순서가 **보장되지 않는** 정렬
    -   (예: 퀵정렬, 힙정렬, 선택정렬)

---

## 📊 주요 정렬 알고리즘 요약

| 정렬 방식 | 시간복잡도(평균) | 안정성 | 특징 및 설명 |
| :--- | :--- | :--- | :--- |
| **`버블 정렬`** | O(n²) | 🟢 안정 | 인접한 요소를 반복 비교 및 교환 |
| **`선택 정렬`** | O(n²) | 🔴 불안정 | 최솟값(또는 최댓값) 선택 후 교환 |
| **`삽입 정렬`** | O(n²) | 🟢 안정 | 이미 정렬된 부분에 삽입 |
| **`병합 정렬`** | O(n log n) | 🟢 안정 | 분할 정복, 추가 메모리 필요 |
| **`힙 정렬`** | O(n log n) | 🔴 불안정 | 힙 자료구조 사용, 제자리 정렬 |
| **`퀵 정렬`** | O(n log n) | 🔴 불안정 | 피벗 기준 분할, 평균적으로 빠름 |

---

## 🏆 파이썬의 기본 정렬: 팀소트(Timsort)

파이썬의 `list.sort()`와 `sorted()` 함수는 **팀소트(Timsort)**라는 하이브리드 정렬 알고리즘을 사용합니다.

### 🤔 팀소트(Timsort)란?

-   **Tim Peters**가 2002년 파이썬을 위해 설계한 정렬 알고리즘
-   **병합정렬(Merge Sort)**과 **삽입정렬(Insertion Sort)**를 결합한 하이브리드 방식
-   **실제 데이터가 부분적으로 이미 정렬되어 있는 경우** 매우 빠른 성능을 보임
-   파이썬, Java SE 7, Android, Swift 등 다양한 언어에서 표준 정렬로 채택

### ⚙️ 동작 원리

1.  **런(Run) 분할**
    -   입력 데이터를 **최소 run 크기(min_run, 보통 32 또는 64)**로 잘라
        각 구간(run)이 이미 정렬되어 있거나,
        삽입정렬로 빠르게 정렬될 수 있도록 함[2][3][4][5][7].
2.  **런 병합**
    -   정렬된 run들을 **병합정렬 방식**으로 합침
    -   병합 시, 스택에 쌓인 run의 크기 비율을 조절해
        **균형잡힌 병합**과 **안정성**을 확보[3][4].
3.  **갤로핑(Galloping) 모드**
    -   병합 과정에서 한 run의 값이 연속적으로 선택되는 경우,
        **이진 탐색**을 활용해 대량의 데이터를 한 번에 병합하여 효율을 높임[3][4].

### ✨ 팀소트의 주요 특징

-   **안정 정렬**: 동일한 값의 입력 순서가 보존됨
-   **최악/평균 시간복잡도**: O(n log n)
-   **부분적으로 정렬된 데이터**에서 매우 빠름
-   **실제 데이터의 패턴**(자연스러운 정렬 구간)을 최대한 활용

### 🚀 팀소트의 핵심 최적화

-   **Run 탐지 및 확장**: 이미 정렬된 구간(run)을 자동으로 인식,
    run의 길이가 min_run보다 짧으면 삽입정렬로 확장[2][3][4][7].
-   **스택 기반 병합 규칙**:
    병합할 run의 크기 비율을 엄격히 관리해
    병합 횟수를 최소화하고, 메모리 캐시 효율을 높임[3].
-   **갤로핑(Galloping) 기법**:
    한쪽 run에서 연속적으로 많은 값이 선택되면,
    이진 탐색으로 병합 속도를 가속화[3][4].

---

### 📝 팀소트의 동작 예시 (간단 요약)

1.  리스트를 `min_run` 크기로 분할
2.  각 run을 삽입정렬로 정렬
3.  인접한 run들을 병합정렬 방식으로 합침
4.  모든 run이 하나로 합쳐질 때까지 반복

---

### 💻 파이썬에서 팀소트 사용 예시

```python
numbers = [5, 3, 8, 6, 2]
numbers.sort() # 내부적으로 팀소트 사용
print(numbers) # [2, 3, 5, 6, 8]
```

---

### 📜 참고: 팀소트 구현의 핵심 코드 구조 (의사코드)

```python
def timsort(arr):
    min_run = 32
    n = len(arr)

    # 1. 각 run마다 삽입정렬 수행
    for start in range(0, n, min_run):
        end = min(start + min_run - 1, n - 1)
        insertion_sort(arr, start, end)

    # 2. run 병합 (병합정렬 방식)
    size = min_run
    while size < n:
        for left in range(0, n, 2 * size):
            mid = min(n - 1, left + size - 1)
            right = min((left + 2 * size - 1), (n - 1))
            merge(arr, left, mid, right)
        size *= 2
```

> ⚠️ 실제 구현은 훨씬 복잡하며, run의 탐지, 스택 관리, galloping 등 다양한 최적화가 적용됨

---

## 📚 참고 자료

-   [Wikipedia: Timsort][3]
-   [GeeksforGeeks: Tim Sort in Python][2]
-   [Real Python: Sorting Algorithms in Python][5]
-   [skerritt.blog: Timsort — the fastest sorting algorithm you've never heard of][4]

---
