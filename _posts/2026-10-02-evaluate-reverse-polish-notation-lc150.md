---
title: "[LeetCode-150] 후위 표기법 - 숫자는 쌓고, 연산자는 꺼내서 계산한다"
date: 2026-10-02 17:00:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, simulation]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다. 895(Hard)로 가려다 아직 이르다고 판단해, 682(Baseball Game)와 같은 계열의 "스택 시뮬레이션" 문제로 한 번 더 다졌다.

- 문제 링크: https://leetcode.com/problems/evaluate-reverse-polish-notation/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0150_EvaluateReversePolishNotation.java`
- 난이도: Medium

## 문제 설명

후위 표기법(Reverse Polish Notation)으로 된 산술 식이 문자열 배열로 주어진다. 이 식을 계산한 값을 반환한다. 나눗셈은 항상 0 쪽으로 버림한다.

```
Input: tokens = ["2","1","+","3","*"]
Output: 9  // ((2+1)*3)
```

## 시행착오

힌트 없이 한 번에 통과했다. 682(Baseball Game)에서 했던 "토큰 하나씩 보면서 연산자인지 숫자인지 분기한다"는 접근을 그대로 가져왔다 — 숫자면 스택에 쌓고, 연산자면 스택에서 두 개를 꺼내서 계산한 뒤 다시 넣는다.

유일하게 신경 쓴 부분은 **뺄셈과 나눗셈의 순서**였다. `+`, `*`는 순서가 안 바뀌어도 상관없지만, `-`와 `/`는 꺼낸 순서가 중요하다 — 나중에 꺼낸 쪽(`num1`, 스택에 먼저 들어가 있던 것)이 앞쪽 피연산자이고, 먼저 꺼낸 쪽(`num2`, 가장 최근에 들어간 것)이 뒤쪽 피연산자다. `num1 - num2`, `num1 / num2` 순서를 맞게 짜서 문제없이 통과했다.

## 최종 코드

```java
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String token : tokens) {
        if (token.equals("+") || token.equals("-") || token.equals("*") || token.equals("/")) {
            int num2 = stack.pop();
            int num1 = stack.pop();
            switch (token) {
                case "+": stack.push(num1 + num2); break;
                case "-": stack.push(num1 - num2); break;
                case "*": stack.push(num1 * num2); break;
                case "/": stack.push(num1 / num2); break;
            }
        } else {
            stack.push(Integer.parseInt(token));
        }
    }
    return stack.peek();
}
```

## 오늘 배운 내용

**후위 표기법은 스택으로 평가하기 위해 애초에 설계된 표기법이다.** 숫자를 만나면 쌓고, 연산자를 만나면 "가장 최근 두 개"를 꺼내서 계산하고 다시 넣는다 — 괄호나 연산자 우선순위를 전혀 고려할 필요가 없다는 게 핵심이다. 895(Hard)가 "빈도별로 값을 분류해서 관리하는" 복잡한 보조 구조였다면, 이 문제는 "스택 하나로 끝까지 가는" 더 단순한 형태라 체감 난이도 차이가 컸다 — 같은 "Medium" 태그라도 요구하는 설계 복잡도는 다를 수 있다는 걸 확인했다.

## 오답노트

이번엔 틀린 게 없었다. `num1`/`num2`를 꺼내는 순서와 그걸 연산에 쓰는 순서(특히 `-`, `/`)를 처음부터 맞게 짠 게 한 번에 통과한 이유였다.

## AI라면 어떻게 풀었을까

`if/else if` 체인 대신 `switch` 표현식으로 연산자 분기를 정리하면 더 짧아진다.

```java
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String token : tokens) {
        switch (token) {
            case "+", "-", "*", "/" -> {
                int b = stack.pop();
                int a = stack.pop();
                stack.push(switch (token) {
                    case "+" -> a + b;
                    case "-" -> a - b;
                    case "*" -> a * b;
                    default -> a / b;
                });
            }
            default -> stack.push(Integer.parseInt(token));
        }
    }
    return stack.peek();
}
```

자바 14+의 `switch` 표현식(`->`)을 쓰면 "연산자인지 판별"과 "그 연산자로 계산하기"를 각각 하나의 `switch`로 표현할 수 있다. 로직은 완전히 같지만, `if/else if`를 네 번 반복하는 대신 분기 전체가 한눈에 들어온다는 차이가 있다.
