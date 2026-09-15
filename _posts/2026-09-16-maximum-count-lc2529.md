---
title: "[LeetCode-2529] \"0은 양수도 음수도 아니다\"가 이진 탐색의 의미를 어떻게 바꾸는가"
date: 2026-09-16 01:30:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search]
---

> 이진 탐색 개념과 패턴 정리는 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/maximum-count-of-positive-integer-and-negative-integer/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC2529_MaximumCountOfPositiveIntegerAndNegativeInteger.java`
- 난이도: Easy

## 문제 설명

오름차순 정렬된 배열 `nums`가 주어진다. 양수 개수(`pos`)와 음수 개수(`neg`) 중 더 큰 값을 반환한다. `0`은 양수도 음수도 아니다.

```
Input: nums = [-3,-2,-1,0,0,1,2]
Output: 3  // neg=3, pos=2
```

## 풀이 과정

### 1차 — 방향이 반대였다

"첫 번째 0 이상인 값의 위치"를 찾으려고 이진 탐색을 짰는데, `nums[mid] < 0`일 때 `right = mid - 1`(왼쪽으로 좁힘)로 짰다. 근데 `nums[mid]`가 음수라면 답(0 이상이 시작되는 위치)은 `mid`보다 **오른쪽**에 있어야 한다. 방향을 반대로(`left = mid + 1`) 고치니 맞았다.

### 2차 — 0을 어떻게 뺄지

`left`(처음 0 이상인 위치)를 구하면 `neg`는 바로 나온다. 문제는 `pos`였다 — `nums.length - left`로 하면 0까지 같이 세어버린다. 처음엔 `nums[mid] == 0`일 때를 실수로 "양수" 방향(`right = mid - 1`)으로 묶어버려서, `left`가 "처음 0 이상인 위치"가 아니라 "처음 진짜 양수인 위치"로 의미가 바뀌어버렸다. 그러면 그 뒤에 있는 "0의 개수를 세는" 반복문이 엉뚱한 구간(이미 양수만 남은 구간)을 보게 되어 항상 0개로 나왔다.

`nums[mid] == 0`을 다시 "음수" 방향(`left = mid + 1`)으로 되돌려서, `left`가 확실히 "처음 0 이상인 위치"를 가리키게 고정한 다음, 그 위치부터 끝까지 0의 개수를 세는 반복문을 추가해서 `pos = nums.length - left - (0의 개수)`로 구했다.

## 최종 코드

```java
public static int maximumCount(int[] nums) {
    int left = 0;
    int right = nums.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < 0) {
            left = mid + 1;
        } else if (nums[mid] > 0) {
            right = mid - 1;
        } else {
            right = mid - 1;
        }
    }

    int count = 0;
    for (int i = left; i < nums.length; i++) {
        if (nums[i] == 0) {
            count++;
        }
    }

    int neg = left;
    int pos = nums.length - left - count;
    return Math.max(pos, neg);
}
```

## 오늘 배운 내용

**이진 탐색에서 "0을 어느 쪽으로 취급하느냐"가 `left`가 최종적으로 가리키는 의미 자체를 바꾼다.** `nums[mid]==0`을 음수 쪽에 붙이면 `left`는 "처음 0 이상인 위치"가 되고, 양수 쪽에 붙이면 "처음 진짜 양수인 위치"가 된다. 둘 다 유효한 이진 탐색이지만, 그 뒤에 이어지는 계산(0 개수를 세는 범위 등)이 그 의미에 맞춰 달라져야 한다는 걸 놓쳤었다.

## 오답노트

- **틀렸던 패턴 1**: `nums[mid] < 0`일 때 `right = mid - 1`(방향 반대). **다음에 떠올릴 시점**: 이진 탐색 방향을 정할 때마다 "이 조건이 참이면 답은 mid보다 왼쪽/오른쪽 중 어디에 있는가"를 말로 먼저 확인한다.
- **틀렸던 패턴 2**: `nums[mid]==0` 분기를 양수 쪽에 붙여서 `left`의 의미가 바뀐 걸 못 알아챔. **다음에 떠올릴 시점**: 경계값(0, 중복값 등)을 어느 쪽으로 묶을지 정할 때, 그 선택이 최종적으로 `left`/`right`가 뭘 의미하게 되는지부터 먼저 확인한다.

## 꿀팁

같은 `lowerBound`(target 이상인 첫 위치 찾기) 로직을 헬퍼 함수로 뽑아두면, "0 이상인 첫 위치"와 "1 이상인 첫 위치"를 각각 구해서 반복문 없이 `neg`/`pos`를 바로 계산할 수도 있다:

```java
private static int lowerBound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return left;
}
// neg = lowerBound(nums, 0)
// pos = nums.length - lowerBound(nums, 1)
```
