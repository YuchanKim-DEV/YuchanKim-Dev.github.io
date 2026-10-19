---
title: "[LeetCode-162] 봉우리 찾기 - 오른쪽 이웃 하나만 본다"
date: 2026-10-19 14:25:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search, peak]
---

> 이진 탐색 개념과 패턴은 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다. 같은 계열의 바로 앞 문제는 [LC852 글]({% post_url 2026-10-19-peak-index-mountain-lc852 %})이다.

- 문제 링크: https://leetcode.com/problems/find-peak-element/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0162_FindPeakElement.java`
- 난이도: Medium

## 문제 설명

양옆 이웃보다 엄격하게 큰 원소를 봉우리라고 한다. 배열에서 봉우리 **하나**의 인덱스를 `O(log n)`으로 반환한다. 봉우리가 여러 개면 아무거나 반환해도 되고, 배열 양 끝 바깥은 `-∞`로 본다.

```
nums = [1,2,3,1]         → 2
nums = [1,2,1,3,5,6,4]   → 1 또는 5
```

## 시행착오

LC852(산 모양 배열의 꼭대기)를 한 번에 푼 직후 "같은 걸로 하나만 더" 풀었다. 그런데 LC852와 다른 점(봉우리가 여러 개, 양 끝은 -∞)에 신경 쓰다가 오히려 두 번 돌아갔다.

### 1차: "지금까지의 최댓값"과 비교

```java
int max = 0;
if (nums[mid] >= max) { max = Math.max(max, nums[mid]); left = mid + 1; }
else { right = mid - 1; }
```

봉우리는 "지금까지 본 값 중 최대"가 아니라 **바로 옆 이웃보다 큰 값**이다. `max`는 이미 지나간 칸의 값이라 어느 쪽에 봉우리가 있는지 알려 주지 못한다. 11개 중 6개가 "봉우리 아님"이었다. `max = 0`으로 시작해서 음수 배열에서도 틀렸고, `right = mid - 1`로 봉우리일 수 있는 `mid`를 버렸다.

### 2차: 양옆을 다 봄

```java
if (nums[mid] >= nums[mid + 1] && nums[mid] >= nums[mid - 1]) { left = mid + 1; }
else { right = mid; }
```

"`nums[mid]`와 `nums[mid + 1]`을 비교하라"를 "양옆을 다 조사하라"로 받아들였다. 두 가지가 틀렸다.

1. `mid = 0`이면 `nums[-1]`을 읽어서 `ArrayIndexOutOfBoundsException`이 났다.
2. **방향이 반대**였다. `mid`가 봉우리(양옆보다 큼)일 때 `left = mid + 1`로 그 봉우리를 버렸다.

### 3차: 오른쪽 이웃 하나만

LC852와 **완전히 같은 코드**로 통과했다. 봉우리가 여러 개여도, 양 끝이 -∞여도 이 비교 하나로 충분하다.

## 최종 코드

```java
public int findPeakElement(int[] nums) {
    int left = 0;
    int right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < nums[mid + 1]) {
            left = mid + 1;          // 오르막 → 오른쪽에 봉우리가 반드시 있다
        } else {
            right = mid;             // 내리막 → mid 또는 왼쪽에 봉우리가 있다
        }
    }
    return left;
}
```

11개 테스트가 모두 "봉우리 맞음"이었고, 랜덤 50,000건(`Integer.MIN_VALUE` 포함)에서도 항상 봉우리를 반환했다.

## 추적표: `[1, 2, 1, 3, 5, 6, 4]`

| 단계 | left | right | mid | nums[mid] vs nums[mid+1] | 이동 |
|---|---|---|---|---|---|
| 1 | 0 | 6 | 3 | 3 < 5 (오르막) | `left = 4` |
| 2 | 4 | 6 | 5 | 6 > 4 (내리막) | `right = 5` |
| 3 | 4 | 5 | 4 | 5 < 6 (오르막) | `left = 5` |
| 끝 | 5 | 5 | | | **5 반환** (봉우리 6) |

## 오늘 배운 내용

**오르막이면 그 오른쪽 어딘가에 봉우리가 반드시 있다.** 계속 올라가다 끝에 닿으면 끝이 봉우리(바깥은 -∞)이고, 중간에 내려가면 그 직전이 봉우리다. 그래서 봉우리가 여러 개여도 "오르막이면 오른쪽, 아니면 왼쪽(`mid` 포함)"만으로 하나는 반드시 찾는다.

**`left < right` 틀에서는 `mid + 1`은 안전하지만 `mid - 1`은 위험하다.** `mid`는 `right`보다 항상 작아서 `mid + 1`은 범위 안이지만, `mid`가 0이면 `mid - 1`은 -1이다. 필요 없는 이웃은 읽지 않는다.

## 오답노트

- **틀렸던 지점 1**: "지금까지의 최댓값"과 비교. **왜**: 봉우리를 "가장 큰 값"으로 착각했다. **다음에**: 봉우리 = 이웃보다 큰 값. 비교 대상은 이웃이다.
- **틀렸던 지점 2**: 양옆을 다 보고 `nums[-1]`에서 에러. **왜**: "이웃과 비교"를 양쪽 모두로 해석했다. **다음에**: 방향을 정하는 데는 **한쪽 이웃(오른쪽)** 하나면 충분하다.
- **틀렸던 지점 3**: 봉우리일 때 `left = mid + 1`. **왜**: 조건을 "봉우리면"으로 쓰고 방향을 따로 생각하지 않았다. **다음에**: 조건은 "오르막인가?"로 쓰고, 오르막일 때만 `mid`를 버린다.
- **좋았던 점**: 결국 LC852와 같은 코드가 답이라는 걸 확인했다. 같은 틀이 두 문제에 그대로 통한다.

## AI라면 어떻게 풀었을까

한 점으로 모일 때까지 좁히는 대신 **`mid`가 봉우리인지 그 자리에서 확인**한다. 양 끝 바깥은 -∞이므로 "`mid`가 0이면 왼쪽은 무조건 낮다", "`mid`가 마지막이면 오른쪽은 무조건 낮다"로 처리해서 배열 밖을 읽지 않는다. `Integer.MIN_VALUE`를 -∞ 대신 쓰면 값이 정말 `MIN_VALUE`인 경우에 틀리므로 **인덱스로** 처리한다.

```java
public int findPeakElementAlt(int[] nums) {
    int left = 0;
    int right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        boolean leftLower = mid == 0 || nums[mid - 1] < nums[mid];
        boolean rightLower = mid == nums.length - 1 || nums[mid + 1] < nums[mid];
        if (leftLower && rightLower) {
            return mid;                      // 양옆보다 크면 봉우리
        } else if (!rightLower) {
            left = mid + 1;                  // 오른쪽이 더 높음 → 오른쪽에 봉우리
        } else {
            right = mid - 1;                 // 왼쪽이 더 높음 → 왼쪽에 봉우리
        }
    }
    return -1;
}
```

트레이드오프: 봉우리를 만나면 바로 끝나고 "양 끝은 -∞"를 코드로 그대로 보여 준다. 대신 비교가 두 번이고 경계 조건(`mid == 0`, `mid == n-1`)을 직접 챙겨야 한다. 랜덤 50,000건에서 본인 풀이와 함께 항상 봉우리를 반환했다.
