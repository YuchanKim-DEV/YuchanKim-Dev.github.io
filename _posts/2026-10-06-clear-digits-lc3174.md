---
title: "[LeetCode-3174] 숫자를 만나면 방금 넣은 글자를 지운다"
date: 2026-10-06 09:45:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/clear-digits/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC3174_ClearDigits.java`
- 난이도: Easy

## 문제 설명

문자열 `s`가 주어진다. 가장 앞에 있는 숫자 하나와, 그 숫자 **왼쪽에서 가장 가까운 글자** 하나를 함께 지우는 작업을 숫자가 없어질 때까지 반복한다. 남은 문자열을 반환한다. 항상 모든 숫자를 지울 수 있는 입력만 주어진다.

```
Input: s = "cb34"
Output: ""   // '3'과 'b'를 지워 "c4" → '4'와 'c'를 지워 ""
```

## 시행착오

"왼쪽에서 가장 가까운 글자 = 가장 최근에 본 글자"라서 스택이라는 건 바로 보였다. 막힌 건 **메서드 사용법**이었다.

- `char`가 숫자인지 판단하는 방법(`Character.isDigit`)을 찾아봤다.
- 처음엔 "deque 안에 숫자가 없는지"를 확인하려 했는데, 그럴 필요 없이 **숫자는 애초에 deque에 넣지 않으면** 된다는 걸 깨닫고 방향을 정했다.

글자면 `offerFirst`로 앞에 넣고, 숫자면 `pop`으로 앞(가장 최근 글자)을 지웠다. 마지막엔 `pollLast`로 **뒤(가장 먼저 넣은 것)부터** 꺼내서 원래 순서대로 이어 붙였다.

## 최종 코드

```java
public String clearDigits(String s) {
    Deque<Character> deque = new ArrayDeque<>();
    String ans = "";
    for (int i = 0; i < s.length(); i++) {
        if (deque.isEmpty()) {
            deque.offerFirst(s.charAt(i));
        } else if (Character.isDigit(s.charAt(i))) {
            if (Character.isLetter(deque.peekLast())) {
                deque.pop();
            }
        } else if (Character.isLetter(s.charAt(i))) {
            deque.offerFirst(s.charAt(i));
        }
    }

    while (!deque.isEmpty()) {
        ans += deque.pollLast();
    }

    return ans;
}
```

## 오늘 배운 내용

**스택에서 꺼낸 걸 원래 순서로 붙이고 싶으면 반대쪽(`pollLast`)에서 꺼내면 된다.** `Deque`는 양쪽이 다 뚫려 있어서, 넣을 때는 스택처럼 앞에 쌓고, 결과를 만들 때는 뒤에서부터 빼면 뒤집을 필요가 없다.

그리고 **"무엇을 스택에 넣을지"부터 정하면 코드가 단순해진다.** 숫자는 지우는 신호일 뿐 보관할 이유가 없으니 넣지 않는다 → 스택엔 글자만 있다.

## 오답노트

- 틀린 곳은 없지만 **필요 없는 검사가 두 개** 있다.
  - `peekLast()`가 글자인지 확인: deque엔 글자만 들어가니 항상 true다. 게다가 `peekLast`는 가장 **오래된** 글자를 보는 거라, 지울 대상(가장 최근, 앞쪽)과 다른 위치를 검사하고 있다.
  - `deque.isEmpty()`일 때 무조건 넣기: 숫자를 만났을 때 deque가 비어 있는 경우는 "모든 숫자를 지울 수 있다"는 조건 때문에 생기지 않는다.
- 반복문 안 `String +=`는 어제(LC2000)에 이어 또 나왔다.
- **다음에 떠올릴 시점**: 검사를 추가하기 전에 "이 상황이 실제로 일어날 수 있나?"를 제약 조건에서 확인한다. 결과 문자열은 `StringBuilder`로.

## AI라면 어떻게 풀었을까

같은 스택 풀이에서 불필요한 분기를 빼면 이렇게 된다.

```java
public String clearDigits(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            stack.pop();          // 가장 최근 글자 지우기
        } else {
            stack.push(c);
        }
    }
    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) {
        sb.append(stack.pollLast());   // 가장 먼저 넣은 것부터 = 원래 순서
    }
    return sb.toString();
}
```

글자면 `push`, 숫자면 `pop` — 두 줄이 문제 규칙 그 자체다. 숫자를 만났을 때 스택이 비어 있을 일은 문제 조건이 막아주므로 방어 코드가 없다. `push`는 앞에 넣고 `pollLast`는 뒤에서 빼므로 꺼내는 순서가 넣은 순서와 같다 ([Java 21 Deque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html)).
