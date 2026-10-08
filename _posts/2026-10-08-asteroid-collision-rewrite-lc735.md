---
title: "[LeetCode-735] 백지 재작성 - 상태 변수는 '언제 되돌리는가'까지 세트"
date: 2026-10-08 10:55:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> [첫 풀이 글]({% post_url 2026-10-07-asteroid-collision-lc735 %})에서는 수도코드를 받아 통과했다. 이번엔 수도코드 없이 백지에서 다시 짠 재작성 글이다. 스택·큐 개념은 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})을 참고.

- 문제 링크: https://leetcode.com/problems/asteroid-collision/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0735_AsteroidCollision.java`
- 난이도: Medium

## 문제 설명

(자세한 설명은 [첫 풀이 글]({% post_url 2026-10-07-asteroid-collision-lc735 %}) 참고) 소행성 배열에서 양수는 오른쪽, 음수는 왼쪽으로 움직이고, 만나면 작은 쪽이 터지며 크기가 같으면 둘 다 터진다. 충돌이 모두 끝난 뒤 남은 소행성을 구한다.

## 시행착오

### 1차: `topNum`을 반복문 밖에서 한 번만 읽음

```java
for (int i = 0; i < asteroids.length; i++) {
    int topNum = deque.peek();
    while (isAlive) {
        if (topNum < Math.abs(asteroids[i])) {
            deque.pop();
        } ...
```

`pop()`을 해도 `topNum`은 그대로라 같은 값으로 `pop()`이 또 실행된다. `while (isAlive)`만으로는 끝나는 조건도 없었다. 질문("pop하면 topNum은 어떻게 돼야 하나", "while은 언제 끝나야 하나")을 받고 **`deque.peek()`를 매번 읽고, `while` 조건에 `!isEmpty()`, `새것 < 0`, `top > 0`을 넣는 구조**로 스스로 고쳤다.

### 2차: 문법 오류

`stream(mapToInt::intValue)`처럼 `mapToInt`를 `stream()` 괄호 안에 넣었다. `stream()`은 인자가 없고, `mapToInt(Integer::intValue)`는 따로 이어 붙이는 메서드다.

### 3차: `isAlive`를 리셋하지 않음 (가장 중요)

```java
boolean isAlive = true;          // for 바깥에서 한 번만 선언
for (int i = 0; i < asteroids.length; i++) {
    while (isAlive && ...) { ... isAlive = false; ... }
    if (isAlive) deque.push(asteroids[i]);
}
```

제공된 테스트 6개는 전부 통과했다. 하지만 `[8, -8, 1, 2]`는 정답이 `[1, 2]`인데 `[]`이 나왔다. `-8`이 8과 함께 터지면서 `isAlive`가 `false`가 되고, **다음 소행성 `1`, `2`에서도 계속 `false`라서 push가 안 된 것**이다. `[1, -1, -1]`도 같은 이유로 `[-1]`이 아니라 `[]`이었다. 테스트에 이 두 케이스를 추가해서 실패를 눈으로 확인한 뒤, `for` 안 첫 줄에 `isAlive = true;`를 넣어 해결했다.

## 최종 코드

```java
public int[] asteroidCollision(int[] asteroids) {
    Deque<Integer> deque = new ArrayDeque<>();
    boolean isAlive = true;
    for (int i = 0; i < asteroids.length; i++) {
        isAlive = true;                       // 소행성마다 리셋
        while (isAlive && !deque.isEmpty() && asteroids[i] < 0 && deque.peek() > 0) {
            if (deque.peek() < Math.abs(asteroids[i])) {
                deque.pop();
            } else if (deque.peek() == Math.abs(asteroids[i])) {
                deque.pop();
                isAlive = false;
            } else if (deque.peek() > Math.abs(asteroids[i])) {
                isAlive = false;
            }
        }

        if (isAlive) {
            deque.push(asteroids[i]);
        }
    }

    return deque.reversed().stream().mapToInt(Integer::intValue).toArray();
}
```

## 추적표: `[8, -8, 1, 2]`

| i | 현재 | 덱(앞=맨 위) | 처리 | isAlive (리셋 없음) | isAlive (리셋 있음) |
|---|---|---|---|---|---|
| 0 | 8 | [] | 부딪힐 상대 없음 → push | true | true |
| 1 | -8 | [8] | 8 == 8 → pop, 둘 다 터짐 | false | false |
| 2 | 1 | [] | 상대 없음 → push? | **false라서 push 안 됨** | true → push |
| 3 | 2 | [1] | 양수끼리 → push? | **false라서 push 안 됨** | true → push |

결과: 리셋 없음 `[]`, 리셋 있음 `[1, 2]`.

## 오늘 배운 내용

**상태 변수는 만드는 것만큼 "언제 되돌리는가"까지 세트로 설계해야 한다.** `isAlive`는 "지금 보고 있는 소행성 하나"의 상태인데, 선언 위치가 `for` 바깥이면 소행성이 바뀌어도 값이 따라온다. 변수의 수명(scope)이 그 변수가 뜻하는 대상의 수명과 같아야 한다. 선언을 `for` 안에 두거나, 밖에 뒀다면 반복 시작 때 초기화한다.

## 오답노트

- **틀렸던 지점 1**: `topNum`을 `while` 밖에서 한 번만 읽음. **왜**: `pop` 뒤에 맨 위가 바뀐다는 걸 코드에 반영하지 않았다. **고침**: 비교할 때마다 `peek()`로 새로 읽는다.
- **틀렸던 지점 2**: `isAlive` 리셋 누락. **왜**: 제공된 예시 6개만 믿고 "통과 = 정답"이라 생각했다. **고침**: 충돌이 일어난 뒤에도 이어지는 입력(`[8,-8,1,2]`)을 직접 만들어 확인.
- **다음에 떠올릴 시점**: 반복문 밖에 `boolean`을 하나 만들었다면, 그 변수가 **언제 true로 돌아오는지** 한 줄로 말해본다. 또 테스트는 "끝나는 케이스"뿐 아니라 **"끝난 뒤에도 입력이 남아있는 케이스"**를 하나 넣는다.
- **이번에 좋았던 점**: 지난번 반복 실수였던 `peek()` 전 `isEmpty` 확인이 처음부터 들어갔다.

## 꿀팁

플래그 변수의 리셋 실수는 "한 번 true/false가 되고 끝나는 문제"와 "반복마다 다시 판정하는 문제"를 헷갈릴 때 나온다. 반복마다 새로 판정해야 하는 값은 **반복문 안에서 선언**하면 리셋을 잊을 일이 없다.

## AI라면 어떻게 풀었을까

별도의 `isAlive` 대신 **현재 소행성 `cur` 자체를 생존 표시**로 쓰고, 터지면 `cur = 0`으로 바꾼다(문제상 0은 입력에 없다). `cur`는 `for (int cur : asteroids)`가 매번 새로 주니까 리셋을 깜빡할 일이 없다. 또 `Deque` 대신 `int[]` 배열 + `top` 포인터로 스택을 구현하면 `reversed()` 없이 `Arrays.copyOf`로 순서 그대로 결과를 만든다.

```java
public int[] asteroidCollisionAlt(int[] asteroids) {
    int[] stack = new int[asteroids.length];
    int top = -1;
    for (int cur : asteroids) {
        while (top >= 0 && cur < 0 && stack[top] > 0) {
            if (stack[top] < -cur) {
                top--;          // cur가 이김 → 다음 상대와 계속 충돌
                continue;
            }
            if (stack[top] == -cur) {
                top--;          // 비김 → 둘 다 터짐
            }
            cur = 0;            // cur 사망 표시
            break;
        }
        if (cur != 0) {
            stack[++top] = cur;
        }
    }
    return Arrays.copyOf(stack, top + 1);
}
```

트레이드오프: 별도 변수가 없어 실수할 곳이 줄지만, `0`이라는 특수값을 "죽음"으로 약속하는 방식이라 입력에 0이 있을 수 있는 문제에는 쓸 수 없다. 배열 스택은 크기를 미리 알아야 하지만 여기선 `asteroids.length`가 상한이라 충분하다.
