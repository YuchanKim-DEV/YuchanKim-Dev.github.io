---
title: "[LeetCode-2073] 큐에는 값이 아니라 '누구인지'를 넣는다"
date: 2026-10-10 23:20:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, queue, simulation]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/time-needed-to-buy-tickets/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC2073_TimeNeededToBuyTickets.java`
- 난이도: Easy

## 문제 설명

n명이 줄을 서서 표를 산다. `tickets[i]`는 i번째 사람이 사고 싶은 표의 수다. 한 사람은 1초에 표 1장만 살 수 있고, 더 사려면 줄 맨 뒤로 다시 가야 한다. 살 표가 없으면 줄을 떠난다. 처음 k번째에 서 있던 사람이 표를 다 사는 데 걸리는 시간을 구한다.

```
Input: tickets = [2, 3, 2], k = 2
Output: 6
```

## 시행착오

이틀을 쉬고 복귀한 날이라 괄호가 아닌 큐 시뮬레이션을 골랐다. 그런데도 세 번을 고쳤다.

### 1차: 인덱스로 줄을 직접 돌림

```java
while (tickets[k] != 0) {
    if (tickets[idx] > 0) {
        tickets[idx]--;
        counter++;
        deque.offer(tickets[idx]);        // 만들기만 하고 쓰지 않음
        if (idx == tickets.length - 1) {
            idx = idx % 2;                // n이 3일 때만 우연히 맞는 값
        } else {
            idx++;
        }
    }
}
```

예시 1(`[2,3,2]`)은 6이 나왔지만, 예시 2(`[5,1,1,1]`)에서 무한 루프였다. 원인은 두 가지였다.

1. **`idx % 2`는 n=3일 때만 맞는다.** 마지막 인덱스가 2라서 `2 % 2 = 0`이 우연히 맞았을 뿐이다.
2. **`idx++`가 `if` 안에 있다.** 이미 0장인 사람을 만나면 `idx`가 움직이지 않아 같은 자리에서 영원히 돈다.

그리고 `deque`는 `offer`만 하고 한 번도 꺼내지 않았다. 사실상 큐를 쓰지 않은 코드였다.

### 2차: Map으로 "번호-표 수 쌍"을 만들려다 컴파일 에러

```java
map.put(i, tickets[i]);
deque.offer(map.getKey(tickets[i]), map.getValue(i));   // 컴파일 에러 3개
```

`getKey()`와 `getValue()`는 `Map`이 아니라 `Map.Entry`의 메서드다. `offer`는 인자를 하나만 받는다. 다만 이 줄에서 **"번호와 표 수를 한 쌍으로 들고 다녀야 k번 사람을 알아볼 수 있다"**는 방향을 스스로 찾아냈다. 방향은 맞았다.

### 3차: 값의 복사본을 줄임

```java
int first = deque.poll();
if (first > 0) { first--; deque.offer(first); counter++; }
...
while (tickets[i] != 0) { ... }     // tickets[i]는 한 번도 안 변한다
```

덱에서 꺼낸 `first`는 **값의 복사본**이다. 복사본을 줄여도 `tickets[i]`는 그대로여서 `while`이 끝나지 않았고, 덱이 비면 `poll()`이 null이라 NPE가 났다. 또 덱에 남은 표 수만 있어서 꺼낸 사람이 k번인지 알 수 없었다. 여기서 수도코드를 받았다.

## 최종 코드

```java
public int timeRequiredToBuy(int[] tickets, int k) {
    Deque<Integer> deque = new ArrayDeque<>();
    int counter = 0;
    for (int i = 0; i < tickets.length; i++) {
        deque.offer(i);                 // 큐에는 "사람 번호"를 넣는다
    }

    while (!deque.isEmpty()) {
        int p = deque.poll();
        tickets[p]--;                   // 남은 표 수는 tickets[]가 들고 있다
        counter++;

        if (p == k && tickets[p] == 0) {
            return counter;
        }
        if (tickets[p] > 0) {
            deque.offer(p);             // 남았으면 다시 맨 뒤로
        }
    }
    return counter;
}
```

## 추적표: `[2, 3, 2]`, k = 2

| 초 | 꺼낸 사람 p | 처리 후 tickets | 줄(앞→뒤) | k가 끝났나? |
|---|---|---|---|---|
| 1 | 0 | [1,3,2] | 1, 2, 0 | 아니오 |
| 2 | 1 | [1,2,2] | 2, 0, 1 | 아니오 |
| 3 | 2 | [1,2,1] | 0, 1, 2 | 아니오 (1장 남음) |
| 4 | 0 | [0,2,1] | 1, 2 | 아니오 (0번은 떠남) |
| 5 | 1 | [0,1,1] | 2, 1 | 아니오 |
| 6 | 2 | [0,1,0] | 1 | **예 → 6 반환** |

## 오늘 배운 내용

**큐에는 "값"이 아니라 "누구인지(식별자)"를 넣는다.** 줄이 계속 돌아가면 사람의 위치는 바뀐다. 남은 표 수만 큐에 넣으면 꺼낸 값이 k번 사람의 것인지 알 수 없다. 그래서 변하지 않는 번호를 큐에 넣고, 변하는 상태(남은 표 수)는 `tickets[]`에서 번호로 찾아 읽고 바꾼다. 큐는 "순서"를, 배열은 "상태"를 맡는 분업이다.

## 오답노트

- **틀렸던 지점 1**: 줄 순환을 `idx % 2`로 하드코딩. **왜**: 예시 1에서 맞은 걸 보고 예시 2를 손으로 따라가지 않았다. **다음에**: 숫자를 쓰고 싶어질 때 `n`이 달라져도 맞는지 확인한다.
- **틀렸던 지점 2**: `idx++`를 `if` 안에 둠. **왜**: "표가 있을 때만 처리"를 "표가 있을 때만 이동"으로 착각했다. 이동은 조건과 상관없이 항상 해야 한다.
- **틀렸던 지점 3**: 꺼낸 값의 복사본을 줄이고 원본을 기대함. **왜**: `int first = deque.poll()`은 복사본인데 원본이 줄 거라 생각했다. 원본을 바꾸려면 원본의 주소(번호)로 접근해야 한다.
- **다음에 떠올릴 시점**: 큐에 무언가를 넣기 전에 "이걸 꺼냈을 때 이게 누구인지 알 수 있나?"를 먼저 묻는다.
- **마음가짐**: "스택·큐를 꼭 써야 한다는 강박" 때문에 아이디어가 막혔다고 했는데, 이 문제의 문장("줄 맨 뒤로 다시 간다")이 이미 큐의 동작이다. 문제 문장을 먼저 말로 옮기면 자료구조가 따라온다.

## AI라면 어떻게 풀었을까

큐에 "사람 번호"만 넣고 남은 표 수를 `tickets[]`에서 따로 읽는 대신, `{사람 번호, 남은 표 수}` 한 쌍을 `int[]`로 묶어 큐에 넣는다. 풀이 중에 스스로 떠올렸던 "번호와 표 수를 한 쌍으로 들고 다니자"는 생각을 그대로 완성한 형태다.

```java
public int timeRequiredToBuyAlt(int[] tickets, int k) {
    Queue<int[]> queue = new ArrayDeque<>();   // {사람 번호, 남은 표 수}
    for (int i = 0; i < tickets.length; i++) {
        queue.offer(new int[] { i, tickets[i] });
    }

    int time = 0;
    while (!queue.isEmpty()) {
        int[] cur = queue.poll();
        cur[1]--;                               // 표 1장 구매
        time++;
        if (cur[0] == k && cur[1] == 0) {
            return time;
        }
        if (cur[1] > 0) {
            queue.offer(cur);                   // 남았으면 다시 맨 뒤로
        }
    }
    return time;
}
```

트레이드오프: 입력 배열 `tickets`를 건드리지 않고 상태가 큐 안에서 닫힌다는 장점이 있다. 대신 매 사람마다 `int[]` 객체를 만들어서 메모리를 조금 더 쓰고, `cur[0]`, `cur[1]`처럼 인덱스로 읽어서 의미가 덜 드러난다. 이 문제는 n이 최대 100이라 어느 쪽을 써도 충분하다.
