---
layout: post
title:  "SHUNTING-YARD 알고리즘: 중위 표기식을 후위 표기식으로 🚂"
date:   2025-09-13 00:00:00 +0900
categories: Computer_Science
tags: [shunting-yard, algorithm, stack, parsing, compiler, infix, postfix, dijkstra]
---

# 🚂 Shunting-Yard 알고리즘: 중위 표기식을 후위 표기식으로

---

## 📜 Shunting-Yard 알고리즘이란?

**Shunting-yard 알고리즘**은 컴퓨터 과학의 아버지 중 한 분인 **에츠허르 데이크스트라(Edsger W. Dijkstra)** 가 고안한 알고리즘이다. 이름이 참 독특한데, 마치 철도역의 **분기선(Shunting Yard)** 에서 화물열차를 목적지별로 재정렬하는 모습과 비슷하다고 해서 붙여졌다고 한다.

이 알고리즘의 핵심 목표는 **사람이 이해하기 쉬운 중위 표기식(Infix Notation)** 을 **컴퓨터가 계산하기 쉬운 후위 표기식(Postfix Notation)** 으로 변환하는 것이다.

- **중위 표기식:** `3 + 4 * 2` (연산자가 피연산자 사이에 있음)
- **후위 표기식:** `3 4 2 * +` (연산자가 피연산자 뒤에 있음, 스택으로 계산 용이)

---

## 💡 핵심 아이디어

알고리즘은 두 개의 핵심 자료구조를 사용해 중위 표기식을 한 번만 훑으면서 변환을 수행한다.

1.  **출력 큐 (Output Queue):** 최종적으로 만들어질 후위 표기식이 차곡차곡 쌓이는 공간이다.
2.  **연산자 스택 (Operator Stack):** 연산자들의 우선순위를 처리하기 위해 잠시 대기하는 공간이다. 괄호도 여기서 처리된다.

---

## ⚙️ 변환 규칙

알고리즘의 변환 과정은 다음과 같은 간단한 규칙들로 이루어진다.

1.  **피연산자 (숫자):** 토큰이 숫자라면, 즉시 출력 큐에 넣는다.

2.  **연산자:** 토큰이 연산자(`+`, `-`, `*`, `/`)라면, **연산자 스택**의 맨 위(top)에 있는 연산자와 우선순위를 비교한다.
    -   스택의 top에 있는 연산자가 **현재 연산자보다 우선순위가 높거나 같으면**, 스택에서 pop하여 출력 큐에 보낸다. (이 과정을 계속 반복)
    -   우선순위 비교가 끝나면, 현재 연산자를 스택에 push한다.

3.  **왼쪽 괄호 `(`:** 왼쪽 괄호는 무조건 연산자 스택에 push한다. 우선순위가 가장 낮은 연산자처럼 취급된다.

4.  **오른쪽 괄호 `)`:** 오른쪽 괄호가 나오면, 연산자 스택에서 **왼쪽 괄호 `(`가 나올 때까지** 모든 연산자를 pop하여 출력 큐로 보낸다. `(`는 스택에서 pop하여 버린다.

5.  **마무리:** 모든 토큰을 다 읽었다면, 연산자 스택에 남아있는 모든 연산자를 pop하여 출력 큐로 보낸다.

---

## 💻 파이썬 구현 예시

다음은 Shunting-yard 알고리즘을 파이썬으로 직접 구현한 코드다.

```python
def shunting_yard(infix_expression):
    # 연산자 우선순위 정의
    precedence = {'+': 1, '-': 1, '*': 2, '/': 2, '^': 3}
    
    output_queue = []
    operator_stack = []
    
    tokens = infix_expression.replace(' ', '')

    for token in tokens:
        # 1. 피연산자인 경우
        if token.isalnum():
            output_queue.append(token)
        # 2. 왼쪽 괄호인 경우
        elif token == '(':
            operator_stack.append(token)
        # 3. 오른쪽 괄호인 경우
        elif token == ')':
            while operator_stack and operator_stack[-1] != '(':
                output_queue.append(operator_stack.pop())
            operator_stack.pop()  # '(' 버리기
        # 4. 연산자인 경우
        else:
            while (operator_stack and 
                   operator_stack[-1] != '(' and 
                   precedence.get(operator_stack[-1], 0) >= precedence.get(token, 0)):
                output_queue.append(operator_stack.pop())
            operator_stack.append(token)

    # 5. 스택에 남은 연산자 처리
    while operator_stack:
        output_queue.append(operator_stack.pop())
        
    return "".join(output_queue)

# --- 예시 실행 ---
# 예시 1: A+B*C
print(f"A+B*C -> {shunting_yard('A+B*C')}") # 결과: ABC*+

# 예시 2: (A+B)*C
print(f"(A+B)*C -> {shunting_yard('(A+B)*C')}") # 결과: AB+C*
```

---

## 🎯 알고리즘 문제에서의 활용

알고리즘 문제에서는 단순히 "후위 표기식으로 바꾸시오"라고 나오지 않는다. 대신, 이 알고리즘의 원리를 응용해야만 풀 수 있는 형태로 출제된다.

-   **계산 결과 요구:** 중위 표기식을 주고 계산 결과를 요구한다. (후위 표기식 변환 + 스택 계산)
-   **다양한 연산자:** `^`, `%` 등 새로운 연산자와 우선순위를 추가하여 직접 처리하도록 요구한다.
-   **수식 검증:** 괄호의 짝이 맞는지, 연산자 위치가 유효한지 등 식의 유효성을 검사하는 문제로 나온다.
-   **수식 트리와 연계:** 중위 표기식으로 수식 트리를 구성하고 순회 결과를 묻는 등 자료구조와 융합하여 출제된다.

결국, **"후위 표기식 변환을 스스로 해야만 풀 수 있도록"** 문제를 설계하는 것이다.

---

## 🚀 실제 서비스에서의 응용

Shunting-yard 알고리즘은 단순히 이론에 그치지 않고, 우리가 매일 사용하는 수많은 서비스의 핵심 엔진으로 작동하고 있다.

-   **프로그래밍 언어 컴파일러/인터프리터:** 파이썬, 자바스크립트 등이 `(a + b) * c` 같은 코드를 계산할 때 내부적으로 이 원리를 사용한다.
-   **스프레드시트 (Excel, Google Sheets):** `= (A1 + B2) * 1.5` 와 같은 복잡한 수식을 파싱하고 계산하는 데 사용된다.
-   **데이터베이스 (SQL 파서):** `SELECT (salary + bonus) * tax_rate` 같은 SQL 내 수식 처리의 기반이 된다.
-   **검색 엔진 (DSL 파서):** `(status:open AND priority:high) OR assignee:"HxWOO"` 와 같은 복잡한 검색 쿼리를 해석하는 데 쓰인다.

---

## ✅ 마무리

**Shunting-yard 알고리즘**은 **사람이 읽는 중위 표기식**을 **기계가 계산 가능한 구조**로 바꾸는 아름다운 해결책이었다. 이 원리가 컴파일러, 데이터베이스, 스프레드시트 등 IT 시스템 전반에 걸쳐 사용된다는 점이 매우 흥미로웠다. 대회 문제 풀이를 넘어, 컴퓨터가 어떻게 세상을 이해하는지 엿볼 수 있는 중요한 알고리즘이라는 것을 다시 한번 느꼈다.
