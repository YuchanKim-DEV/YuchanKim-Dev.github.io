---
title: "[LeetCode-410] 판정 함수 안의 인덱스는 그룹이 바뀌어도 이어져야 한다"
date: 2026-09-23 13:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search]
---

> 이진 탐색 개념과 패턴 정리는 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다. [LC875]({% post_url 2026-09-22-koko-eating-bananas-revisit-lc875 %}), [LC1011]({% post_url 2026-09-23-capacity-to-ship-packages-revisit-lc1011 %})에 이은 파라메트릭 서치 세 번째 문제다.

- 문제 링크: https://leetcode.com/problems/split-array-largest-sum/
- 파일 경로: `src/main/java/coding_test/DP_이진탐색/LC0410_SplitArrayLargestSum.java`
- 난이도: Hard

## 문제 설명

정수 배열 `nums`와 정수 `k`가 주어진다. `nums`를 `k`개의 비어있지 않은 연속 부분 배열로 나눌 때, 그 부분 배열들의 합 중 최댓값을 최소로 만드는 값을 구한다.

```
Input: nums = [7,2,5,10,8], k = 2
Output: 18  // [7,2,5]와 [10,8]로 나누면 최댓값이 18
```

폴더 이름은 `DP_이진탐색`이지만 DP 지식 없이도 풀리는 문제다 — 875(먹는 속도), 1011(배 용량)과 똑같이 "이 값이면 가능한가?"를 판정하는 파라메트릭 서치다.

## 시행착오

**판정 함수의 구조부터 잘못 짰다.** 처음엔 이렇게 짰다.

```java
for (int i = 0; i < k; i++) {
    int idx = 0;  // 매번 0부터 다시!
    int sum = 0;
    while (sum <= mid && idx < nums.length) { ... }
}
```

`k`번 반복하는 각 그룹마다 `idx`를 `0`으로 다시 초기화하고 있었다. 이러면 두 번째, 세 번째 그룹이 "이전 그룹이 끝난 지점부터" 이어서 시작하지 못하고 매번 배열 맨 앞부분만 반복해서 본다. 연속된 부분 배열로 나누는 문제니까, 한 그룹이 소비한 위치(`idx`)를 다음 그룹이 이어받아야 하는데 그 연결이 끊겨 있었다. `idx`를 `for` 루프 밖으로 빼서 k번 반복 동안 계속 누적되게 고쳤다.

**판정 기준도 틀려 있었다.** `max >= mid`로 "그룹 합이 mid를 넘었는지" 확인하려 했는데, 애초에 안쪽 `while`이 `nums[idx] + sum > mid`면 멈추도록 짜여 있어서 `sum`은 절대 `mid`를 넘을 수 없다. 즉 이 비교 자체가 의미가 없었다. 진짜 확인해야 하는 건 "k개의 그룹으로 나눴을 때 배열 전체를 다 커버했는가"였다 — `idx != nums.length`로 바꿔서, 남는 원소가 있으면(=k개로는 부족해서) `mid`가 너무 작다고 판단하도록 고쳤다.

**탐색 범위(`left`/`right`)가 뒤바뀌어 있었다.** `left=0`, `right=max(nums)`로 잡았는데, 실제 정답(`18`)은 `max(nums)`(`10`)보다 크기 때문에 애초에 탐색 범위 밖에 있었다. 그룹 하나의 합은 최소 그 안에 든 원소 하나보다는 크거나 같아야 하니 하한은 `max(nums)`, 최악의 경우(전부 한 그룹) 상한은 `sum(nums)`라는 걸 다시 정리해서 고쳤다.

## 최종 코드

```java
public int splitArray(int[] nums, int k) {
    int left = Arrays.stream(nums).max().getAsInt();
    int right = Arrays.stream(nums).sum();
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int idx = 0;
        int sum = 0;

        for (int i = 0; i < k; i++) {
            sum = 0;
            while (sum <= mid && idx < nums.length) {
                if (nums[idx] + sum > mid) {
                    break;
                } else {
                    sum += nums[idx];
                    idx++;
                }
            }
        }

        if (idx != nums.length) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return left;
}
```

## 오늘 배운 내용

**판정 함수가 "실제로 얼마가 나왔는지"를 몰라도, "가능한가/불가능한가"만 정확히 답하면 이진 탐색은 정답을 찾아낸다.** 이 문제를 풀면서 "그룹 합의 실제 최댓값을 안 구해도 정답이 나오는 게 맞나?"라는 의문이 들었는데, 875에서 "실제로 먹는 데 걸린 시간"을 따로 안 구했던 것과 같은 원리였다 — 가능/불가능이 갈리는 경계 자체가 정답이라서, 그 경계를 찾는 이진 탐색만 정확하면 실제 값을 몰라도 된다.

그리고 **"그룹을 나눈다"는 문제에서는, 한 그룹의 처리가 끝난 지점이 다음 그룹의 시작 지점으로 이어져야 한다.** 반복문 안에서 인덱스를 다시 초기화하고 싶어질 때마다, "이게 독립적인 반복인가, 아니면 이전 반복의 결과를 이어받아야 하는가"부터 확인해야 한다는 걸 이번에 제대로 겪었다.

## 오답노트

- **틀렸던 패턴 1**: `for (i=0;i<k;i++)` 안에서 `idx`를 매번 `0`으로 초기화.
- **왜 틀렸나**: k개의 그룹이 배열을 나눠 가지는 거지, k번 똑같은 배열을 독립적으로 보는 게 아니라는 걸 놓쳤다.
- **틀렸던 패턴 2**: `max >= mid`로 판정 — 하지만 `max`는 애초에 `mid`를 넘을 수 없도록 안쪽 로직이 이미 막고 있어서 이 비교가 항상 무의미했다.
- **왜 틀렸나**: "그룹 합이 mid를 넘는가"와 "k개로 배열 전체를 다 커버했는가"가 서로 다른 질문이라는 걸 구분하지 못했다.
- **틀렸던 패턴 3**: `left=0`, `right=max(nums)` — 정답이 탐색 범위 밖에 있었음.
- **왜 틀렸나**: "그룹 하나의 합은 최소한 그 안의 가장 큰 원소보다는 커야 한다"는 하한과 "전부 한 그룹에 몰아넣은 경우"라는 상한을 헷갈렸다.
- **다음에 떠올릴 시점**: "배열을 몇 개의 연속 구간으로 나눈다"는 문제를 보면, 반복문 안에서 인덱스가 각 구간마다 독립적으로 초기화되고 있는지부터 확인한다. 그리고 판정 함수를 짤 때 "이 비교가 항상 참/항상 거짓이 되도록 이미 다른 코드가 막고 있지는 않은가"를 의심한다.

## AI라면 어떻게 풀었을까

판정 로직을 별도 메서드로 뽑고, `left < right` + `right = mid` 템플릿(278에서 소개했던 "즉시 리턴 없음" 템플릿)으로 짜면 이렇게 정리된다.

```java
public int splitArray(int[] nums, int k) {
    int left = Arrays.stream(nums).max().getAsInt();
    int right = Arrays.stream(nums).sum();

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canSplit(nums, k, mid)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

private boolean canSplit(int[] nums, int k, int maxSum) {
    int groups = 1;
    int sum = 0;
    for (int n : nums) {
        if (sum + n > maxSum) {
            groups++;
            sum = 0;
        }
        sum += n;
    }
    return groups <= k;
}
```

`canSplit`은 "이 `maxSum`으로 나누면 최소 몇 그룹이 필요한가"를 한 번의 순회로 세고, 그 그룹 수가 `k` 이하인지만 돌려준다. `idx`나 `for(i<k)` 이중 구조 없이, 그룹 수를 세는 카운터 하나로 끝난다 — "정확히 k개로 나눴을 때 끝까지 커버했는가"를 확인하는 대신 "필요한 최소 그룹 수가 k 이하인가"를 확인하는 것으로 질문을 바꾼 것뿐인데, 구현이 훨씬 단순해진다. 판정 함수를 별도 메서드로 분리하면 이진 탐색 루프 자체도 875/1011 때와 똑같은 모양으로 깔끔하게 남는다.
