---
layout: post
title:  "2주차 7일차: 파이썬 클래스 심화 - 상속, 매직 메서드, 데코레이터"
date:   2025-07-16 09:00:00 +0900
categories: [AI Engineering, Woori FISA]
tags: [Python, Class, Inheritance, Magic Methods, Name Mangling, Decorator]
---

## 🐍 파이썬 클래스 심화 기능: 더 똑똑하게 클래스 활용하기

클래스 기초에 이어, 클래스를 더 강력하고 유연하게 만들어주는 심화 기능들을 정리했습니다. **상속, 매직 메서드, 맹글링, 데코레이터**는 OOP 활용도를 극대화하는 핵심 도구입니다.

---

### 1. 상속 (Inheritance) 👨‍👩‍👧

- **정의**: 한 클래스(자식)가 다른 클래스(부모)의 속성과 메서드를 그대로 물려받는 기능입니다. 코드 재사용에 좋고, 클래스 간의 논리적인 계층 구조를 만들 수 있습니다.

#### ✨ 특징
- **코드 재사용**: 부모 클래스 기능을 자식에서 또 만들 필요가 없습니다.
- **기능 확장**: 부모 기능을 물려받으면서, 새 기능을 추가하거나 기존 기능을 **오버라이딩(Overriding)**해서 재정의할 수 있습니다.
- **유지보수**: 공통 기능은 부모 클래스만 수정하면 모든 자식 클래스에 반영됩니다.

#### 📝 사용 예시

```python
# 부모 클래스
class Animal:
    def __init__(self, name):
        self.name = name
    def eat(self):
        return f"{self.name}이(가) 밥을 먹는다."

# 자식 클래스 (Animal 상속)
class Dog(Animal):
    # 메서드 오버라이딩
    def speak(self):
        return f"{self.name}이(가) 멍멍!"

my_dog = Dog("해피")
print(my_dog.eat())   # 부모 메서드 사용
print(my_dog.speak()) # 자식 메서드 사용
```

---

### 2. 매직 메서드 (Magic Methods) ✨

- **정의**: 이름 앞뒤에 **더블 언더스코어(`__`)**가 붙은 특별한 메서드입니다. `print()`, `len()`, `+` 같은 파이썬 내장 기능과 내 클래스를 연동시킵니다.

#### 📝 주요 매직 메서드

| 매직 메서드 | 호출되는 상황 | 설명 |
| :--- | :--- | :--- |
| `__init__(self, ...)` | 객체 생성 시 (`MyClass()`) | 객체 속성을 초기화하는 생성자입니다. |
| `__str__(self)` | `print(obj)`, `str(obj)` | 객체를 사람이 읽기 좋은 **문자열**로 변환합니다. |
| `__len__(self)` | `len(obj)` | 객체의 길이를 반환합니다. |
| `__add__(self, other)`| `obj1 + obj2` | `+` 연산자를 쓸 때 동작을 정의합니다. |

#### 📝 사용 예시: `Book` 클래스

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def __str__(self): # print() 했을 때 나올 문장
        return f"'{self.title}' by {self.author}"

    def __len__(self): # len() 했을 때 나올 값
        return self.pages

my_book = Book("파이썬 정복", "김파이", 300)
print(my_book) # '파이썬 정복' by 김파이
print(f"페이지 수: {len(my_book)}") # 페이지 수: 300
```

---

### 3. 이름 맹글링 (Name Mangling) 🔒

- **정의**: `private` 변수를 흉내 내는 기능입니다. 변수 이름 앞에 **더블 언더스코어(`__`)**를 붙이면, 파이썬이 내부적으로 `_클래스이름__변수이름` 형태로 이름을 바꿔버립니다.
- **목적**: 외부에서 속성에 직접 접근하는 것을 막고, 자식 클래스가 부모 클래스의 속성을 실수로 덮어쓰는 걸 방지합니다.

#### 📝 사용 예시

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance # 이름 맹글링 적용

    def get_balance(self):
        return f"현재 잔액: {self.__balance}원"

account = BankAccount(10000)
# print(account.__balance) # 이렇게 직접 접근하면 AttributeError 발생
print(account.get_balance()) # 메서드를 통해 접근해야 함
```

---

### 4. 데코레이터 (Decorator) 🎨

- **정의**: **다른 함수를 감싸서(wrapping)** 기존 함수 코드를 수정하지 않고도 새로운 기능을 덧붙이는 문법입니다. `@` 기호로 사용합니다.
- **용도**: 로깅, 실행 시간 측정, 권한 확인 등 여러 함수에 공통적으로 적용할 기능에 주로 사용합니다.

#### ✨ 작동 방식
1. 데코레이터 함수가 꾸며줄 함수를 인자로 받습니다.
2. 그 안에 새로운 "래퍼(wrapper)" 함수를 정의합니다.
3. 래퍼 함수 안에서 원래 함수를 실행하고, 그 전후에 추가 로직을 넣습니다.
4. 마지막으로 이 래퍼 함수를 반환합니다.

#### 📝 사용 예시: 실행 시간 측정 데코레이터

```python
import time

# 데코레이터 정의
def measure_time(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs) # 원래 함수 실행
        end = time.time()
        print(f"'{func.__name__}' 실행 시간: {end - start:.4f}초")
        return result
    return wrapper

class Calculator:
    @measure_time # 데코레이터 적용
    def sum_huge_list(self, n):
        return sum(range(n))

calc = Calculator()
calc.sum_huge_list(10000000)
# 'sum_huge_list' 실행 시간: 0.1234초 (환경마다 다름)
```