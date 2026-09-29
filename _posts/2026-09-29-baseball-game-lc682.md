---
title: "[LeetCode-682] 몬로토닉 스택 반복 후 다시 만난 '단순 스택 시뮬레이션'"
date: 2026-09-29 15:00:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, simulation]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/baseball-game/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0682_BaseballGame.java`
- 난이도: Easy

## 문제 설명

이상한 규칙의 야구 점수 기록 문제. 문자열 리스트 `ops`의 각 원소가 정수(새 점수 기록), `'+'`(직전 두 점수의 합), `'D'`(직전 점수의 2배), `'C'`(직전 점수 무효화) 중 하나다. 모든 연산을 적용한 뒤 기록에 남은 점수의 합을 반환한다.

```
Input: ops = ["5","2","C","D","+"]
Output: 30
```

## 시행착오

지금까지(739/503/901/1475) 전부 "조건에 맞는 걸 찾을 때까지 계속 `pop()`하는" 몬로토닉 스택 패턴이었는데, 이 문제는 그 패턴이 전혀 필요 없었다. `ops[i]` 하나마다 **딱 한 번**의 동작만 일어나는 단순 스택 시뮬레이션이다. 처음엔 `while(!stack.isEmpty() && ...)`로 몬로토닉 스택 틀을 그대로 가져오려다가, 이 문제엔 "반복해서 비교하며 꺼내는" 로직 자체가 없다는 걸 지적받고 `if/else` 구조로 바꿨다.

문자열이 숫자인지 판단하는 것도 처음엔 헷갈렸는데, `"C"`/`"D"`/`"+"` 세 가지가 아니면 그냥 숫자라고 보고 `Integer.parseInt()`로 바꾸면 된다는 걸 확인하고 나서는 로직 자체는 막힘없이 짰다.

## 최종 코드

```java
public int calPoints(String[] ops) {
    int ans = 0;
    Deque<Integer> stack = new ArrayDeque<>();
    for (int i = 0; i < ops.length; i++) {
        if (ops[i].equals("C")) {
            stack.pop();
        } else if (ops[i].equals("D")) {
            stack.push(stack.peek() * 2);
        } else if (ops[i].equals("+")) {
            int num = stack.pop();
            int num2 = num + stack.peek();
            stack.push(num);
            stack.push(num2);
        } else {
            stack.push(Integer.parseInt(ops[i]));
        }
    }

    while (!stack.isEmpty()) {
        ans += stack.pop();
    }
    return ans;
}
```

## 오늘 배운 내용

**최근에 반복해서 풀었던 패턴(몬로토닉 스택)을 무의식적으로 새 문제에 그대로 투영하려는 경향이 있다는 걸 확인했다.** 901에서도 503(원형 배열)과 헷갈렸던 적이 있는데, 이번엔 739 계열 문제들과 헷갈려서 안 필요한 `while` 비교 구조를 가져오려 했다. 문제를 받으면 "이게 정말 같은 패턴인가"부터 확인하는 습관이, 비슷한 카테고리(스택) 안에서도 여전히 필요하다는 걸 다시 느꼈다. 스택이라는 자료구조는 같아도, "그 스택을 어떻게 쓰는가"(매번 한 번씩 push/pop하는 단순 시뮬레이션 vs 조건 만족할 때까지 반복 pop하는 몬로토닉 패턴)는 완전히 다른 이야기다.

## 오답노트

- **틀렸던 패턴**: 몬로토닉 스택의 `while(!stack.isEmpty() && 조건)` 틀을 이 문제에도 그대로 적용하려 함.
- **왜 틀렸나**: "스택을 쓰는 문제"라는 공통점만 보고, 실제로 필요한 동작(한 번의 push/pop vs 반복적인 조건부 pop)이 다르다는 걸 구분하지 못했다.
- **다음에 떠올릴 시점**: 스택/큐를 쓰는 문제를 만나면, "이 문제가 한 번의 연산마다 스택을 한 번만 건드리는가, 아니면 조건을 만족할 때까지 반복해서 건드리는가"부터 구분한다.

## AI라면 어떻게 풀었을까

`stack.pop()` 후 다시 `stack.push()`로 되돌리는 대신(지금 코드의 `'+'` 분기), `Deque`를 배열처럼 인덱스로 접근하는 `ArrayList`나 `Deque`의 `peek` 두 번으로 더 간결하게 쓸 수도 있다. 다만 `Deque`는 임의 인덱스 접근이 안 되므로, `ArrayList`를 스택처럼 쓰는 방식으로 바꿔보면:

```java
public int calPoints(String[] ops) {
    List<Integer> record = new ArrayList<>();
    for (String op : ops) {
        int n = record.size();
        switch (op) {
            case "C" -> record.remove(n - 1);
            case "D" -> record.add(record.get(n - 1) * 2);
            case "+" -> record.add(record.get(n - 1) + record.get(n - 2));
            default -> record.add(Integer.parseInt(op));
        }
    }
    return record.stream().mapToInt(Integer::intValue).sum();
}
```

`ArrayList`는 "맨 뒤에서 몇 번째"를 인덱스로 바로 읽을 수 있어서(`record.get(n-2)`), `'+'` 연산에서 지금 코드처럼 꺼냈다가 다시 넣는 과정이 필요 없다. 자바 21의 `switch` 표현식(`->`)을 쓰면 분기도 더 짧아진다. 다만 `Deque`가 스택 개념(맨 위만 접근)에 더 충실한 반면, `ArrayList`는 "맨 뒤 두 번째"처럼 임의 위치 접근이 필요한 문제에 더 잘 맞는다는 차이가 있다.
