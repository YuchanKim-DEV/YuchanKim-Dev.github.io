---
title: "[LeetCode-2696] 짝을 지우면 끝 — 이번엔 while이 아니라 if"
date: 2026-10-07 15:05:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/minimum-string-length-after-removing-substrings/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC2696_MinimumStringLengthAfterRemovingSubstrings.java`
- 난이도: Easy

## 문제 설명

대문자로만 이루어진 문자열 `s`에서 부분 문자열 `"AB"` 또는 `"CD"`를 원하는 만큼 지울 수 있다. 지우고 나면 양옆이 붙어서 새로운 `"AB"`/`"CD"`가 생길 수 있다. 만들 수 있는 가장 짧은 길이를 반환한다.

```
Input: s = "ABFCACDB"
Output: 2   // AB 삭제 → FCACDB → CD 삭제 → FCAB → AB 삭제 → FC
```

## 시행착오

[LC1544]({% post_url 2026-10-06-make-the-string-great-lc1544 %})와 같은 "붙은 짝 지우기"라 스택은 바로 떠올렸다. 그런데 바로 전에 푼 [LC735]({% post_url 2026-10-07-asteroid-collision-lc735 %})의 `while` 패턴을 그대로 가져오면서 꼬였다.

1. **첫 시도**: `while`로 짝을 반복 확인하고, 짝을 지우면서 `s`를 `substring`으로 잘랐고, 짝을 지운 뒤에도 push했고, `s.length()`를 반환했다. 게다가 빈 스택에서 `peek()`을 해서 NPE.
2. **두 번째 시도**: `if/else`로 바꾸고 `deque.size()`를 반환하도록 고쳤다. 빈 스택 문제는 **맨 앞에 첫 글자를 미리 push**하는 걸로 피하려 했는데, 반복문이 `i = 0`부터 돌아 첫 글자가 두 번 들어가서 모든 답이 1씩 컸다. 그리고 `"ABAB"`처럼 **중간에 스택이 다시 비는** 경우는 미리 넣어두는 걸로는 막을 수 없었다.
3. **최종**: 미리 push를 지우고, `if` 조건 맨 앞에 `!deque.isEmpty() &&`를 붙여 통과.

## 최종 코드

```java
public int minLength(String s) {
    Deque<Character> deque = new ArrayDeque<>();

    for (int i = 0; i < s.length(); i++) {
        if (!deque.isEmpty()
                && ((deque.peek() == 'A' && s.charAt(i) == 'B')
                || (deque.peek() == 'C' && s.charAt(i) == 'D'))) {
            deque.pop();
        } else {
            deque.push(s.charAt(i));
        }
    }
    return deque.size();
}
```

## 오늘 배운 내용

**`while`인지 `if`인지는 "처리 후에도 지금 원소가 살아 있나?"로 정한다.**
- LC735: `-5`가 2를 이겨도 `-5`는 살아 있다 → 다음 상대와 또 붙는다 → `while`
- LC2696: `B`가 `A`와 짝이 되면 `B`도 같이 사라진다 → 더 비교할 게 없다 → `if`

**남은 문자열은 스택이 들고 있다.** 원본 `s`를 자를 필요가 없다. 스택에 쌓인 것 = 지금까지 살아남은 글자, 그 개수 = 답.

**빈 스택 방어는 "시작할 때 한 번"이 아니라 "매번"이다.** 짝을 지우다 보면 중간에 언제든 다시 빌 수 있다.

## 오답노트

- **틀렸던 패턴 1**: 직전 문제(735)의 `while`을 그대로 적용. **왜**: 패턴의 모양만 기억하고 "왜 while이었는지"를 따지지 않았다.
- **틀렸던 패턴 2**: 빈 스택 `peek()` → NPE (1544에 이어 두 번째). **왜**: 짝 조건을 먼저 쓰고 방어를 나중에 생각했다.
- **틀렸던 패턴 3**: 첫 글자 미리 push + 반복문 0부터 → 중복.
- **다음에 떠올릴 시점**: `peek()`을 쓰는 순간 그 앞에 `!isEmpty() &&`가 있는지 확인한다.

## AI라면 어떻게 풀었을까

구조는 같고, 맨 위 글자를 변수에 한 번 꺼내두면 조건이 더 읽기 쉬워진다.

```java
public int minLength(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        char top = stack.isEmpty() ? ' ' : stack.peek();
        if ((top == 'A' && c == 'B') || (top == 'C' && c == 'D')) {
            stack.pop();
        } else {
            stack.push(c);
        }
    }
    return stack.size();
}
```

빈 스택일 때 `top`에 짝이 될 수 없는 값(`' '`)을 넣어두면, 짝 조건에서 `isEmpty` 검사를 뺄 수 있다. `peek()`을 한 번만 부르니 괄호도 줄고 "맨 위(top)와 지금 글자(c)가 AB 또는 CD인가"가 그대로 읽힌다.
