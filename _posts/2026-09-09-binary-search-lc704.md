---
title: "[LeetCode-704] left = mid가 무한루프를 만드는 이유 - 이진 탐색 기본형의 탐색 구간 제외 불변식"
date: 2026-09-09 21:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search]
---

- 문제 링크: https://leetcode.com/problems/binary-search/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0704_BinarySearch.java`
- 난이도: Easy

## 이진 탐색이란?

정렬된 데이터에서 값을 찾을 때, 처음부터 하나씩 확인하면(선형 탐색) 최악의 경우 데이터 개수만큼(`O(n)`) 봐야 한다. 이진 탐색은 **"가운데를 한 번 확인해서 절반을 통째로 버린다"**는 전략으로 이걸 `O(log n)`으로 줄인다.

핵심 3문장:
1. 정렬된 데이터에서 가운데(`mid`)를 확인한다.
2. 찾는 값이 `mid`보다 작으면 오른쪽 절반을 통째로 버리고, 크면 왼쪽 절반을 통째로 버린다.
3. 후보가 하나 남거나 없어질 때까지 반복한다.

이번 시리즈는 이 이진 탐색을 실제로 손으로 짜보면서 겪은 시행착오를 기록한 것이다. 앞으로 쭉 이어서 올릴 예정.

## 문제 설명

정렬된 정수 배열 `nums`와 정수 `target`이 주어진다. `target`이 배열에 있으면 그 인덱스를, 없으면 `-1`을 반환한다. O(log n) 시간복잡도로 풀어야 한다.

```
Input: nums = [-1,0,3,5,9,12], target = 9
Output: 4

Input: nums = [-1,0,3,5,9,12], target = 2
Output: -1
```

## 시행착오

이진 탐색을 처음 손으로 짜본 문제. 첫 시도는 이랬다.

```java
int left = 0;
int right = nums.length - 1;
int mid = (left + right) / 2;

while (nums[mid] != target) {
    if (target > nums[mid]) {
        left = mid;                 // 문제: mid를 그대로 남김
        mid = (left + right) / 2;
    } else if (target < nums[mid]) {
        right = mid;                // 문제: mid를 그대로 남김
        mid = (left + right) / 2;
    } else {
        return mid;
    }
}
return -1;
```

`target = 2`(배열에 없는 값)로 실행하면 무한 루프에 빠졌다. 원인은 `left = mid`, `right = mid`로 갱신한 부분 — `mid`를 확인해서 정답이 아니라는 걸 알았으면 다음 탐색 범위에서 `mid`를 반드시 빼야 하는데, 그대로 남겨두니 `left`와 `right`가 더 이상 좁혀지지 않는 상태에서 계속 같은 `mid`를 반복 계산하게 됐다.

그래서 `left = mid + 1`, `right = mid - 1`로 고쳤다. 오버플로우 방지를 위해 `mid` 계산도 `(left+right)/2` 대신 `left + (right-left)/2`로 바꿨다 — 수학적으로 같은 값이지만, `left`와 `right`가 둘 다 `int` 최댓값 근처로 클 경우 `left+right`가 오버플로우로 음수가 될 수 있기 때문.

## 최종 코드

```java
public static int solution(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;
}
```

## 변수 추적표 (target = 9)

`nums = [-1, 0, 3, 5, 9, 12]`

| 반복 | left | right | mid = left+(right-left)/2 | nums[mid] | 비교 | 다음 동작 |
|---|---|---|---|---|---|---|
| 1 | 0 | 5 | 0+(5-0)/2 = **2** | 3 | 9 > 3 | left = mid+1 = 3 |
| 2 | 3 | 5 | 3+(5-3)/2 = **4** | 9 | 9 == 9 | return 4 |

2번의 반복 만에 정답 발견.

## 오늘 배운 내용 (불변식)

이 문제에서 가장 중요한 한 가지는 이거다.

> `mid`를 확인해서 정답이 아니었다면, `mid`는 다음 탐색 구간에서 반드시 제외해야 한다.

이진 탐색이 매 반복마다 탐색 범위를 절반으로 줄이는 이유는 "확인한 `mid`는 다시 보지 않는다"는 이 불변식 덕분이다. `left = mid`, `right = mid`처럼 `mid` 자신을 범위에 남겨두면, 극단적인 경우(`left`와 `right`가 인접) `mid`가 매번 같은 값으로 계산되면서 범위가 전혀 줄어들지 않아 무한 루프에 빠진다.

## 오답노트

- **틀렸던 패턴**: `left = mid`, `right = mid`로 갱신 — 확인이 끝난 `mid`를 탐색 범위에 남겨두는 실수.
- **왜 틀렸나**: `mid`가 정답이 아니라고 판정한 순간, 다음 탐색에서 `mid`를 다시 볼 이유가 없다. 그런데도 범위에 남겨두면 `left`/`right`가 좁혀지지 않는 구간이 생기고, 그 구간에서 `mid`가 계속 같은 값으로 재계산되어 루프가 끝나지 않는다.
- **고친 패턴**: `left = mid + 1`, `right = mid - 1` — 확인이 끝난 `mid`는 항상 다음 범위에서 제외한다.
- **덤으로 고친 것**: `mid` 계산을 `(left+right)/2`가 아니라 `left + (right-left)/2`로 — `left+right`가 `int` 범위를 넘으면 오버플로우로 음수가 될 수 있기 때문.
- **언제 다시 떠올릴까**: 앞으로 어떤 이진 탐색을 짜든, "확인한 `mid`가 다음 범위에 남아있지 않은가"부터 제일 먼저 점검한다.

## 꿀팁

- `mid` 계산은 항상 `left + (right - left) / 2`로 쓰는 습관을 들이자. `(left + right) / 2`는 값이 작을 땐 문제없지만, 코테 문제에 흔히 등장하는 `10^9`대의 큰 입력값에서는 오버플로우를 조용히 만들어낼 수 있다.
- 무한 루프가 의심되면, 반복마다 `left`/`right`/`mid` 값을 출력해서 "이 세 값이 실제로 줄어들고 있는지"부터 눈으로 확인하는 게 제일 빠른 디버깅 방법이었다.
