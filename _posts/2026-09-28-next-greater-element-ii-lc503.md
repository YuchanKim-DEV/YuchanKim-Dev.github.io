---
title: "[LeetCode-503] 원형 배열은 2n번 순회 + %n으로, Map은 (인덱스→값) 순서로"
date: 2026-09-28 15:00:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, monotonic-stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다. [LC496]({% post_url 2026-09-26-next-greater-element-lc496 %}), [LC739]({% post_url 2026-09-27-daily-temperatures-lc739 %})에 이은 몬로토닉 스택 세 번째 문제.

- 문제 링크: https://leetcode.com/problems/next-greater-element-ii/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0503_NextGreaterElementII.java`
- 난이도: Medium

## 문제 설명

원형 정수 배열 `nums`(마지막 원소 다음이 다시 첫 원소로 이어짐)가 주어진다. 각 원소에 대해, 순환해서라도 찾을 수 있는 다음으로 더 큰 값을 반환한다. 없으면 `-1`.

```
Input: nums = [1,2,3,4,3]
Output: [2,3,4,-1,4]
```

## 시행착오

**1. 원형 순회를 어떻게 표현할지부터 막힘.** 배열 끝까지 훑고 나서 다시 처음부터 봐야 하는데, 어떻게 "한 바퀴 더" 돌게 할지 감이 안 왔다. 배열 길이 `n`의 **2배만큼 인덱스를 늘리고, 실제 접근은 `i % n`으로 순환**시키는 방법을 배워서 적용했다.

```java
int len = nums.length;
for (int i = 0; i < 2 * len; i++) {
    int idx = i % len;
    // ...
}
```

**2. 739에서 겪었던 "값으로 구분한 Map" 버그가 그대로 재발함.** 처음엔 `Map<Integer, Integer> nextGreater`에 값을 키로 넣었다. `nums = [3,5,3,4]`처럼 같은 값(`3`)이 두 번 나오는데 정답이 서로 다른 경우(`5`와 `4`)를 테스트하니, 먼저 들어간 값이 나중 값으로 덮어써져서 틀렸다. 739에서 이미 겪었던 것과 똑같은 유형의 실수였다 — 값이 유일하지 않은 배열에서는 값 자체를 키로 못 쓴다는 걸 다시 확인했다. 스택에 값 대신 인덱스를 담는 걸로 바꿨다.

**3. 인덱스로 바꿨는데도 여전히 틀림 — 이번엔 `Map.put()`의 인자 순서가 반대였다.**

```java
nextGreater.put(val, num);  // val: 지금 값, num: 방금 꺼낸 인덱스
```

`put(키, 값)`인데, 저장하고 싶은 건 "그 인덱스의 정답은 이 값이다"였다. 그러려면 키가 인덱스(`num`)이고 값이 정답 값(`val`)이어야 하는데, 순서를 반대로 넣었다. 최종 조회 부분도 `nextGreater.getOrDefault(nums[i], -1)`처럼 값으로 찾고 있었는데, 맵의 키가 인덱스로 바뀌었으니 조회도 `i`(인덱스)로 해야 했다. 두 군데(저장 순서, 조회 기준) 다 인덱스 기준으로 맞추고 나서야 통과했다.

## 최종 코드

```java
public int[] nextGreaterElements(int[] nums) {
    Map<Integer, Integer> nextGreater = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();

    int len = nums.length;
    for (int i = 0; i < 2 * len; i++) {
        int idx = i % len;
        int val = nums[idx];
        while (!stack.isEmpty() && nums[stack.peek()] < nums[idx]) {
            int num = stack.pop();
            nextGreater.put(num, val);
        }
        stack.push(idx);
    }

    int[] ans = new int[nums.length];
    for (int i = 0; i < nums.length; i++) {
        ans[i] = nextGreater.getOrDefault(i, -1);
    }
    return ans;
}
```

## 오늘 배운 내용

**"값으로 구분되지 않는 원소는 인덱스로 구분한다"는 739의 교훈이, 스택뿐 아니라 Map에도 똑같이 적용된다.** 스택에 인덱스를 담기 시작했으면, 그 인덱스를 쓰는 모든 지점(스택 → Map 저장 → Map 조회)이 전부 "인덱스가 기준"이라는 일관성을 유지해야 한다. 중간에 한 군데라도 값 기준으로 되돌아가면(이번엔 `put()`의 키 자리) 그 지점에서 바로 깨진다. 하나의 자료구조 설계 결정(값 대신 인덱스)은 코드 전체에 퍼져 있는 모든 사용처에 일관되게 반영돼야 한다.

## 오답노트

- **틀렸던 패턴 1**: 739와 같은 실수 재발 — `Map<값, 정답>`으로 접근, 중복값에서 덮어써짐.
- **왜 틀렸나**: 바로 전 문제에서 배운 교훈을 새 문제에 적용할 때, "이 문제도 값이 중복될 수 있는가"부터 확인하는 습관이 아직 자동으로 안 나온다.
- **틀렸던 패턴 2**: `nextGreater.put(val, num)` — 키/값이 반대로 들어감.
- **왜 틀렸나**: "인덱스를 키로 써야 한다"는 결정은 했지만, 실제 `put()` 호출에서 어느 변수가 키 자리에 와야 하는지 헷갈렸다.
- **틀렸던 패턴 3**: 최종 조회에서 `nextGreater.getOrDefault(nums[i], -1)`로 여전히 값 기준 조회.
- **왜 틀렸나**: 저장 방식(인덱스 키)을 바꿨는데, 그 변경이 나중에 조회하는 코드까지 이어지지 않았다.
- **다음에 떠올릴 시점**: 스택/맵에 "무엇을 담을지"를 인덱스로 바꾸기로 결정했으면, 그 자료구조를 저장하는 곳과 꺼내 쓰는 곳 **전부**를 한 번씩 점검한다 — 한 곳만 바꾸고 나머지를 그대로 두면 그 지점에서 다시 값/인덱스 혼동이 생긴다.

## AI라면 어떻게 풀었을까

`Map` 없이, 정답 배열(`int[] ans`)에 스택을 도는 즉시 바로 기록하면 한 단계를 줄일 수 있다.

```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] ans = new int[n];
    Arrays.fill(ans, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < 2 * n; i++) {
        int idx = i % n;
        while (!stack.isEmpty() && nums[stack.peek()] < nums[idx]) {
            ans[stack.pop()] = nums[idx];
        }
        if (i < n) {
            stack.push(idx);
        }
    }
    return ans;
}
```

`Map`에 "인덱스 → 정답"을 모아뒀다가 마지막에 한 번 더 `ans` 배열로 옮기는 대신, 스택에서 꺼내는 그 순간 바로 `ans[꺼낸인덱스]`에 적어버리는 것이다. 중간에 `Map`이라는 별도 저장소가 없어지니 "키/값 순서를 헷갈릴 자리" 자체가 사라진다. 또 하나, `if (i < n) stack.push(idx)`로 **두 번째 바퀴에서는 새로 push하지 않는다** — 이미 한 바퀴를 돌면서 기회를 다 준 원소를 또 스택에 쌓을 필요가 없어서, 불필요한 중복 push를 막는 최적화다.
