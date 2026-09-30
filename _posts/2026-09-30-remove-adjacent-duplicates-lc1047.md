---
title: "[LeetCode-1047] 스택에서 꺼낸 순서와 원래 순서는 반대다"
date: 2026-09-30 10:30:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC1047_RemoveAllAdjacentDuplicatesInString.java`
- 난이도: Easy

## 문제 설명

소문자 문자열 `s`가 주어진다. 인접하고 같은 두 글자를 반복해서 제거한다. 더 이상 제거할 수 없을 때의 최종 문자열을 반환한다.

```
Input: s = "abbaca"
Output: "ca"
```

## 시행착오

전략(스택에 쌓다가, 맨 위와 지금 글자가 같으면 꺼내고, 다르면 쌓는다)은 처음부터 맞게 짜서 예제를 다 통과했다. 통과는 했는데 두 가지가 걸렸다.

**1. `String ans += ...`로 결과를 이어붙이고 있었다.** `String`은 불변이라 `+=`을 할 때마다 지금까지의 문자열 전체를 복사한 새 객체를 만든다. 문자 하나 이어붙일 때마다 전체 복사가 일어나서, 입력이 커지면(`10^5`) 최악의 경우 시간 초과로 이어질 수 있다는 지적을 받고 `StringBuilder`로 바꿨다.

**2. 스택에서 꺼낸 순서를 그대로 이어붙이면 원래 순서와 반대가 된다.** `push()`는 스택 맨 앞(head)에 넣으니까, 가장 최근에 넣은 글자가 head에 있다. 다 처리한 뒤 결과를 복원할 때 `removeFirst()`로 꺼내면 "가장 최근에 넣은 것부터" 나와서 순서가 거꾸로 나온다(`"ca"`가 아니라 `"ac"`). 가장 오래전에 넣은 것부터(꼬리, tail) 꺼내야 원래 왼쪽→오른쪽 순서가 복원되므로, `removeLast()`로 고쳐서 해결했다.

## 최종 코드

```java
public String removeDuplicates(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (int i = 0; i < s.length(); i++) {
        if (!stack.isEmpty() && s.charAt(i) == stack.peek()) {
            stack.pop();
        } else {
            stack.push(s.charAt(i));
        }
    }
    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) {
        sb.append(stack.removeLast());
    }
    return sb.toString();
}
```

## 오늘 배운 내용

**스택은 "넣은 순서"와 "꺼내는 순서"가 반대(LIFO)라는 성질을, 스택 안에 데이터를 쌓을 때만이 아니라 스택에서 결과를 "복원"할 때도 계속 의식해야 한다.** 데이터를 쌓는 동안엔 이 성질이 문제를 푸는 데 쓰였지만(인접 중복 제거), 마지막에 순서가 있는 문자열로 되돌릴 땐 그 성질이 오히려 순서를 뒤집는 부작용을 낸다. `removeFirst()`(head, 최근 것)와 `removeLast()`(tail, 오래된 것) 중 어느 쪽이 "원래 순서"를 복원하는지 매번 헷갈리지 않으려면, "지금 head에 있는 게 최근 것"이라는 사실 하나만 기억하면 된다.

그리고 **`String`을 반복문 안에서 `+=`로 계속 이어붙이면 O(n²)가 된다**는 것도 다시 확인했다 — 문자열을 반복적으로 조립해야 하는 코드에서는 `StringBuilder`가 기본값이어야 한다.

## 오답노트

- **틀렸던 패턴 1**: `String += `로 반복문 안에서 결과 조립.
- **왜 틀렸나**: `String`이 불변 객체라는 걸 알고는 있었지만, 반복문 안에서 그 비용이 누적된다는 걸 실감하지 못했다.
- **고친 패턴**: `StringBuilder`로 교체.
- **틀렸던 패턴 2**: 결과 복원 시 `removeFirst()` 사용 — 순서가 뒤집힘.
- **왜 틀렸나**: `push()`가 head에 넣는다는 것과, 복원할 때 "어느 끝에서부터 꺼내야 원래 순서인지"를 연결하지 못했다.
- **고친 패턴**: `removeLast()`(가장 오래전에 넣은 것부터)로 변경.
- **다음에 떠올릴 시점**: 스택에서 여러 원소를 한 번에 꺼내서 순서 있는 결과(문자열, 배열 등)로 만들 때는, "지금 스택의 어느 쪽 끝이 가장 오래된 데이터인가"부터 확인한다.

## AI라면 어떻게 풀었을까

`Deque<Character>` 대신 `StringBuilder`를 스택처럼 직접 써도 된다 — 문자를 추가/삭제할 수 있는 자료구조라는 점에서 스택 역할을 그대로 할 수 있다.

```java
public String removeDuplicates(String s) {
    StringBuilder sb = new StringBuilder();
    for (char c : s.toCharArray()) {
        int last = sb.length() - 1;
        if (last >= 0 && sb.charAt(last) == c) {
            sb.deleteCharAt(last);
        } else {
            sb.append(c);
        }
    }
    return sb.toString();
}
```

`sb.append(c)`가 `push()` 역할을, `sb.deleteCharAt(sb.length()-1)`이 `pop()` 역할을 한다. 이 방식의 장점은 **결과를 다시 조립하는 과정 자체가 없다는 것**이다 — `StringBuilder`가 이미 정방향 순서로 문자를 들고 있으니, 처리 끝나면 그냥 `toString()`만 하면 된다. `Deque`를 썼을 때 겪었던 "꺼내는 순서가 반대"라는 문제 자체가 애초에 발생하지 않는다.
