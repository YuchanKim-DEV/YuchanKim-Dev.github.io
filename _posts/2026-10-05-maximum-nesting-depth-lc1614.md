---
title: "[LeetCode-1614] 스택에 쌓인 개수가 곧 지금의 깊이다"
date: 2026-10-05 00:10:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/maximum-nesting-depth-of-the-parentheses/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC1614_MaximumNestingDepthOfTheParentheses.java`
- 난이도: Easy

## 문제 설명

올바른 괄호 문자열 `s`가 주어진다. 괄호가 가장 깊게 겹쳐진 횟수(중첩 깊이)를 반환한다.

```
Input: s = "(1+(2*3)+((8)/4))+1"
Output: 3  // 숫자 8이 괄호 3겹 안에 있다
```

## 시행착오

스택을 써야 한다는 건 바로 알았는데, 그다음에 "깊이를 어떻게 재지?"에서 멈췄다. `(`를 만나면 쌓고 `)`를 만나면 빼면, **그 순간 스택에 쌓여 있는 개수가 바로 지금 몇 겹 안에 있는지**라는 걸 힌트로 받고 나서 방향이 잡혔다.

그다음 두 군데에서 틀렸다.

1. **마지막에 `return stack.size();`를 반환했다.** 괄호는 항상 짝이 맞으니 반복문이 끝나면 스택은 비어서 늘 `0`이 나왔다. 열심히 계산한 `max`를 정작 반환하지 않고 있었다.
2. **`max`를 `pop()`한 다음에 쟀다.** 괄호를 닫고 나서 재면 이미 한 겹 빠진 뒤라 실제보다 1 작게 나온다. `pop()` **전에** 재도록 순서를 바꿨다.

## 최종 코드

```java
public int maxDepth(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    int max = 0;
    for (int i = 0; i < s.length(); i++) {
        if (s.charAt(i) == '(') {
            stack.push(s.charAt(i));
        } else if (s.charAt(i) == ')') {
            max = Math.max(max, stack.size());
            stack.pop();
        }
    }
    return max;
}
```

## 오늘 배운 내용

**스택에 지금 몇 개 쌓여 있는지(`size()`) 자체가 정보가 될 수 있다.** 지금까지는 스택에서 "맨 위에 뭐가 있나"(`peek`)만 봤는데, 이 문제는 "몇 개 쌓였나"가 곧 답이었다. 그리고 값을 재는 시점(넣기 전/후, 빼기 전/후)에 따라 결과가 1씩 달라질 수 있으니, "가장 깊은 순간이 언제인가"를 먼저 정하고 그 시점에 재야 한다.

## 오답노트

- **틀렸던 패턴 1**: `max`를 계산해놓고 `stack.size()`를 반환. **왜**: 반복문이 끝났을 때 스택 상태를 생각하지 않았다.
- **틀렸던 패턴 2**: `pop()` 뒤에 깊이를 잼. **왜**: "가장 깊은 순간"이 닫기 직전이라는 걸 놓쳤다.
- **다음에 떠올릴 시점**: 스택 크기로 무언가를 잴 때는 "넣은 직후/빼기 직전 중 언제 재야 가장 큰가"를 먼저 정한다. 그리고 `return` 줄에 내가 계산한 변수가 들어가 있는지 마지막에 확인한다.

## AI라면 어떻게 풀었을까

닫을 때가 아니라 **열 때** 재는 방식도 있다 — `(`를 넣은 직후가 그 순간 가장 깊은 시점이기 때문이다.

```java
public int maxDepth(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    int max = 0;
    for (char c : s.toCharArray()) {
        if (c == '(') {
            stack.push(c);
            max = Math.max(max, stack.size());
        } else if (c == ')') {
            stack.pop();
        }
    }
    return max;
}
```

지금 코드는 "닫기 직전"에, 이 버전은 "연 직후"에 잰다. 두 시점의 스택 크기는 같아서 결과도 같다. 깊이가 **늘어나는 순간에 바로 기록**하는 쪽이 "왜 여기서 재는지"가 더 직관적으로 읽힌다.
