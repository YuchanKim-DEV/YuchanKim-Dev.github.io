---
title: "[LeetCode-735] 이기면 끝이 아니라 다음 상대와 또 붙는다 — if가 아니라 while"
date: 2026-10-07 14:20:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/asteroid-collision/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0735_AsteroidCollision.java`
- 난이도: Medium

## 문제 설명

한 줄로 늘어선 소행성 배열 `asteroids`가 주어진다. 절댓값은 크기, 부호는 방향(양수=오른쪽, 음수=왼쪽)이다. 두 소행성이 만나면 작은 쪽이 터지고, 크기가 같으면 둘 다 터진다. 같은 방향끼리는 만나지 않는다. 모든 충돌이 끝난 뒤 남은 소행성을 반환한다.

```
Input: asteroids = [10, 2, -5]
Output: [10]   // 2와 -5 충돌 → -5 남음 → 10과 -5 충돌 → 10 남음
```

## 시행착오

"왼쪽으로 가는 소행성은 바로 앞(가장 최근)의 오른쪽 소행성과 부딪힌다" → 스택이라는 건 알았다. 첫 시도는 이랬다.

```java
} else if (asteroids[i] < 0) {
    if (deque.peek() < Math.abs(asteroids[i]) && deque.peek() > 0) {
        deque.pop();
        deque.push(asteroids[i]);   // 이기고 바로 push
    } else if (deque.peek() == Math.abs(asteroids[i])) {
        deque.pop();
    } else {
        deque.push(asteroids[i]);   // 져도 push
    }
}
```

테스트 6개 중 3개가 틀렸고 원인은 두 가지였다.

1. **지는 소행성도 push했다.** `[5, 10, -5]`에서 `-5`는 10에게 져서 터져야 하는데, 마지막 `else`에 "맨 위가 음수라 안 부딪힘(push가 맞음)"과 "맨 위가 더 커서 짐(아무것도 안 해야 함)"이 섞여 있었다.
2. **한 번 이기면 거기서 끝냈다.** `[10, 2, -5]`에서 `-5`가 2를 이긴 뒤 바로 push해버렸다. 실제로는 계속 왼쪽으로 가서 10과 또 부딪혀야 한다. → **`if`가 아니라 `while`**이어야 했다.

여기서 막혀서 수도코드를 받았다. 핵심은 **"지금 소행성이 살아 있는가"(`alive`)를 들고 다니면서**, 부딪히는 동안 반복하고, 반복이 끝난 뒤 살아 있을 때만 push하는 구조였다. 솔직히 이 구조를 혼자서는 못 떠올렸을 것 같다.

## 최종 코드

```java
public int[] asteroidCollision(int[] asteroids) {
    Deque<Integer> deque = new ArrayDeque<>();
    for (int i = 0; i < asteroids.length; i++) {
        boolean alive = true;

        while (alive && !deque.isEmpty() && deque.peek() > 0 && asteroids[i] < 0) {
            if (deque.peek() < Math.abs(asteroids[i])) {
                deque.pop();
            } else if (deque.peek() == Math.abs(asteroids[i])) {
                deque.pop();
                alive = false;
            } else if (deque.peek() > Math.abs(asteroids[i])) {
                alive = false;
            }
        }
        if (alive) {
            deque.push(asteroids[i]);
        }
    }

    int[] ans = deque.reversed().stream().mapToInt(Integer::intValue).toArray();
    return ans;
}
```

## 오늘 배운 내용

**"연쇄"가 일어나는 문제는 `if`가 아니라 `while`이다.** 하나를 처리하고 나서 새로 드러난 맨 위와 또 비교해야 하면 반복이다. 몬로토닉 스택(LC739)에서 "작은 걸 계속 pop"하던 것과 같은 모양이다.

**push는 반복이 끝난 뒤 한 번만.** 반복 안에서 push까지 하려니 분기가 꼬였다. "부딪히는 동안 처리 → 끝나고 살아남았으면 push"로 나누면 각 분기가 하는 일이 하나씩만 남는다.

**while 조건에 "부딪히는 조건"을 다 넣으면 "안 부딪히는 경우"는 따로 안 써도 된다.** 스택이 비었거나, 맨 위가 음수거나, cur가 양수면 while이 바로 끝나고 push로 간다.

## 오답노트

- **틀렸던 패턴 1**: 이긴 뒤 바로 push. **왜**: 연쇄 충돌(`[10, 2, -5]`)을 손으로 안 따라가봤다.
- **틀렸던 패턴 2**: "안 부딪힘"과 "짐"을 같은 `else`에 넣음. **왜**: 경우를 표로 나눠보지 않고 분기부터 썼다.
- **기타**: `Integer::intValue`를 `Integer::invalue`로 오타.
- **다음에 떠올릴 시점**: 처리 후 "다음 것과 또 비교해야 하나?"를 물어본다. 그렇다면 while + 상태 변수.

## AI라면 어떻게 풀었을까

`alive` 변수 없이도 된다. while에서는 **cur가 확실히 이기는 동안만** pop하고, 반복이 끝난 뒤 남은 경우를 if-else로 나눈다.

```java
public int[] asteroidCollision(int[] asteroids) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (int cur : asteroids) {
        while (!stack.isEmpty() && cur < 0 && stack.peek() > 0 && stack.peek() < -cur) {
            stack.pop();                       // cur가 이김 → 계속
        }
        if (stack.isEmpty() || cur > 0 || stack.peek() < 0) {
            stack.push(cur);                   // 부딪힐 상대가 없음
        } else if (stack.peek() == -cur) {
            stack.pop();                       // 비김 → 둘 다 터짐
        }
        // 그 외: 맨 위가 더 큼 → cur만 터짐 (아무것도 안 함)
    }
    int[] ans = new int[stack.size()];
    for (int i = 0; i < ans.length; i++) {
        ans[i] = stack.pollLast();
    }
    return ans;
}
```

while이 끝났을 때 가능한 상황은 셋뿐이다: 부딪힐 상대가 없음(push), 같은 크기(둘 다 pop), 상대가 더 큼(cur만 사라짐). 결과 배열은 `reversed()`(Java 21+) 대신 `pollLast()`로 뒤에서부터 꺼내서, 시험 사이트의 Java 버전과 상관없이 동작하게 했다.
