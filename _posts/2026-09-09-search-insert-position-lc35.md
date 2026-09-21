---
title: "[LeetCode-35] 이진 탐색 종료 조건은 left == right가 아니라 left > right다"
date: 2026-09-09 21:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
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

## 오늘 배운 내용

이 문제에서 가장 중요한 한 가지는, 루프가 끝나는 정확한 시점이다.

> 루프가 끝나는 순간(`left > right`), **`left`가 정확히 삽입 위치를 가리킨다.**

`nums[mid] < target`이었던 마지막 갱신은 `left = mid+1`이 되고, `nums[mid] > target`이었던 마지막 갱신은 `right = mid-1`이 되는데 — 두 경우 모두 `left`가 최종적으로 "target보다 큰 첫 원소의 위치"를 가리키게 된다. 즉 종료 조건과 반환값은 세트로 외워야 한다: `left <= right`로 도는 이진 탐색은 종료 후 `left`가 답이 되는 경우가 많다.

## 오답노트

- **틀렸던 생각**: "루프는 `left == right`일 때 끝난다"고 착각했다. 실제 종료 조건은 루프 조건(`left <= right`)의 부정, 즉 `left > right`다.
- **왜 이 착각이 위험한가**: `left == right`를 기준으로 생각하면, 답이 배열의 양 끝을 벗어나는 케이스(모든 원소보다 작거나 큰 target)에서 `left`와 `right`가 아예 교차해버리는 상황을 놓치게 된다. 이번 문제처럼 `target = 0`인 경우 `right`는 `-1`까지 내려가고 `left`는 `0`에 머무는데, "언젠가 둘이 같아지겠지"라고 생각하면 이 케이스를 이해할 수 없다.
- **고친 인식**: 루프 조건이 `while (A)`이면, 루프는 항상 `!A`가 될 때 끝난다. `A`가 `left <= right`니까 종료는 `left > right`. 이 관계를 조건문 그대로 뒤집어서 확인하는 습관을 들였다.
- **언제 다시 떠올릴까**: 이진 탐색에서 "종료 후 반환값이 뭐가 되어야 하나" 헷갈릴 때마다, 루프 조건을 먼저 문자 그대로 뒤집어서 종료 조건부터 확인한다.

## 꿀팁

- 이 문제는 일종의 "lower bound(하한) 이진 탐색"의 기본형이다 — "target 이상인 첫 위치를 찾아라" 같은 문제를 만나면 이 패턴(`left <= right`, 종료 후 `left` 반환)을 그대로 재사용할 수 있다.
- 헷갈릴 때는 극단적인 입력(배열의 모든 원소보다 크거나 작은 target)을 손으로 직접 추적해보는 게 종료 조건을 이해하는 데 제일 효과적이었다.

## AI라면 어떻게 풀었을까

704에서 언급한 `Arrays.binarySearch`의 "못 찾았을 때 음수를 반환한다"는 규약이 바로 이 문제(삽입 위치 찾기)를 위해 설계된 것이다. `Arrays.binarySearch(nums, target)`가 음수 `r`을 반환하면, 삽입 위치는 정확히 `-(r) - 1`이다.

```java
int r = Arrays.binarySearch(nums, target);
return r >= 0 ? r : -(r) - 1;
```

값을 찾으면 그 인덱스, 못 찾으면 코드 한 줄로 삽입 위치가 나온다. 직접 짠 `left <= right` 템플릿을 이해하고 나면, 이 라이브러리 메서드가 내부적으로 정확히 같은 일을 하고 있다는 걸 알아채는 게 포인트다 — 이해 없이 라이브러리만 썼다면 못 봤을 연결이다.
