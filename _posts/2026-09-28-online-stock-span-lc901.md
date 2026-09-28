---
title: "[LeetCode-901] 메서드가 반복 호출될 때 상태를 기억하려면 클래스 필드로"
date: 2026-09-28 17:20:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, monotonic-stack, design]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다. [LC496]({% post_url 2026-09-26-next-greater-element-lc496 %}), [LC739]({% post_url 2026-09-27-daily-temperatures-lc739 %}), [LC503]({% post_url 2026-09-28-next-greater-element-ii-lc503 %})에 이은 몬로토닉 스택 네 번째 문제.

- 문제 링크: https://leetcode.com/problems/online-stock-span/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0901_OnlineStockSpan.java`
- 난이도: Medium

## 문제 설명

`next(price)`가 호출될 때마다 "오늘"이라는 새 날짜가 스트림으로 추가된다. 오늘부터 과거로 거슬러 올라가면서, 오늘 가격 이하였던 날이 며칠 연속으로 이어지는지(span)를 반환한다.

```
next(100) -> 1
next(80)  -> 1
next(60)  -> 1
next(70)  -> 2
next(60)  -> 1
next(75)  -> 4
next(85)  -> 6
```

## 시행착오

지금까지 풀었던 496/739/503은 전부 "배열 하나가 통째로 주어지고, 그 안에서 계산"하는 문제였는데, 901은 처음 만난 **"메서드가 여러 번 반복 호출되는" 설계(Design) 문제**였다.

처음엔 "지금 인덱스에서 과거로, 맨 처음이면 맨 마지막으로 순환하며 세는 건가?"라고 오해했다 — 503(원형 배열)과 헷갈린 것이었다. 901은 순환이 전혀 없고, 그냥 과거로 거슬러 올라가다 배열(스트림) 시작 지점에서 멈추는 게 전부였다.

그다음 막힌 건 **"next()가 반복 호출될 때 이전 호출의 정보를 어떻게 기억하고 있지?"**였다. 매번 새 배열이 주어지는 게 아니라 `next()` 호출 자체가 계속 이어지는 구조라, "스택을 어디에 선언해야 하는가"부터 헷갈렸다. `next()` 메서드 안에서 `new ArrayDeque<>()`로 매번 새로 만들면 호출할 때마다 텅 빈 스택이 되어 이전 날짜 정보가 다 날아간다는 걸 깨닫고, 스택을 **클래스 필드(인스턴스 변수)**로 선언해서 생성자에서 한 번만 초기화하도록 고쳤다. 그러면 같은 `StockSpanner` 객체로 `next()`를 몇 번을 불러도 그 필드는 계속 살아있다.

구조를 잡은 뒤 로직 자체는 매끄럽게 짰다 — 스택에 `(가격, span)` 쌍을 담아서, 오늘 가격이 스택 맨 위 가격 이상이면 꺼내면서 그 span을 누적하는 방식.

## 최종 코드

```java
public class LC0901_OnlineStockSpan {
    private Deque<int[]> stack;

    public LC0901_OnlineStockSpan() {
        stack = new ArrayDeque<>();
    }

    public int next(int price) {
        int count = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            count += stack.pop()[1];
        }
        stack.push(new int[] { price, count });
        return count;
    }
}
```

## 오늘 배운 내용

**메서드가 여러 번 호출되면서 이전 호출의 결과를 기억해야 하는 "설계(Design)" 문제는, 지역 변수가 아니라 클래스 필드에 상태를 둬야 한다.** 지금까지 풀었던 문제들은 전부 메서드 하나가 한 번 호출되고 끝나는 구조라 이 개념이 필요 없었는데, 901에서 처음으로 "여러 번의 호출에 걸쳐 유지되는 상태"를 다뤘다. 그리고 겉보기에 비슷해 보이는 문제(503의 원형 순회 vs 901의 스트리밍)라도, "정말 순환이 있는가"부터 문제 설명을 다시 정확히 읽고 확인해야 한다는 것도 다시 느꼈다.

## 오답노트

- **틀렸던 패턴**: 503(원형 배열)과 헷갈려서, 901도 "맨 처음까지 가면 맨 마지막으로 순환한다"고 착각.
- **왜 틀렸나**: 최근 푼 문제의 패턴을 다음 문제에 그대로 투영했다. 문제 설명에 "순환"이라는 말이 없는데도 있다고 가정했다.
- **틀렸던 패턴 2**: 스택을 `next()` 메서드 안에서 매번 새로 생성하려 함.
- **왜 틀렸나**: "메서드가 반복 호출되는 구조"에서 상태를 어디에 저장해야 하는지(지역 변수 vs 클래스 필드)를 처음 다뤄봐서 개념 자체가 없었다.
- **다음에 떠올릴 시점**: `next()`, `push()`, `pop()`처럼 여러 번 호출되는 이름의 메서드를 가진 클래스를 설계할 때는, "이 메서드가 이전 호출의 결과를 알아야 하는가"부터 확인하고, 그렇다면 그 정보는 반드시 클래스 필드에 있어야 한다.

## AI라면 어떻게 풀었을까

`(가격, span)` 쌍 대신, 503/739처럼 **인덱스**를 저장하는 방식으로도 짤 수 있다 — "오늘이 몇 번째 날인지"를 직접 세면서, span을 인덱스 차이로 계산하는 것이다.

```java
public class LC0901_OnlineStockSpan {
    private Deque<int[]> stack; // [인덱스, 가격]
    private int day = -1;

    public LC0901_OnlineStockSpan() {
        stack = new ArrayDeque<>();
    }

    public int next(int price) {
        day++;
        while (!stack.isEmpty() && stack.peek()[1] <= price) {
            stack.pop();
        }
        int span = stack.isEmpty() ? day + 1 : day - stack.peek()[0];
        stack.push(new int[] { day, price });
        return span;
    }
}
```

지금 코드는 "span 값 자체"를 스택에 들고 다니며 누적하고, 이 버전은 "몇 번째 날이었는지"를 들고 다니다가 필요한 순간 뺄셈으로 span을 계산한다. 496/739/503에서 일관되게 써온 "인덱스 기반" 접근을 901에도 그대로 적용한 셈이라, 몬로토닉 스택 문제 전체를 하나의 패턴("값 대신 위치를 기억하고, 필요할 때 위치 차이를 계산")으로 묶어볼 수 있다.
