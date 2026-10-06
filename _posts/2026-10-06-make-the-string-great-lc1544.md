---
title: "[LeetCode-1544] aA도 Aa도 짝이다 — 순서를 가정하지 말 것"
date: 2026-10-06 10:30:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/make-the-string-great/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC1544_MakeTheStringGreat.java`
- 난이도: Easy

## 문제 설명

문자열 `s`에서 **같은 글자인데 대소문자만 다른 두 글자가 붙어 있으면**(`aA`, `Bb` 등) 둘 다 지운다. 더 지울 게 없을 때까지 반복한 결과를 반환한다.

```
Input: s = "abBAcC"
Output: ""   // bB 삭제 → aAcC → aA 삭제 → cC → cC 삭제 → ""
```

## 시행착오

"붙어 있는 짝을 지운다"는 [LC1047]({% post_url 2026-09-30-remove-adjacent-duplicates-lc1047 %})와 같은 구조라 스택은 바로 떠올렸다. 막힌 건 **짝 조건**이었다.

1. **소문자 뒤에 대문자만 온다고 착각했다.** `isUpperCase(c)`일 때만 짝을 확인해서, `"Pp"`처럼 대문자 다음 소문자가 오는 경우는 그냥 쌓았다.
2. **`peekLast()`로 맨 위를 봤다.** `push`는 앞에 넣으니 가장 최근 글자는 `peek()`(앞)에 있다. `peekLast()`는 가장 오래된 글자다.
3. **`if (isEmpty)`와 다음 `if`를 `else` 없이 이어 써서**, 비었을 때 push한 뒤 아래 분기로도 내려갔다.
4. 조건을 하나로 합치면서 `isEmpty` 검사까지 같이 지웠더니, 첫 글자에서 `peek()`이 `null`을 돌려줘 `NullPointerException`이 났다.

결국 조건을 "**원래 글자는 다르고, 소문자로 바꾸면 같다**" 하나로 합치고, 그 앞에 `!deque.isEmpty() &&`를 붙여서 통과했다. 솔직히 조건이 복잡해지니까 생각을 덜 하고 대충 고치면서 돌렸던 게 시행착오를 늘렸다.

## 최종 코드

```java
public String makeGood(String s) {
    Deque<Character> deque = new ArrayDeque<>();

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (!deque.isEmpty() && c != deque.peek()
                && Character.toLowerCase(c) == Character.toLowerCase(deque.peek())) {
            deque.pop();
        } else {
            deque.push(c);
        }
    }

    StringBuilder sb = new StringBuilder();
    while (!deque.isEmpty()) {
        sb.append(deque.pollLast());
    }
    return sb.toString();
}
```

## 오늘 배운 내용

**짝 조건은 순서를 가정하지 말고 "대칭"으로 쓴다.** "a 다음 A"만 생각하면 분기가 늘어나고 빠지는 경우가 생긴다. "다르다 && 소문자로 바꾸면 같다"처럼 두 글자를 바꿔 넣어도 똑같이 성립하는 조건으로 쓰면 분기 하나로 끝난다.

**`&&`는 앞이 false면 뒤를 실행하지 않는다(단락 평가).** 그래서 `!deque.isEmpty() && ... deque.peek()` 순서로 쓰면 빈 스택에서 `peek()`을 부르지 않는다. 순서를 바꾸면 NPE가 난다.

## 오답노트

- **틀렸던 패턴 1**: 예시(`aA`)만 보고 순서를 가정. **왜**: `"Pp"` 같은 반대 순서 케이스를 손으로 안 돌려봤다.
- **틀렸던 패턴 2**: `push` 후 `peekLast()`로 맨 위 확인. **왜**: `push`=앞, `pollLast/peekLast`=뒤라는 걸 헷갈렸다. 스택의 맨 위는 `peek()`.
- **틀렸던 패턴 3**: 빈 스택에서 `peek()` → `null` → `char`로 받다가 NPE.
- **다음에 떠올릴 시점**: 조건이 복잡해져서 "대충 고쳐서 돌려보기"를 시작하면 멈추고, 예시 하나를 표로 손으로 따라가본다.

## AI라면 어떻게 풀었을까

스택 구조는 그대로 두고, 짝 조건을 아스키 코드 차이로 한 번에 쓴다.

```java
public String makeGood(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (!stack.isEmpty() && Math.abs(c - stack.peek()) == 32) {
            stack.pop();
        } else {
            stack.push(c);
        }
    }
    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) {
        sb.append(stack.pollLast());
    }
    return sb.toString();
}
```

`'a'`(97)와 `'A'`(65)의 차이는 32이고, 알파벳 전체가 같은 간격으로 배치돼 있다([ASCII 표](https://www.ascii-code.com/)). 그래서 두 글자 차이의 **절댓값이 32**면 "같은 글자, 다른 대소문자"다. `abs`가 순서(aA/Aa)를 알아서 처리해주니 조건이 한 줄로 끝난다. 다만 처음 보는 사람에겐 `32`가 무슨 뜻인지 바로 안 보이니, 읽기 쉬운 쪽은 지금의 `toLowerCase` 비교다.
