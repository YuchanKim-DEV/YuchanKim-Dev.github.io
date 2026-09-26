---
title: "[LeetCode-496] '다음 값 하나'가 아니라 '더 큰 값을 만날 때까지'다"
date: 2026-09-26 20:00:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, monotonic-stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/next-greater-element-i/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0496_NextGreaterElementI.java`
- 난이도: Easy

## 문제 설명

`nums1`은 `nums2`의 부분집합이다. `nums1`의 각 원소에 대해, 그 값이 `nums2`에서 등장하는 위치를 찾고, 그 오른쪽에 있는 원소 중 자기보다 큰 첫 번째 값을 찾는다. 없으면 `-1`.

```
Input: nums1 = [4,1,2], nums2 = [1,3,4,2]
Output: [-1,3,-1]
```

## 시행착오

`nums2`를 `Deque`에 다 담아두고, `nums1`의 각 원소마다 그 덱을 **통째로 복제**해서 값을 찾는 방식으로 접근했다. 복제한 덱에서 `nums1[j]`가 나올 때까지 계속 꺼내고, 그다음부터가 "오른쪽 나머지"가 된다.

여기서 처음엔 이렇게 짰다 — `nums1[j]`를 찾은 다음, **바로 다음 값 하나만** 확인해서 그게 더 크면 답으로, 아니면 무조건 `-1`로 처리했다.

```java
pollNum = clone.pollFirst();
if (pollNum > nums1[j]) {
    ans[j] = pollNum;
} else {
    ans[j] = -1;
}
```

`nums1=[1,3,5,2,4]`, `nums2=[6,5,4,3,2,1,7]`로 제출했더니 `[7,-1,-1,-1,-1]`이 나왔는데 기대값은 `[7,7,7,7,7]`이었다. `3`의 경우를 보면, `3` 다음 값은 `2`(더 작음)라서 바로 `-1`로 포기했는데, 실제로는 `2`, `1`을 지나 그다음에 있는 `7`까지 봐야 했다 — "바로 다음 값"이 아니라 "더 큰 값을 만날 때까지 계속 찾아야" 하는 문제였다. `if`로 한 번만 확인하던 걸 `while`로 바꿔서, 더 큰 값을 만나거나 덱이 빌 때까지 계속 훑도록 고쳤다.

## 최종 코드

```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Deque<Integer> queue = new ArrayDeque<>();
    int[] ans = new int[nums1.length];

    for (int n : nums2) {
        queue.offer(n);
    }

    for (int j = 0; j < nums1.length; j++) {
        Deque<Integer> clone = new ArrayDeque<>(queue);
        int pollNum = clone.pollFirst();
        while (nums1[j] != pollNum) {
            pollNum = clone.pollFirst();
        }

        if (clone.isEmpty()) {
            ans[j] = -1;
        } else {
            pollNum = clone.pollFirst();
            while (!(pollNum > nums1[j]) && !clone.isEmpty()) {
                pollNum = clone.pollFirst();
            }
            ans[j] = (pollNum > nums1[j]) ? pollNum : -1;
        }
    }

    return ans;
}
```

## 오늘 배운 내용

**"다음"이라는 말이 나오면 "바로 다음 원소 하나"인지 "조건을 만족할 때까지 계속되는 다음"인지부터 구분해야 한다.** 이번 문제의 "next greater element"는 후자였다 — 조건(더 큰 값)을 만족하는 걸 찾을 때까지 계속 나아가야 하는데, 처음엔 이걸 "한 칸 다음"으로 좁게 해석했다. 반복문으로 "계속 찾는다"를 표현해야 하는 상황을, `if` 한 번으로 "확인하고 끝"으로 짜면 딱 한 걸음만 보고 판단이 끝나버린다.

## 오답노트

- **틀렸던 패턴**: `nums1[j]`를 찾은 바로 다음 값 하나만 `if`로 확인하고 끝냄.
- **왜 틀렸나**: "다음으로 더 큰 값"을 "바로 다음 값이 더 크다면"으로 좁게 해석했다. 사이에 더 작은 값들이 여러 개 끼어있을 수 있다는 걸 고려 안 했다.
- **고친 패턴**: `if`를 `while`로 바꿔서, 더 큰 값을 찾거나 남은 원소가 없을 때까지 계속 반복.
- **다음에 떠올릴 시점**: "다음/이후에 나오는 ~"라는 표현을 문제에서 보면, 그게 "정확히 한 칸 뒤"인지 "조건을 만족하는 가장 가까운 것"인지부터 구분한다. 후자라면 `if` 한 번이 아니라 반복문(또는 반복 가능한 구조)이 필요하다.

## AI라면 어떻게 풀었을까

지금 방식은 `nums1`의 원소마다 `nums2`를 복제해서 처음부터 다시 훑기 때문에 O(n·m)이다. 이 문제의 정석 풀이는 **몬로토닉 스택**을 한 번만 써서 O(n+m)에 끝내는 것이다.

```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Map<Integer, Integer> nextGreater = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();

    for (int n : nums2) {
        while (!stack.isEmpty() && stack.peek() < n) {
            nextGreater.put(stack.pop(), n);
        }
        stack.push(n);
    }

    int[] ans = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++) {
        ans[i] = nextGreater.getOrDefault(nums1[i], -1);
    }
    return ans;
}
```

`nums2`를 왼쪽에서 오른쪽으로 딱 한 번 훑으면서, 스택에 "아직 자기보다 큰 값을 못 만난 원소들"을 쌓아둔다. 새 값이 스택 맨 위보다 크면, 그건 스택에 쌓여있던 값들의 "다음으로 큰 값"이 확정된 순간이라 바로 꺼내서 `Map`에 기록하고 계속 비교한다. 지금 코드처럼 매번 "나올 때까지 다시 찾는" 대신, **한 번 지나가면서 스택에 쌓인 "아직 답을 못 찾은 값들"을 그때그때 해소**하는 방식이라 훨씬 빠르다. 이게 몬로토닉 스택 패턴의 핵심이고, 739(Daily Temperatures)에서도 똑같은 구조가 쓰인다.
