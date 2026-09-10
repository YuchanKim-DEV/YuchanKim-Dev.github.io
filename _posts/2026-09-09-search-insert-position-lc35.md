---
title: "이진 탐색 루프는 언제 끝날까? left == right가 아니라 left > right였다 (LeetCode 35)"
date: 2026-09-09 21:00:00 +0900
categories: [코딩테스트, 이진탐색]
tags: [java, leetcode, binary-search]
---

- 문제 링크: https://leetcode.com/problems/search-insert-position/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0035_SearchInsertPosition.java`
- 난이도: Easy

## 문제 설명

정렬된 서로 다른 정수 배열 `nums`와 `target`이 주어진다. `target`이 배열에 있으면 인덱스를, 없으면 정렬 순서를 유지하며 삽입될 위치를 반환한다.

```
Input: nums = [1,3,5,6], target = 5   -> Output: 2
Input: nums = [1,3,5,6], target = 2   -> Output: 1
Input: nums = [1,3,5,6], target = 7   -> Output: 4
```

## 시행착오

704 템플릿을 그대로 가져와서 짜고, 루프 밖에서 뭘 반환할지가 문제였다. 첫 시도:

```java
return mid + 1;
```

`target = 7`(모든 값보다 큼)에서는 우연히 맞았지만, `target = 0`(모든 값보다 작음)으로 테스트하니 `1`이 나왔다. 정답은 `0`이어야 하는데.

원인을 찾다가 "루프가 언제 끝나는가"부터 잘못 알고 있었다는 걸 깨달았다. 처음엔 `left == right`일 때 끝난다고 생각했는데, 실제 루프 조건은 `while (left <= right)`라 **`left > right`가 됐을 때** 끝난다. `target = 0` 케이스를 손으로 추적해보면 `right`가 계속 줄어들면서 `left`는 그대로 `0`에 머무는데, `mid + 1`은 마지막 `mid`(0) + 1 = 1을 반환해서 틀린 것이었다.

## 깨달은 것

루프가 끝나는 순간 (`left > right`), **`left`가 정확히 삽입 위치를 가리킨다.** `nums[mid] < target`이었던 마지막 갱신은 `left = mid+1`이 되고, `nums[mid] > target`이었던 마지막 갱신은 `right = mid-1`이 되는데 — 두 경우 모두 `left`가 최종적으로 "target보다 큰 첫 원소의 위치"를 가리키게 된다.

## 최종 코드

```java
public static int solution(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int mid = 0;
    while (left <= right) {
        mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return left;
}
```

## 변수 추적표 (target = 7, 못 찾는 케이스)

`nums = [1, 3, 5, 6]`

| 반복 | left(시작) | right(시작) | mid | nums[mid] | 비교 | 갱신 |
|---|---|---|---|---|---|---|
| 1 | 0 | 3 | 0+(3-0)/2=**1** | 3 | 3 < 7 | left = 2 |
| 2 | 2 | 3 | 2+(3-2)/2=**2** | 5 | 5 < 7 | left = 3 |
| 3 | 3 | 3 | 3+(3-3)/2=**3** | 6 | 6 < 7 | left = 4 |
| — | 4 | 3 | `left(4) <= right(3)`? 거짓 → 종료 | | | |

루프 종료 시 `left = 4`, `right = 3` → `return left` = `4` (정답)

## 한 줄 오답노트

> 루프는 `left == right`가 아니라 `left > right`일 때 끝난다. 이 착각 하나 때문에 답이 완전히 틀어질 수 있다.
