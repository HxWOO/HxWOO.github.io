---
layout: post
title: "[BOJ] 1918 - 후위 표기식"
date: 2025-09-13 00:00:00 +0900
categories: [Algorithm_Solve]
tags: [stack, string, implementation, shunting-yard]
---

<br>

### 💻 백준 1918번: 후위 표기식

- **문제 링크:** [https://www.acmicpc.net/problem/1918](https://www.acmicpc.net/problem/1918)
- **알고리즘 분류:** 스택, 문자열 처리

## 문제 소개 🧐

우리가 일반적으로 사용하는 수식은 연산자가 피연산자 사이에 있는 **중위 표기식(Infix Notation)** 이다. 이와 달리 연산자가 피연산자 뒤에 오는 **후위 표기식(Postfix Notation)** 이 있다. 예를 들어 `A+B`는 `AB+`가 된다.

후위 표기식의 장점은 괄호 없이도 연산자 우선순위를 표현할 수 있다는 점이다. 컴퓨터가 수식을 계산할 때 스택을 이용해 효율적으로 처리할 수 있다.

이 문제는 주어진 중위 표기식을 후위 표기식으로 변환하는 것이다.

### 입력 📝

첫째 줄에 중위 표기식이 주어진다. 피연산자는 알파벳 대문자이며, 연산자는 `+`, `-`, `*`, `/`와 괄호 `(`, `)`로 이루어져 있다. 길이는 100을 넘지 않는다.

### 출력 📤

첫째 줄에 후위 표기식으로 바뀐 식을 출력한다.

---

### 첫 번째 시도: 재귀와 스택으로 어떻게든... 🤯

#### Idea

처음에는 연산자 우선순위를 고려해서 스택과 재귀를 함께 사용하면 풀 수 있을 것이라 생각했다.

- 괄호가 나오면 그 내부를 재귀 함수로 넘겨 처리한다.
- 우선순위가 낮은 연산자(+, -)가 나오면, 그 전에 스택에 쌓인 높은 우선순위 연산자(*, /)와 피연산자를 먼저 결과에 추가한다.
- 피연산자는 별도의 덱에, 연산자는 스택에 저장하며 로직을 구현했다.

#### Code

```python
import sys
from collections import deque

sys.setrecursionlimit(30000)
s = deque(sys.stdin.readline().rstrip())

def transfer_to_suffix(s: deque):
    let_deq = deque()
    op_stack = []
    answer_stack = []
    while s:
        cur = s.popleft()
        if cur == '(':  # 괄호 시작
            inner_s = deque()
            nested = 1
            while nested > 0 and s:
                cur = s.popleft()
                if cur == '(': nested += 1
                if cur == ')':
                    nested -= 1
                    if nested == 0: break
                inner_s.append(cur)
            
            parentheses = transfer_to_suffix(inner_s)
            let_deq.append(parentheses)
        elif cur == '*' or cur == '/':
            op_stack.append(cur)
        elif cur == '+' or cur == '-':
            if '*' in op_stack or '/' in op_stack:
                while let_deq:
                    val = let_deq.popleft()
                    answer_stack.append(val)
                while op_stack:
                    answer_stack.append(op_stack.pop())
            op_stack.append(cur)
        else:
            let_deq.append(cur)
    
    while let_deq:
        val = let_deq.popleft()
        answer_stack.append(val)
    while op_stack:
        answer_stack.append(op_stack.pop())

    return answer_stack

# ... 결과가 리스트의 리스트로 나와서 flatten이 필요했음 ...
print(*transfer_to_suffix(s), sep='')
```

#### Result

- **결과: 실패** 😭
- 로직이 너무 복잡해졌고, 다양한 예외 케이스(특히 중첩된 연산자 우선순위 처리)를 제대로 다루지 못했다. 재귀와 여러 개의 자료구조가 얽히면서 제어하기가 너무 어려웠다.

---

### 두 번째 시도: 션팅 야드(Shunting-yard) 알고리즘 ✨

#### Idea

첫 시도의 실패 후, 이 문제를 해결하는 정석적인 방법이 있다는 것을 알게 되었다. 바로 컴퓨터 과학자 다익스트라가 고안한 [션팅 야드(Shunting-yard) 알고리즘](https://hxwoo.github.io/computer_science/2025/09/12/shunting-yard-algorithm.html)이다. 이 알고리즘은 스택 하나를 이용해 중위 표기식을 후위 표기식으로 변환하는 매우 효율적이고 깔끔한 방법을 제공한다.

알고리즘의 규칙은 다음과 같다.

1.  피연산자(알파벳)는 바로 출력한다.
2.  연산자를 만나면, 스택의 맨 위(top)에 있는 연산자가 자신보다 우선순위가 높거나 같으면 스택에서 꺼내(pop) 출력하고, 자신을 스택에 넣는다(push). (단, 스택 top이 여는 괄호이면 멈춘다)
3.  여는 괄호 `(`는 무조건 스택에 넣는다.
4.  닫는 괄호 `)`를 만나면, 여는 괄호 `(`가 나올 때까지 스택에서 모든 연산자를 꺼내 출력한다. `(`는 스택에서 꺼내기만 하고 출력하지는 않는다.
5.  모든 토큰을 처리한 후, 스택에 남아있는 모든 연산자를 꺼내 출력한다.

#### Code

```python
import sys

s = list(sys.stdin.readline().rstrip())

precedence = {'(':0, '+':1, '-':1, '*':2, '/':2}
output = []
op_stack = []

for token in s:
    if token.isalpha():  # 1. 피연산자
        output.append(token)
    
    elif token == '(':  # 3. 여는 괄호
        op_stack.append(token)
    
    elif token == ')':  # 4. 닫는 괄호
        while op_stack and op_stack[-1] != '(':
            output.append(op_stack.pop())
        op_stack.pop()  # '(' 버리기
    else:  # 2. 연산자
        while op_stack and precedence.get(op_stack[-1], 0) >= precedence[token]:
            output.append(op_stack.pop())
        op_stack.append(token)
    
while op_stack:  # 5. 남은 연산자 처리
    output.append(op_stack.pop())

print(''.join(output))
```

#### Result

- **결과: 성공!** 🎉
- 션팅 야드 알고리즘의 규칙을 그대로 구현하니 복잡했던 문제가 아주 간단하게 풀렸다.

---

## 배운 점 및 개선할 부분 🤔

- **알고리즘의 중요성:** 복잡한 문자열 파싱이나 수식 처리 문제는 직접 규칙을 만들어 해결하려 하기보다, 이미 검증된 고전적인 알고리즘이 있는지 찾아보는 것이 훨씬 효율적이라는 것을 깨달았다.
- **션팅 야드 알고리즘:** 스택을 이용해 연산자 우선순위를 처리하는 이 알고리즘의 원리를 명확하게 이해할 수 있었다. 컴퓨터가 어떻게 수식을 해석하고 계산하는지에 대한 근본적인 이해를 돕는 좋은 경험이었다.

> "바퀴를 다시 발명하려 하지 말자." 훌륭한 선배 개발자들이 만들어 놓은 알고리즘을 배우고 활용하는 것이 성장의 지름길이라는 것을 느꼈다.

<div style="text-align: center">⁂</div>
