---
layout: post
title:  "2주차 6일차: 파이썬 구조화 - 모듈, 패키지, 클래스"
date:   2025-07-06 09:00:00 +0900
categories: [AI Engineering, Woori FISA]
tags: [Python, Module, Package, Exception Handling, Class, OOP, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

## 🐍 파이썬 모듈과 패키지: 코드 깔끔하게 관리하기

코드를 체계적으로 만들고 유지보수를 쉽게 하려고 **모듈(Module)**과 **패키지(Package)**를 사용합니다.

---

### 1. 모듈 (Module) 📦

- **정의**: 간단히 말해 **하나의 파이썬 파일 (`.py`)**입니다. 관련된 함수, 클래스, 변수들을 모아놓은 것입니다.

#### ✨ 특징
- **코드 재사용**: `import`로 언제든 다른 파일에서 불러와 쓸 수 있습니다.
- **이름 충돌 방지**: `모듈명.함수명`으로 호출해서 다른 파일의 같은 이름의 함수와 겹치지 않습니다.
- **단순성**: 기능별로 코드를 파일 단위로 나누니 구조가 명확해집니다.

#### 📝 사용 예시

```python
# my_math.py 모듈
PI = 3.14159
def add(a, b):
    return a + b
```
```python
# main.py 에서 사용
import my_math

print(f"5 + 3 = {my_math.add(5, 3)}") # 8
```

---

### 2. 패키지 (Package) 🗂️

- **정의**: 모듈들을 모아놓은 **디렉토리(폴더)**입니다. 관련된 여러 모듈을 묶어서 관리합니다.

#### ✨ 특징
- **계층적 구조**: `game.sound.echo`처럼 폴더 구조로 코드를 관리할 수 있습니다.
- **모듈화 강화**: 큰 프로젝트에서 관련된 모듈끼리 묶어두니 응집도가 높아집니다.

> **꿀팁**: 폴더 안에 `__init__.py` 파일이 있으면 파이썬이 "아, 이건 패키지구나!" 하고 인식합니다. (내용은 비어있어도 OK)

#### 📝 사용 예시

```
my_calculator/           # 패키지 루트
├── __init__.py
├── addition.py        # 덧셈 모듈
└── subtraction.py     # 뺄셈 모듈
```
```python
# main.py 에서 사용
from my_calculator import addition, subtraction

print(f"10 + 5 = {addition.add(10, 5)}") # 15
```

---

### 📊 모듈 vs 패키지

| 구분 | 모듈 (Module) | 패키지 (Package) |
| :--- | :--- | :--- |
| **정의** | 파이썬 파일 (`.py`) | 모듈을 담는 디렉토리 |
| **목적** | 코드 재사용 및 분리 | 관련된 모듈들을 그룹화 |

---

### 3. 예외 처리 (Exception Handling) 🚨

- **정의**: 프로그램 실행 중 발생할 수 있는 오류(예외)에 대처해서, 프로그램이 중단되는 것을 막는 기능입니다.

#### ✨ 기본 구조 (try-except-else-finally)

```python
try:
    # 오류가 날 수 있는 코드
    result = 10 / 0
except ZeroDivisionError as e: # 0으로 나누는 에러만 잡음
    print(f"오류 발생: {e}")
except Exception as e: # 그 외 모든 에러를 잡음
    print(f"알 수 없는 오류: {e}")
else:
    # 에러가 나지 않았을 때만 실행
    print("정상 실행 완료!")
finally:
    # 에러 여부와 상관없이 무조건 실행
    print("프로그램 종료.")
```

---

### 4. 클래스 (Class) 🏛️

- **정의**: **객체 지향 프로그래밍(OOP)**의 핵심으로, 데이터(속성)와 기능(메서드)을 하나로 묶은 **설계도**입니다.

#### ✨ 특징
- **캡슐화**: 데이터와 기능을 클래스 안에 묶고 외부 접근을 막아 정보를 보호합니다.
- **상속**: 부모 클래스의 기능을 자식 클래스가 물려받아 재사용하고 확장합니다.
- **다형성**: 똑같은 이름의 메서드라도, 다른 클래스에서 다르게 동작하는 것입니다.

#### 📝 사용 예시

```python
class Dog:
    # 초기화 메서드 (객체 만들 때 딱 한 번 실행됨)
    def __init__(self, name, age):
        self.name = name # 인스턴스 변수
        self.age = age

    # 인스턴스 메서드
    def bark(self):
        return f"{self.name}가 멍멍!"

# Dog 클래스의 인스턴스(객체) 생성
my_dog = Dog("뽀삐", 3)
print(my_dog.bark()) # 뽀삐가 멍멍!
```