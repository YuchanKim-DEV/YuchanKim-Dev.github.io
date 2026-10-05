---
title: "[LeetCode-2000] 뒤집기는 스택에 넣었다 빼면 끝"
date: 2026-10-05 17:20:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/reverse-prefix-of-word/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC2000_ReversePrefixOfWord.java`
- 난이도: Easy

## 문제 설명

문자열 `word`와 글자 `ch`가 주어진다. `word`에서 `ch`가 **처음 나오는 위치까지**만 뒤집고, 나머지는 그대로 둔다. `ch`가 없으면 `word`를 그대로 반환한다.

```
Input: word = "abcdefd", ch = 'd'
Output: "dcbaefd"   // "abcd"만 뒤집힘
```

## 시행착오

힌트 없이 한 번에 통과했다.

- `indexOf(ch)`로 뒤집을 끝 위치를 먼저 찾았다.
- 그 위치까지 스택에 `push` → 다 꺼내면 거꾸로 나온다(LIFO).
- 나머지 뒷부분은 순서대로 이어 붙였다.

`ch`가 없을 때 `indexOf`가 `-1`을 돌려주는데, 그러면 첫 반복문(`i <= -1`)이 아예 안 돌고 뒷부분 반복문이 `0`부터 전체를 붙인다. **따로 예외 처리를 안 해도 저절로 원본이 나온다.**

방금 배운 `poll()`도 써봤다. `push`로 앞에 넣고 `poll`로 앞에서 빼니까 스택처럼 동작한다.

## 최종 코드

```java
public String reversePrefix(String word, char ch) {
    Deque<Character> stack = new ArrayDeque<>();
    String ans = "";
    int idx = word.indexOf(ch);
    for (int i = 0; i <= idx; i++) {
        char c = word.charAt(i);
        stack.push(c);
    }
    while (!stack.isEmpty()) {
        ans += stack.poll();
    }

    for (int j = idx + 1; j < word.length(); j++) {
        ans += word.charAt(j);
    }
    return ans;
}
```

## 오늘 배운 내용

**"거꾸로"가 보이면 스택.** 넣은 순서의 반대로 나오는 게 스택의 성질(LIFO)이라, 뒤집기는 "전부 넣고 전부 빼기"로 끝난다.

그리고 `indexOf`가 못 찾았을 때 돌려주는 `-1`이 반복문 범위와 맞물리면, 특수 케이스를 따로 안 써도 되는 경우가 있다. 분기를 추가하기 전에 "이 값이 들어가면 반복문이 어떻게 도나?"를 먼저 따져보면 코드가 짧아진다.

## 오답노트

- 이번엔 틀린 곳 없음.
- **아쉬운 점**: 반복문 안에서 `String +=`를 썼다. 통과는 하지만 길이가 길어지면 느려진다(아래 참고).
- **다음에 떠올릴 시점**: 반복문 안에서 문자열을 이어 붙이고 있으면 `StringBuilder`로 바꾼다.

## AI라면 어떻게 풀었을까

스택으로 뒤집는 아이디어는 그대로 두고, 문자열을 만드는 방식만 바꿨다.

```java
public String reversePrefix(String word, char ch) {
    int idx = word.indexOf(ch);
    Deque<Character> stack = new ArrayDeque<>();
    for (int i = 0; i <= idx; i++) {
        stack.push(word.charAt(i));
    }
    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) {
        sb.append(stack.pop());
    }
    sb.append(word.substring(idx + 1));
    return sb.toString();
}
```

`String`은 한 번 만들면 수정할 수 없어서(immutable), `ans += c`를 할 때마다 기존 내용을 복사한 새 문자열을 만든다. 반복문 안에서 n번 하면 복사량이 1+2+…+n이라 O(n²)이 된다. `StringBuilder`는 내부 버퍼에 이어 쓰기만 해서 O(n)이다 ([Java 21 StringBuilder 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html)). 뒷부분은 `substring(idx + 1)`로 한 번에 붙였다 — `idx`가 `-1`이면 `substring(0)`이라 전체 문자열이 그대로 붙는다.
