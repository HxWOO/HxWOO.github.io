---
layout: post
title:  "1주차 4일차: 파이썬 핵심 개념 - Iterable, Docstring, Decorator"
date:   2025-07-10 09:00:00 +0900
categories: [AI Engineering, Woori FISA]
tags: [Python, Iterable, Iterator, Docstring, Decorator, *args, **kwargs]
---

### 🔁 Iterable 객체

- 내부에 여러 요소를 가지고 있으며, *한 번에 하나씩* 꺼낼 수 있는 **반복(iteration) 가능한 객체**입니다.
- **리스트, 문자열, 집합(Set), 딕셔너리** 등이 대표적인 이터러블 객체입니다.
- 파이썬의 `for`문과 같은 반복문에서 활용 가능합니다.

#### ✨ 특징

- **반복 가능**: `for`문 등에서 사용할 수 있습니다.
- **프로토콜 만족**:
  - `__iter__()` 메서드를 가지고 있으며, 이 메서드는 이터레이터(iterator) 객체를 반환합니다.
  - 이터레이터는 `__next__()` 메서드를 통해 요소를 하나씩 반환합니다.

#### 🔎 iterable vs iterator

| 구분 | iterable(이터러블) | iterator(이터레이터) |
| :-- | :-- | :-- |
| **정의** | 반복 가능한 객체 | 실제로 반복을 수행하는 객체 |
| **예시** | 리스트, 문자열, 집합, 딕셔너리 등 | `iter()`의 결과, 리스트의 이터레이터 등 |
| **필수 메서드** | `__iter__()` | `__iter__()`, `__next__()` |
| **사용 목적** | 반복문에 전달되어 반복을 시작함 | 반복문 내부에서 실제 값을 차례로 반환함 |

#### 🛠️ 직접 iterator 만들기

```python
class CountUpTo:
    def __init__(self, max):
        self.max = max
        self.current = 1

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.max:
            raise StopIteration
        self.current += 1
        return self.current - 1

# 사용 예시
counter = CountUpTo(5)
for number in counter:
    print(number)
```
> **참고**: `__init__` 메서드의 오타를 수정했습니다.

---

### 📝 Docstring

- **모듈, 함수, 클래스, 메서드**의 정의 바로 아래에 위치하는 설명 문자열입니다.
- 코드 블록의 **용도, 사용법, 동작 방식** 등을 문서화하는 데 사용됩니다.
- `__doc__` 속성으로 접근하거나 `help()` 함수로 확인할 수 있습니다.

#### ✏️ 작성법

- **모듈**: 파일의 맨 위에 작성합니다.
- **클래스/함수/메서드**: 선언 직후, `"""` (큰따옴표 3개)를 사용하여 작성합니다.

```python
def add(a, b):
    """
    두 숫자를 더한 값을 반환합니다.

    Args:
        a (int): 첫 번째 숫자
        b (int): 두 번째 숫자

    Returns:
        int: 두 숫자의 합
    """
    return a + b
```

#### 💡 특징 & 활용

- 코드의 사용법, 인자, 반환값, 발생 가능한 예외 등을 설명할 수 있습니다.
- **Sphinx**와 같은 자동 문서화 도구에서 활용됩니다.
- 코드의 유지보수, 협업, 재사용성을 향상시킵니다.

---

### 🛠️ Destructive & Undestructive Function

#### ⚠️ Destructive function (파괴적 함수)

- **입력 객체(특히 mutable 객체)의 내부 상태를 직접 변경**하는 함수입니다.
- 예시: 리스트의 `append()`, `sort()` 메서드 등.

```python
def destructive_append(lst, item):
    lst.append(item) # 원본 리스트가 직접 변경됨
```


#### 🟢 Undestructive function (비파괴적 함수)

- **입력 객체를 변경하지 않고, 새로운 객체를 반환**하는 함수입니다.
- 예시: `sorted()` 함수, `+` 연산자 등.

```python
def undestructive_append(lst, item):
    return lst + [item] # 새로운 리스트를 반환하며, 원본은 그대로 유지됨
```

| 구분 | Destructive function | Undestructive function |
| :-- | :-- | :-- |
| **원본 객체 변화** | 변경됨 | 변경되지 않음 |
| **반환 값** | 보통 `None` 또는 변경된 객체 자신 | 보통 새로운 객체 |
| **예시** | `list.append()`, `list.sort()` | `sorted()`, `+` 연산 |
| **사용 목적** | 메모리 절약, 객체 상태의 직접 수정 | 안전성, 데이터의 불변성 유지 |

---

### 🌟 가변 인자: *args, **kwargs

- **함수의 인자 개수나 형식을 유연하게 처리**하기 위해 사용됩니다.
- 대표적으로 `*args`와 `**kwargs`가 있습니다.

#### *args

- 여러 개의 **위치 인자(positional arguments)**를 하나의 **튜플(tuple)**로 묶어서 받습니다.

```python
def print_args(*args):
    for arg in args:
        print(arg)

print_args(1, 2, 3) # 1, 2, 3이 차례로 출력됨
```

#### **kwargs

- 여러 개의 **키워드 인자(keyword arguments)**를 하나의 **딕셔너리(dict)**로 묶어서 받습니다.

```python
def print_kwargs(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_kwargs(name="Alice", age=30)
# name: Alice
# age: 30
```

#### *args와 **kwargs 함께 쓰기

- 함수 정의 시, 항상 `*args`가 `**kwargs`보다 먼저 와야 합니다.


| 구분 | 역할 | 함수 내 자료형 |
| :-- | :-- | :-- |
| ***args** | 위치 인자를 튜플로 받음 | `tuple` |
| ****kwargs** | 키워드 인자를 딕셔너리로 받음 | `dict` |

---

### 🪄 Decorator

- **기존 함수나 메서드의 코드를 직접 수정하지 않고, 기능을 확장하거나 변경**할 때 사용하는 문법입니다.
- `@데코레이터이름` 형태로 함수 선언부 위에 붙여서 사용합니다.
- 다른 함수를 인자로 받아 새로운 함수를 반환하는 **고차 함수(Higher-Order Function)**입니다.

#### 🏗️ 기본 사용법

```python
def my_decorator(func):
    def wrapper():
        print("함수 실행 전")
        func()
        print("함수 실행 후")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()

# 출력:
# 함수 실행 전
# Hello!
# 함수 실행 후
```

#### 💡 활용 예시

- **로깅(Logging)**: 함수 호출 기록을 남깁니다.
- **실행 시간 측정**: 함수의 실행 시간을 계산합니다.
- **입력값 검증**: 함수에 전달되는 인자의 유효성을 검사합니다.
- **접근 권한 제어**: 특정 사용자만 함수를 실행할 수 있도록 제어합니다.

#### 📝 참고

- 데코레이터를 사용할 때 원본 함수의 메타 정보(이름, 독스트링 등)를 보존하기 위해 `functools.wraps`를 사용하는 것이 좋습니다.
- 여러 개의 데코레이터를 중첩하여 적용할 수 있습니다.