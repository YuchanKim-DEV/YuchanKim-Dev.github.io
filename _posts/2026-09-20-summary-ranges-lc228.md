---
title: "[LeetCode-228] 마지막 구간이 3개 이상 이어질 때만 터지는 경계 조건"
date: 2026-09-20 11:00:00 +0900
categories: [알고리즘 연습, 배열]
tags: [java, leetcode, array]
---

- 문제 링크: https://leetcode.com/problems/summary-ranges/
- 파일 경로: `src/main/java/coding_test/배열_리스트/LC0228_SummaryRanges.java`
- 난이도: Easy

## 문제 설명

오름차순으로 정렬되어 있고 중복이 없는 정수 배열 `nums`가 주어진다. 배열의 모든 숫자를 빠짐없이 덮는, 가장 적은 개수의 구간 목록을 반환한다. 구간 `[a,b]`는 `a != b`면 `"a->b"`, `a == b`면 `"a"`로 표현한다.

```
Input: nums = [0,1,2,4,5,7]
Output: ["0->2","4->5","7"]
```

## 시행착오

정렬된 배열을 한 번 순회하면서, 이전 값과 현재 값의 차이가 1이면 "계속 이어지는 중"으로 보고 `count` 플래그를 켜두고, 차이가 1이 아니면 그 지점에서 구간이 끊긴 것으로 보고 지금까지의 구간을 문자열로 만들어 리스트에 추가하는 구조로 짰다.

주어진 예제들은 다 통과했는데, `{1,3,4,5,6,10,11,12}` 같은 입력에서 실패했다 — 기대값은 `["1", "3->6", "10->12"]`인데 마지막 `"10->12"`가 통째로 빠졌다.

원인은 루프가 끝난 뒤 "아직 리스트에 안 들어간 마지막 구간"을 처리하는 조건문에 있었다. 그 조건문이 `list.isEmpty()`이거나, 마지막 구간의 차이가 정확히 `0`이거나 `1`인 경우만 다루고 있었다 — 즉 마지막 구간이 원소 1개나 2개일 때만 처리하고, **이미 앞에서 다른 구간이 리스트에 들어간 상태에서, 마지막 구간이 3개 이상으로 길게 이어지는 경우(차이가 2 이상)** 를 통째로 빠뜨리고 있었다. 조건을 `차이 == 1`에서 `차이 >= 1`로 바꾸니 해결됐다.

## 최종 코드

```java
public List<String> summaryRanges(int[] nums) {
    ArrayList<String> list = new ArrayList<>();

    if (nums.length == 0) {
        return list;
    } else if (nums.length == 1) {
        list.add("" + nums[0]);
        return list;
    }

    int initial = nums[0];
    boolean count = false;
    for (int i = 1; i < nums.length; i++) {
        int prev = nums[i - 1];
        if (nums[i] - prev != 1) {
            if (!count) {
                list.add("" + initial);
            } else {
                list.add("" + initial + "->" + nums[i - 1]);
            }
            initial = nums[i];
            count = false;
        } else {
            count = true;
        }
    }

    if (nums.length > 0 && list.isEmpty()) {
        list.add("" + initial + "->" + nums[nums.length - 1]);
    } else if (nums[nums.length - 1] - initial == 0) {
        list.add("" + initial);
    } else if (nums[nums.length - 1] - initial >= 1) {
        list.add("" + initial + "->" + nums[nums.length - 1]);
    }

    return list;
}
```

## 오늘 배운 내용

**루프 종료 후 "남은 마지막 구간"을 처리할 때는, 그 구간의 길이가 몇 개까지 가능한지부터 전부 따져야 한다.** 원소 1개(차이 0), 2개(차이 1)만 생각하고 3개 이상(차이 2 이상)인 경우를 빠뜨리면, 준비된 예제는 다 통과해도 특정 입력에서만 조용히 실패한다. "몇 개일 수 있는가"를 임의의 개수로 상정하고 조건을 짜야 안전하다.

## 오답노트

- **틀렸던 패턴**: 마지막 구간 처리 조건을 `차이 == 1`로만 걸어서, 3개 이상 이어지는 마지막 구간(차이 >= 2)을 놓침.
- **왜 틀렸나**: 마지막 구간의 길이가 딱 1개 아니면 2개일 거라고 무의식적으로 단정했다. 실제로는 배열 끝까지 임의 개수로 이어질 수 있다.
- **고친 패턴**: `차이 == 1`을 `차이 >= 1`로 바꿔서, 1개 이상 이어지는 모든 경우를 하나의 조건으로 커버.
- **다음에 떠올릴 시점**: 루프 종료 후 남은 상태를 처리하는 코드를 짤 때, "이 값이 딱 1개나 2개일 수만 있는가, 아니면 임의 개수일 수 있는가"부터 먼저 확인한다.
