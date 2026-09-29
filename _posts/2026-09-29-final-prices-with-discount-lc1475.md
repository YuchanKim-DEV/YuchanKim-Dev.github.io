---
title: "[LeetCode-1475] 이번엔 처음부터 인덱스를 키로 — 반복된 실수가 안 반복됨"
date: 2026-09-29 14:00:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, monotonic-stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다. [LC739 두 번째 백지 재작성]({% post_url 2026-09-28-daily-temperatures-revisit-lc739 %})에서 나온 체크포인트를 다지려고 고른, 몬로토닉 스택 계열의 비슷한 문제.

- 문제 링크: https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC1475_FinalPricesWithASpecialDiscountInAShop.java`
- 난이도: Easy

## 문제 설명

가격 배열 `prices`가 주어진다. `i`번째 상품을 사면, `j > i`이고 `prices[j] <= prices[i]`를 만족하는 가장 가까운 `j`를 찾아서 그 가격만큼 할인받는다. 그런 `j`가 없으면 할인은 없다. 할인 반영한 최종 가격 배열을 반환한다.

```
Input: prices = [8,4,6,2,3]
Output: [4,2,4,2,3]
```

## 시행착오

739, 503에서 반복했던 "값을 키로 쓴 Map" 실수를 이번엔 처음부터 피했다 — 스택에 **인덱스**를 담고, `Map`도 인덱스를 키로 쓰는 걸로 바로 짰다.

```java
while (!stack.isEmpty() && prices[stack.peek()] >= prices[i]) {
    map.put(stack.peek(), prices[stack.pop()] - prices[i]);
}
stack.push(i);
```

`stack.peek()`으로 먼저 확인하고, 조건이 맞을 때만 `stack.pop()`하는 것도 739 두 번째 재작성에서 배운 대로 바로 적용했다. 유일하게 막힌 부분은 `map.get(...)`을 쓰다가 인자를 안 채워서 컴파일 에러가 난 것 정도였고, 그것도 바로 고쳤다.

## 최종 코드

```java
public int[] finalPrices(int[] prices) {
    Map<Integer, Integer> map = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();
    int[] ans = new int[prices.length];

    for (int i = 0; i < prices.length; i++) {
        while (!stack.isEmpty() && prices[stack.peek()] >= prices[i]) {
            map.put(stack.peek(), prices[stack.pop()] - prices[i]);
        }
        stack.push(i);
    }

    for (int j = 0; j < prices.length; j++) {
        ans[j] = map.getOrDefault(j, prices[j]);
    }
    return ans;
}
```

## 오늘 배운 내용

**739와 503에서 두 번 반복했던 실수(값을 키로 쓰기, 조건 확인과 제거를 한 줄에 섞기)가 이번엔 아예 나오지 않았다.** 지난 며칠 동안 같은 계열 문제를 여러 번 겪으면서, "값이 중복될 수 있으면 인덱스로 구분한다"와 "peek으로 확인, 조건 통과 시에만 pop"이 이제 시작할 때부터 자연스럽게 나오는 체크리스트가 됐다는 뜻이다. 감독관이 "체크리스트가 아직 자동화 안 됐다"고 진단했던 부분이, 다른 문제(1475)로 넘어와서도 재현되는지가 이 문제의 진짜 검증 포인트였는데, 재현됐다.

## 오답노트

이번엔 로직 자체에서 틀린 게 없었다. `map.get(...)`에 인자를 안 넣어서 생긴 문법 오류 하나뿐이었고, 이건 로직 이해와는 무관한 단순 실수였다.

## AI라면 어떻게 풀었을까

지금 코드는 `Map`에 "인덱스 → 할인액"을 모아뒀다가 마지막에 한 번 더 순회해서 `ans` 배열을 채운다. 739의 AI 대안에서 했던 것처럼, `Map` 없이 `ans` 배열에 바로 기록하면 한 단계를 줄일 수 있다.

```java
public int[] finalPrices(int[] prices) {
    int[] ans = prices.clone();
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < prices.length; i++) {
        while (!stack.isEmpty() && prices[stack.peek()] >= prices[i]) {
            int idx = stack.pop();
            ans[idx] = prices[idx] - prices[i];
        }
        stack.push(i);
    }
    return ans;
}
```

`ans`를 `prices.clone()`으로 시작하면 "할인을 못 받은 경우 원래 가격 그대로"라는 기본값이 저절로 채워지고, 할인받는 경우에만 그 자리를 덮어쓰면 된다. `Map`이라는 중간 저장소가 사라지니 코드도 짧아지고, "이 문제가 인덱스 기반이라는 걸 기억하고 있는가"를 매번 다시 확인할 필요도 없어진다 — 애초에 배열 인덱스 자체를 그대로 쓰기 때문이다.
