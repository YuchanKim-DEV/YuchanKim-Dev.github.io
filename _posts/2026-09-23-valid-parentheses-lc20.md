---
title: "[LeetCode-20] 짝이 맞았다고 확인만 하고 꺼내는 걸 잊으면 안 된다"
date: 2026-09-23 14:10:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/valid-parentheses/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0020_ValidParentheses.java`
- 난이도: Easy

## 문제 설명

괄호 문자(`(`, `)`, `{`, `}`, `[`, `]`)로만 이루어진 문자열 `s`가 주어진다. 여는 괄호가 같은 종류로, 올바른 순서로 닫혔는지 확인한다.

```
Input: s = "()[]{}"
Output: true

Input: s = "(]"
Output: false
```

## 시행착오

전략은 처음부터 맞게 잡았다 — 여는 괄호를 만나면 스택에 쌓고, 닫는 괄호를 만나면 `HashMap`으로 짝을 찾아서 스택 맨 위와 비교한다.

```java
Map<Character, Character> map = new HashMap<>();
map.put('}', '{');
map.put(')', '(');
map.put(']', '[');
```

문제는 실행 결과였다 — `"{[]}"`(기대값 `true`)가 계속 `false`로 나왔다. 원인은 짝이 맞는지 **확인만 하고 스택에서 꺼내는 걸(`pop()`) 안 한 것**이었다. `[`까지 쌓인 상태에서 `]`를 만나면 짝이 맞는지는 확인했지만, 스택에서 그 `[`를 빼지 않았다. 그러면 다음 `}`를 만났을 때 스택 맨 위가 여전히 `[`로 남아있어서, `{`랑 짝을 맞춰야 하는데 엉뚱한 `[`와 비교하게 되어 실패했다.

`pop()`을 추가한 뒤에도 한 번 더 걸렸다 — 루프가 끝난 뒤 스택에 뭔가 남아있는지 확인하는 부분이 없어서, `"((("`처럼 여는 괄호만 있고 하나도 안 닫힌 경우를 걸러내지 못했다. `if (!stack.isEmpty()) return false;`를 추가해서 해결했다.

## 최종 코드

```java
public boolean isValid(String s) {
    Map<Character, Character> map = new HashMap<>();
    map.put('}', '{');
    map.put(')', '(');
    map.put(']', '[');

    Deque<Character> stack = new ArrayDeque<>();
    char[] charArr = s.toCharArray();
    for (int i = 0; i < charArr.length; i++) {
        if (charArr[i] == '(' || charArr[i] == '{' || charArr[i] == '[') {
            stack.push(charArr[i]);
        } else {
            if (map.get(charArr[i]) == stack.peek()) {
                stack.pop();
            } else {
                return false;
            }
        }
    }

    if (!stack.isEmpty()) {
        return false;
    }

    return true;
}
```

## 오늘 배운 내용

**스택 문제에서 "짝을 확인하는 것"과 "짝을 확인한 다음 스택에서 제거하는 것"은 서로 다른 두 단계다.** 짝이 맞았다고 판단한 순간, 그 원소는 더 이상 "아직 안 닫힌 괄호"가 아니니 스택에서 빠져야 다음 비교가 정확해진다. 확인만 하고 상태(스택)를 갱신하지 않으면, 그다음 판단이 낡은 상태를 기준으로 이루어져 버린다. 그리고 "루프 안에서 문제가 안 생겼다"는 게 "전체가 유효하다"는 뜻은 아니다 — 루프가 끝난 뒤 남은 상태(이번엔 스택에 남은 원소)까지 확인해야 완전한 판정이 된다.

## 오답노트

- **틀렸던 패턴**: 짝이 맞는지 확인만 하고 `stack.pop()`을 호출하지 않음.
- **왜 틀렸나**: "짝이 맞다"는 판단과 "그 원소를 스택에서 없앤다"는 상태 변경을 하나의 동작으로 묶어서 생각하지 못했다.
- **틀렸던 패턴 2**: 루프 종료 후 스택이 비어있는지 확인하는 코드가 없어서 `"((("` 같은 케이스를 못 걸러냄.
- **왜 틀렸나**: "루프 안에서 return false가 한 번도 안 나왔다"를 "유효하다"와 동일시했다.
- **다음에 떠올릴 시점**: 스택/큐를 쓰는 문제에서 "조건을 확인"하는 코드를 짤 때마다, 그 확인이 "상태를 갱신해야 하는 확인"인지 자문한다. 그리고 반복문을 다 돌고 나서 자료구조(스택/큐)에 뭔가 남아있는 게 유효한 상황인지 아닌지 항상 마지막에 점검한다.

## AI라면 어떻게 풀었을까

여는 괄호를 직접 나열하는 대신, `HashMap`을 "닫는 괄호 → 여는 괄호" 하나로만 쓰고 `containsKey`로 여는/닫는 괄호를 구분하면 조건문이 더 짧아진다.

```java
public boolean isValid(String s) {
    Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');
    Deque<Character> stack = new ArrayDeque<>();

    for (char c : s.toCharArray()) {
        if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) {
                return false;
            }
        } else {
            stack.push(c);
        }
    }
    return stack.isEmpty();
}
```

여기서는 `map.put()`을 세 줄 반복하는 대신 `Map.of(...)`로 한 줄에 불변 맵을 만들었고, `stack.pop() != pairs.get(c)`처럼 **비교와 제거를 한 줄로 합쳐서** "확인했는데 꺼내는 걸 잊는" 실수 자체가 구조적으로 안 생기게 했다. 다만 `stack.isEmpty()`를 조건 안에서 먼저 확인해야 `pop()`이 빈 스택에서 예외를 던지는 걸 막을 수 있다는 점은 똑같이 신경 써야 한다.
