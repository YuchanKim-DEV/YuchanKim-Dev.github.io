---
title: "[LeetCode-844] 부정(!)을 빠뜨리면 조건이 정확히 반대로 동작한다"
date: 2026-09-29 16:00:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, simulation]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/backspace-string-compare/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0844_BackspaceStringCompare.java`
- 난이도: Easy

## 문제 설명

문자열 `s`, `t`가 주어진다. `'#'`는 백스페이스(직전 글자 삭제)를 뜻하고, 빈 텍스트에서 백스페이스를 누르면 계속 빈 상태다. 둘 다 타이핑했을 때 결과가 같으면 `true`.

```
Input: s = "ab#c", t = "ad#c"
Output: true  // 둘 다 "ac"
```

이 문제는 사실 뒤에서부터 투 포인터로 훑으면 스택 없이도 풀리지만, 이번 배치는 "스택/큐를 다루는 근육"을 다지는 게 목적이라 일부러 스택으로 짰다.

## 시행착오

각 문자열마다 스택 하나씩 두고, 일반 문자는 `push`, `'#'`는 `pop`하는 전략은 처음부터 맞게 잡았다. 문제는 **"스택이 비어있는데 `'#'`를 또 만나면"**이었다 — 빈 스택에서 `pop()`을 부르면 예외가 터진다.

```java
if (!s_stack.isEmpty()) {
    s_stack.pop();
}
```

이 형태로 가야 한다는 걸 알고 고쳤는데, 막상 코드에 옮기면서 `!`(부정)를 빠뜨렸다.

```java
if (s_stack.isEmpty()) {   // ! 없음!
    s_stack.pop();
}
```

이러면 뜻이 정반대가 된다 — "스택이 **비어있을 때만** `pop()`한다"가 되어버려서, 비어있으면 예외가 나고(사실은 비어있을 때 `pop()`을 부르니 실제로는 크래시), 정작 비어있지 않을 때는 아무 일도 안 일어나서 `'#'`가 있어도 글자가 안 지워졌다. `!`를 추가해서 "비어있지 않을 때만 pop"으로 고치고 나서야 통과했다.

## 최종 코드

```java
public boolean backspaceCompare(String s, String t) {
    Deque<Character> s_stack = new ArrayDeque<>();
    Deque<Character> t_stack = new ArrayDeque<>();

    for (int i = 0; i < s.length(); i++) {
        if (s.charAt(i) == '#') {
            if (!s_stack.isEmpty()) {
                s_stack.pop();
            }
        } else {
            s_stack.push(s.charAt(i));
        }
    }

    for (int i = 0; i < t.length(); i++) {
        if (t.charAt(i) == '#') {
            if (!t_stack.isEmpty()) {
                t_stack.pop();
            }
        } else {
            t_stack.push(t.charAt(i));
        }
    }

    String s_str = "";
    String t_str = "";
    while (!s_stack.isEmpty()) {
        s_str += s_stack.removeFirst();
    }
    while (!t_stack.isEmpty()) {
        t_str += t_stack.removeFirst();
    }

    return s_str.equals(t_str);
}
```

## 오늘 배운 내용

**`!`(부정) 하나를 빠뜨리면 조건의 의미가 정확히 반대로 뒤집힌다.** 이번엔 의도(비어있지 않을 때만 pop)는 정확히 알고 있었는데, 그걸 코드로 옮기는 마지막 단계에서 부정 기호 하나를 놓쳤다. 조건문을 다 쓰고 나서 "이게 내가 말로 설명했던 그 조건이 맞는지" 한 번 더 읽어보는 습관이 이런 실수를 잡아준다.

## 오답노트

- **틀렸던 패턴**: `if (s_stack.isEmpty())`로 `!`를 빠뜨림 — 의도와 정반대 조건이 됨.
- **왜 틀렸나**: "비어있지 않을 때 pop"이라는 의도는 맞게 세웠지만, 코드로 옮기는 과정에서 부정 기호를 누락했다.
- **고친 패턴**: `if (!s_stack.isEmpty())`로 수정.
- **다음에 떠올릴 시점**: 조건문에 `isEmpty()`, `contains()`처럼 "긍정형" 이름의 메서드를 쓸 때, 내가 원하는 게 그 메서드의 참/거짓 중 어느 쪽인지 말로 한 번 더 확인하고 `!`가 필요한지 점검한다.

## AI라면 어떻게 풀었을까

`Deque<Character>` 두 개와 마지막에 문자열로 복원하는 과정 대신, `StringBuilder`를 스택처럼 바로 쓰면 더 짧아진다.

```java
public boolean backspaceCompare(String s, String t) {
    return build(s).equals(build(t));
}

private String build(String s) {
    StringBuilder stack = new StringBuilder();
    for (char c : s.toCharArray()) {
        if (c == '#') {
            if (stack.length() > 0) {
                stack.deleteCharAt(stack.length() - 1);
            }
        } else {
            stack.append(c);
        }
    }
    return stack.toString();
}
```

`stack.append(c)`가 `push()`, `stack.deleteCharAt(stack.length()-1)`이 `pop()` 역할을 한다 — 여전히 스택(LIFO)이지만, `Deque<Character>`로 쌓은 다음 `removeFirst()`로 순서를 맞춰 문자열로 복원하는 과정 자체가 없어진다. `StringBuilder`가 이미 정방향 순서를 들고 있어서, 처리가 끝나면 바로 `toString()`하면 끝이다. 두 문자열을 처리하는 로직이 똑같으니 `build()`로 중복도 줄였다.
