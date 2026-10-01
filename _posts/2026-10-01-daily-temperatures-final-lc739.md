---
title: "[LeetCode-739] 백지 재작성 3차 - 드디어 힌트 없이"
date: 2026-10-01 11:00:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, monotonic-stack]
---

> [1차]({% post_url 2026-09-27-daily-temperatures-lc739 %}), [2차]({% post_url 2026-09-28-daily-temperatures-revisit-lc739 %}) 백지 재작성에 이은 세 번째이자 마지막 검증 글이다. 9/28부터 739를 기준으로 몬로토닉 스택 체크리스트가 자동화됐는지 확인해왔다.

- 문제 링크: https://leetcode.com/problems/daily-temperatures/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0739_DailyTemperatures.java`
- 난이도: Medium

## 문제 설명

(자세한 문제 설명은 [1차 글]({% post_url 2026-09-27-daily-temperatures-lc739 %}) 참고)

## 이번엔 무엇이 달랐나

기존 코드를 완전히 지우고 다시 짰는데, **이번엔 힌트를 하나도 받지 않고 통과했다.** 그동안 두 번의 재작성에서 걸렸던 지점들을 전부 스스로 피해 갔다.

```java
public int[] dailyTemperatures(int[] temperatures) {
    Deque<Integer> stack = new ArrayDeque<>();
    int[] ans = new int[temperatures.length];

    for (int i = 0; i < temperatures.length; i++) {
        while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
            int days = i - stack.peek();
            ans[stack.pop()] = days;
        }
        stack.push(i);
    }
    return ans;
}
```

1차에서 걸렸던 "값을 키로 쓴 Map" 문제는 애초에 Map을 쓰지 않고 배열(`ans`)에 직접 기록하는 방식으로 설계해서 발생할 여지가 없었다.

2차에서 걸렸던 두 가지도 이번엔 자연스럽게 나왔다:
- **조건 확인과 실제 제거를 분리** — `while` 조건에서 `stack.peek()`으로만 확인하고, 조건을 통과한 뒤에야 `stack.pop()`을 호출했다.
- **카운터 대신 실제 값 활용** — `days`를 계산할 때 `stack.peek()`(아직 꺼내기 전의 인덱스)으로 먼저 날짜 차이를 구하고, 그다음 `stack.pop()`으로 그 인덱스를 꺼내서 `ans[]`에 바로 쓰는 한 줄(`ans[stack.pop()] = days;`)로 처리했다. 별도의 `idx` 카운터도, `poppedIdx`를 따로 저장하는 변수도 필요 없었다.

## 오늘 배운 내용

**반복된 백지 재작성이 실제로 체크리스트를 자동화시킨다는 걸 직접 확인했다.** 9/27(1차)과 9/28(2차)에서 각각 다른 지점에 걸렸던 게, 9/28 저녁부터 10/1 사이에 비슷한 계열의 Easy 문제 5개(1475/682/844/1047/933)를 더 풀면서 "스택에서 조건 확인과 pop을 분리한다", "꺼낸 값 자체를 바로 활용한다"는 감각이 여러 문제에 걸쳐 반복 적용됐고, 그 결과가 오늘 739 3차 시도에서 합쳐져 나타났다. 하나의 문제를 반복해서만 익히는 게 아니라, 같은 계열의 여러 문제를 거치면서 패턴이 더 단단해진다는 걸 보여주는 사례다.

## 오답노트

이번엔 틀린 게 없었다. 9/27~9/28에 반복됐던 두 가지 실수(값 기반 Map, 조건 안 pop 부작용, 카운터와 실제 인덱스 혼동)가 전부 처음부터 안 나왔다.

## 감독관 메모

9/28에 "코드를 짤 때 확인해야 할 체크리스트가 아직 자동화되지 않았다"고 진단했던 게, 오늘(10/1) 3차 시도에서 자동화된 것으로 확인됐다. 9/28 밤에 739를 더 반복하는 대신 비슷한 Easy 5문제를 풀자고 제안했던 유저의 판단이 맞았다 — 같은 문제를 반복하는 것보다 같은 패턴의 여러 문제를 거치는 게 체크리스트를 더 빨리 자동화시켰다. 이걸로 739를 기준으로 한 몬로토닉 스택 검증은 마무리하고, 다음 Medium(`LC0962_MaximumWidthRamp`)으로 넘어간다.
