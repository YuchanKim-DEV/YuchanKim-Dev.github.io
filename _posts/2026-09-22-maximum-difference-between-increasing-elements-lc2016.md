---
title: "[LeetCode-2016] 브루트포스로 먼저 통과시키고 나서 최적화를 생각한다"
date: 2026-09-22 10:00:00 +0900
categories: [알고리즘 연습, 배열]
tags: [java, leetcode, array]
---

- 문제 링크: https://leetcode.com/problems/maximum-difference-between-increasing-elements/
- 파일 경로: `src/main/java/coding_test/배열_리스트/LC2016_MaximumDifferenceBetweenIncreasingElements.java`
- 난이도: Easy

## 문제 설명

크기 n인 정수 배열 `nums`가 주어진다. `0 <= i < j < n`이고 `nums[i] < nums[j]`를 만족하는 i, j 중에서 `nums[j] - nums[i]`의 최댓값을 구한다. 그런 쌍이 없으면 `-1`을 반환한다.

```
Input: nums = [7,1,5,4]
Output: 4  // i=1, j=2: 5-1=4
```

## 시행착오

이번엔 힌트 없이 바로 통과했다. `i < j`이고 `nums[i] < nums[j]`인 모든 쌍을 이중 반복문으로 전부 확인하면서, 그중 차이가 가장 큰 값을 `ans`에 갱신하는 브루트포스로 접근했다. 조건을 만족하는 쌍이 하나도 없을 수 있다는 것도 문제에 나와 있어서, `checker`라는 불리언을 하나 두고 조건을 만족하는 쌍을 한 번이라도 찾으면 `true`로 바꿔서, 끝까지 한 번도 못 찾았으면 `-1`을 반환하도록 짰다.

## 최종 코드

```java
public int maximumDifference(int[] nums) {
    int ans = 0;
    boolean checker = false;
    for (int i = 0; i < nums.length - 1; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] < nums[j]) {
                checker = true;
                ans = Math.max(ans, nums[j] - nums[i]);
            }
        }
    }
    return !checker ? -1 : ans;
}
```

## 오늘 배운 내용

**조건을 만족하는 경우가 "존재하지 않을 수도 있는" 문제에서는, 값 자체(`ans`)와 "찾았는지 여부"를 분리해서 관리하는 게 안전하다.** `ans`의 초기값을 `-1`로 두고 그걸로 존재 여부까지 겸사겸사 판단하려 했다면, 실제 정답이 우연히 `-1`이 될 수 있는 문제에서는 헷갈릴 수 있다(이 문제는 차이값이라 `-1`이 정답이 될 일은 없지만, `checker`처럼 "찾았는가"를 별도 변수로 분리해두면 그런 문제에서도 안전하게 재사용할 수 있는 패턴이 된다).

## 오답노트

이번엔 틀린 게 없었다. 조건(존재하지 않는 경우 처리)을 코드를 짜기 전에 먼저 챙기고 시작한 게 한 번에 통과한 이유였다.

## AI라면 어떻게 풀었을까

지금 코드는 O(n²)로 모든 쌍을 다 확인한다. 이 문제는 "지금까지의 최솟값"만 기억하면서 한 번만 순회해도 O(n)에 풀린다 — `nums[j]`가 최댓값 후보가 되려면, 그 이전에 나온 값들 중 최솟값과의 차이만 확인하면 되기 때문이다(더 큰 이전 값과 짝지어봐야 차이가 더 작아질 뿐이다).

```java
public int maximumDifference(int[] nums) {
    int minSoFar = nums[0];
    int maxDiff = -1;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] > minSoFar) {
            maxDiff = Math.max(maxDiff, nums[j] - minSoFar);
        } else {
            minSoFar = nums[j];
        }
    }
    return maxDiff;
}
```

`checker` 플래그 없이도 `maxDiff`의 초기값을 `-1`로 두는 것만으로 "못 찾은 경우"가 자연스럽게 표현된다 — 이 문제는 정답이 항상 양수(차이값)이기 때문에 `-1`이 "값"과 "존재하지 않음"을 동시에 표현해도 안전하다. 브루트포스 버전에서 굳이 `checker`를 따로 둔 이유(정답과 초기값이 겹칠 수 있는 상황을 피하려는 조심성)가, 이 문제 특성상 사실은 필요 없었다는 것도 함께 알아두면 좋다. 핵심은 이중 루프의 "이전 원소들"이라는 정보가 사실은 "그중 최솟값 하나"로 압축된다는 것 — n²을 n으로 줄이는 전형적인 패턴이다.
