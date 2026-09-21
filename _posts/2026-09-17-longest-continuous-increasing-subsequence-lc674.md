---
title: "[LeetCode-674] nums[i-1] 대신 직전 값을 변수로 기억하기"
date: 2026-09-17 20:20:00 +0900
categories: [알고리즘 연습, 배열]
tags: [java, leetcode, array]
---

- 문제 링크: https://leetcode.com/problems/longest-continuous-increasing-subsequence/
- 파일 경로: `src/main/java/coding_test/배열_리스트/LC0674_LongestContinuousIncreasingSubsequence.java`
- 난이도: Easy

## 문제 설명

정렬되지 않은 정수 배열 `nums`가 주어진다. 값이 계속 커지는(엄격히 증가하는) 연속 구간 중 가장 긴 것의 길이를 반환한다.

```
Input: nums = [1,3,5,4,7]
Output: 3  // [1,3,5]
```

## 시행착오

앞 두 문제(485, 1550)와 같은 "누적 카운터 + 끊기면 리셋 + 최댓값 갱신" 구조의 변형이다. 다만 여기서는 인접한 두 원소를 비교해야 해서, `nums[i-1]`을 직접 참조하는 대신 `current`라는 변수에 직전 값을 저장해두고 그걸 `nums[i]`와 비교하는 방식을 선택했다.

증가 중이면 `raise`를 늘리고, 아니면 `max`를 갱신한 뒤 `raise`를 1로 리셋한다. 초안에서 이 두 갈래(증가/비증가) 코드에 비슷한 줄이 중복돼 있는 걸 다 쓰고 나서 스스로 알아채고 정리했다.

485에서 배운 "루프 종료 후 남은 구간 처리"가 자동으로 떠올라서, 루프가 끝난 뒤에도 `max = Math.max(max, raise);`를 한 번 더 넣어서 배열이 증가로 끝나는 경우를 놓치지 않게 했다.

## 최종 코드

```java
public int findLengthOfLCIS(int[] nums) {
    int raise = 1;
    int max = 1;
    int current = nums[0];
    for (int i = 1; i < nums.length; i++) {
        if (current < nums[i]) {
            raise++;
        } else {
            max = Math.max(raise, max);
            raise = 1;
        }
        current = nums[i];
    }
    max = Math.max(max, raise);
    return max;
}
```

## 오늘 배운 내용

인접한 값을 비교할 때 `nums[i-1]`로 직접 인덱싱하는 것과, 별도 변수에 직전 값을 "기억"해두는 것 두 방식이 있는데, 이번엔 후자를 골랐다. 그리고 485에서 다진 "루프 종료 후 남은 구간 처리" 패턴이 이 문제에서도 그대로 재사용됐다 — 같은 실수를 미리 막았다는 뜻이라, 패턴이 손에 붙기 시작한 신호로 본다.

## 오답노트

- **틀렸던 패턴**: 처음 초안에서 증가/비증가 두 분기에 비슷한 코드가 중복돼 있었음.
- **왜 틀렸나**: 두 분기를 따로 생각하며 짜다 보니, 공통으로 필요한 처리(`current` 갱신 등)가 양쪽에 겹쳐 들어갔다.
- **고친 패턴**: 두 분기가 공유하는 부분(`current = nums[i];`)을 `if/else` 바깥으로 빼서 한 번만 실행되게 정리했다.
- **다음에 떠올릴 시점**: `if/else` 양쪽에 똑같은 줄이 보이면, 그 줄이 조건과 무관하게 항상 실행돼야 하는 코드인지부터 의심한다.

## AI라면 어떻게 풀었을까

485에서 소개한 슬라이딩 윈도우 방식을 그대로 여기에도 적용할 수 있다 — "구간의 시작 인덱스"를 변수로 들고, 증가가 끊기는 순간 시작 인덱스를 현재 위치로 옮기는 방식이다.

```java
int start = 0, max = 1;
for (int i = 1; i < nums.length; i++) {
    if (nums[i - 1] >= nums[i]) {
        start = i;
    }
    max = Math.max(max, i - start + 1);
}
return max;
```

`raise`라는 별도의 카운터 변수 없이, "구간 길이 = 현재 인덱스 - 시작 인덱스 + 1"로 계산하는 게 핵심 차이다. 카운터를 늘렸다 리셋하는 대신 "시작점이 어디였는지"를 기억하는 이 방식은, 나중에 "그 구간의 시작/끝 인덱스 자체가 필요한" 문제(예: 그 구간을 실제로 잘라내야 하는 문제)에서 카운터 방식보다 바로 써먹을 수 있다는 장점이 있다.
