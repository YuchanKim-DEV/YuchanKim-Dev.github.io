---
title: "[LeetCode-33] 회전 배열 검색 - 정렬된 쪽을 찾고, target이 그 안에 있는지 본다"
date: 2026-10-19 11:14:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search, rotated-array]
---

> 이진 탐색 개념과 패턴은 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다. 같은 패턴의 첫 문제는 [LC153 글]({% post_url 2026-10-16-find-minimum-rotated-lc153 %})이다.

- 문제 링크: https://leetcode.com/problems/search-in-rotated-sorted-array/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0033_SearchInRotatedSortedArray.java`
- 난이도: Medium

## 문제 설명

서로 다른 값의 오름차순 배열이 어떤 위치에서 회전되어 있다(회전 안 된 경우도 있음). `target`이 있으면 **회전된 배열에서의** 인덱스를, 없으면 `-1`을 `O(log n)`으로 반환한다.

```
nums = [4,5,6,7,0,1,2], target = 0 → 4
nums = [4,5,6,7,0,1,2], target = 3 → -1
nums = [1], target = 0             → -1
```

## 시행착오

LC153(회전 배열의 최솟값)을 푼 지 사흘 뒤, 같은 패턴을 정착시키려고 이어서 풀었다. 결과적으로 거의 모든 단계에서 도움을 받았다.

### 0차: `Arrays.sort` 해도 되나?

안 된다. ① 문제는 **회전된 원래 배열에서의** 인덱스를 묻는데 정렬하면 위치가 바뀐다(`0`은 4번인데 정렬하면 0번). ② 정렬은 `O(n log n)`이라 조건을 어긴다.

### 1차: LC153 코드를 그대로 가져옴 → 없는 값에서 무한 루프

```java
} else if (nums[mid] > nums[right]) {
    left = mid + 1;
} else if (nums[mid] < nums[right]) {
    right = mid - 1;
}
```

두 가지 문제가 있었다.

1. **`nums[mid] == nums[right]`(즉 `left == right`)인 경우가 어느 분기에도 안 들어가서** `left`, `right`가 안 움직이고 같은 자리를 계속 돌았다.
2. 더 큰 문제는 **분기가 target을 전혀 보지 않았다**는 것이다. LC153의 "최솟값 쪽으로 가기"만 했고, 예시 1은 target `0`이 마침 최솟값이라 우연히 찾았다.

### 2차: "정렬된 쪽"이 무슨 뜻인지 헷갈림

"정렬된 쪽을 찾으라는 게 target이 포함된 범위를 찾으라는 건가? `right`를 target으로 두면 되나?"를 물었다. 아니다.

`mid`로 반을 나누면 **두 반 중 하나는 항상 끊김 없이 오름차순**이다. `[4,5,6,7,0,1,2]`에서 `mid = 7`이면 왼쪽 `[4,5,6,7]`이 그런 쪽이다. 정렬된 쪽은 **양 끝 값만 보면** target이 안에 있는지 알 수 있다(0은 4~7 사이가 아니다 → 반대쪽으로 간다).

### 3차: 규칙대로 옮겼는데 다시 무한 루프 + 끝값 누락

```java
} else if (nums[mid] < nums[right]) { ... }   // "그 외에는"을 else if로 옮김 → == 경우 누락
if (nums[mid] > target && nums[left] < target)  // target == nums[left]면 범위 밖으로 판단
```

규칙의 "그 외에는"은 조건 없는 `else`였는데 `else if (<)`로 써서 같은 값인 경우가 또 빠졌다. 또 범위 확인에서 끝값을 `<`로 써서 target이 정렬된 쪽의 끝값과 같을 때 놓쳤다(`[4,5,6,7,0,1,2]`, target `4`). 끝값은 `<=`, `>=`여야 했다.

### 4차: 무한 루프를 피하려고 추가한 분기

```java
} else if (nums[left] == nums[right]) {
    return left;            // target인지 확인하지 않고 반환
}
```

이 분기 때문에 없는 값(`target = 3`, `[1]`에서 `0` 등)에서 `-1` 대신 인덱스가 나왔다. 이 분기를 지우고 위의 `else if`를 `else`로 바꿔 통과했다.

## 최종 코드

```java
public int search(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] > nums[right]) {               // 왼쪽 반이 정렬된 쪽
            if (nums[mid] > target && nums[left] <= target) {
                right = mid - 1;                             // target이 왼쪽 범위 안
            } else {
                left = mid + 1;
            }
        } else {                                             // 오른쪽 반이 정렬된 쪽
            if (nums[mid] < target && nums[right] >= target) {
                left = mid + 1;                              // target이 오른쪽 범위 안
            } else {
                right = mid - 1;
            }
        }
    }
    return -1;
}
```

12개 테스트가 모두 통과했고, 랜덤 50,000건(모든 회전, 없는 값 포함)에서도 정답과 일치했다.

## 추적표: `[4,5,6,7,0,1,2]`, `target = 0`

| 단계 | 범위 | mid 값 | 정렬된 쪽 | 0이 그 안에? | 이동 |
|---|---|---|---|---|---|
| 1 | 0~6 | 7 | 왼쪽 `[4~7]` | 아니요 | `left = 4` |
| 2 | 4~6 | 1 | 오른쪽 `[1~2]` | 아니요 | `right = 4` |
| 3 | 4~4 | 0 | | `nums[mid] == target` | **4 반환** |

## 오늘 배운 내용

**회전 배열에서 "정렬된 쪽"은 `mid`와 `right` 비교로 알 수 있고, 그 쪽은 양 끝 값만으로 target 포함 여부를 판단할 수 있다.** 포함되면 그쪽으로, 아니면 반대쪽으로 간다. 모든 분기에서 `left`나 `right`가 반드시 움직이므로 무한 루프가 없다.

**분기를 `if / else if / else if`로 쓰면 빠진 경우가 생긴다.** "그 외에는"은 조건 없는 `else`로 써야 모든 경우가 처리된다.

## 오답노트

- **틀렸던 지점 1**: `Arrays.sort`를 떠올림. **왜**: "정렬된 배열"이라는 말에 끌렸다. **다음에**: 반환값이 **인덱스**면 배열 순서를 바꾸는 순간 답이 깨진다.
- **틀렸던 지점 2**: LC153 분기를 그대로 써서 target을 안 봄. **왜**: 같은 패턴이라 코드도 같다고 생각했다. **다음에**: 패턴(정렬된 쪽 찾기)은 같아도 **무엇을 찾는지**(최솟값 vs target)에 따라 이동 조건이 달라진다.
- **틀렸던 지점 3**: "그 외에는"을 `else if`로 씀. **왜**: 조건을 대칭으로 쓰고 싶었다. **다음에**: 분기를 다 쓰고 나서 "어느 분기에도 안 걸리는 경우가 있나?"를 확인한다. 있으면 무한 루프다.
- **틀렸던 지점 4**: 범위 끝값을 `<`로 씀. **왜**: 끝값이 target인 경우를 대입해 보지 않았다. **다음에**: 범위 확인은 정렬된 쪽의 **양 끝값이 target일 때**를 꼭 넣어 본다.
- **틀렸던 지점 5**: 무한 루프를 피하려고 `return left`를 추가. **왜**: 증상(멈춤)만 막으려 했다. **다음에**: 반환하는 분기에는 "정말 target인가?"가 들어 있어야 한다.

## AI라면 어떻게 풀었을까

"정렬된 쪽 + 범위 확인"을 한 반복에서 같이 하는 대신 **이진 탐색을 두 번으로 나눈다.** ① LC153 방식으로 최솟값 위치(`pivot`)를 찾고, ② "회전을 되돌린 가상의 정렬 배열"에서 평범한 이진 탐색을 한다. 가상 인덱스 `mid`의 실제 위치는 `(mid + pivot) % n`이다.

```java
public int searchAlt(int[] nums, int target) {
    int n = nums.length;
    int lo = 0, hi = n - 1;
    while (lo < hi) {                              // 1) 최솟값 위치 찾기 (LC153)
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    int pivot = lo;

    lo = 0;
    hi = n - 1;
    while (lo <= hi) {                             // 2) 평범한 이진 탐색 (LC704)
        int mid = lo + (hi - lo) / 2;
        int real = (mid + pivot) % n;              // 가상 인덱스 → 실제 인덱스
        if (nums[real] == target) {
            return real;
        } else if (nums[real] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return -1;
}
```

트레이드오프: 각 단계가 이미 아는 틀(LC153, LC704)이라 조건을 섞지 않아도 되고, 끝값 `<=` 실수도 생기기 어렵다. 대신 이진 탐색을 두 번 돌고(`O(log n)`은 그대로), `(mid + pivot) % n`이라는 인덱스 변환을 이해해야 한다. 랜덤 50,000건에서 본인 풀이와 출력이 일치했다.
