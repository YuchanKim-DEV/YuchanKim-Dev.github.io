---
title: "[LeetCode-35] 백지 재작성 - 끝났을 때 right는 '작은 수 중 최대', left는 그 다음 칸"
date: 2026-10-16 09:58:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search, lower-bound]
---

> [첫 풀이 글]({% post_url 2026-09-09-search-insert-position-lc35 %})에 이은 백지 재작성이다. 이진 탐색 개념과 패턴은 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/search-insert-position/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0035_SearchInsertPosition.java`
- 난이도: Easy

## 문제 설명

(자세한 설명은 [첫 풀이 글]({% post_url 2026-09-09-search-insert-position-lc35 %}) 참고) 오름차순 정렬된 서로 다른 정수 배열에서 `target`이 있으면 그 인덱스를, 없으면 순서를 유지하며 삽입할 위치를 `O(log n)`으로 반환한다.

```
nums = [1,3,5,6], target = 5 → 2
nums = [1,3,5,6], target = 2 → 1
nums = [1,3,5,6], target = 7 → 4
```

## 왜 다시 풀었나

스택·큐에 3주를 쓰고 이진 탐색으로 돌아오니 "감각을 다 잃어버린 것 같다"는 느낌이 들었다. 전날 LC1351에서도 `mid` 식이나 `+1` 같은 오타성 실수가 이어졌다. 그래서 새 문제로 가기 전에 **가장 기본형**을 백지에서 한 번 다시 짰다.

## 시행착오

### 1차: target이 없을 때만 전부 틀림

```java
while (left < right) { ... }
return right;
```

10개 중 4개가 틀렸고, **전부 target이 배열에 없는 경우**였다. 스스로 짚은 지점은 이거였다. "없을 때 **작은 수 중 최대값의 인덱스**를 찾는 걸 못 했다."

생각은 거의 맞았다. `[1,3,5,6]`, `target = 2`라면 작은 수 중 최대값은 `1`(인덱스 0)이다. 그런데 답은 **1**이다. 2는 1 **다음 칸**에 들어가기 때문이다.

> 삽입 위치 = (작은 수 중 최대값의 인덱스) **+ 1**

### 2차: 반복 조건은 고쳤지만 반환이 그대로

`right = mid - 1`을 쓰는 틀은 `left <= right`까지 돌아야 하는데 `<`로 써서 `left == right`인 칸을 확인하지 않고 끝났다. `<=`로 고치자 이번엔 출력이 이렇게 나왔다.

| 테스트 | 정답 | 출력 |
|---|---|---|
| 예시 2 | 1 | 0 |
| 예시 3 | 4 | 3 |
| Edge 1 | 0 | -1 |
| Edge 5 | 0 | -1 |

**틀린 것들이 전부 정답보다 정확히 1 작았다.** `right`가 바로 "작은 수 중 최대값의 위치"이기 때문이다. 삽입 위치는 그 다음 칸인 `left`다.

## 최종 코드

```java
public int searchInsert(int[] nums, int target) {
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
    return left;                 // 끝나면 left = right + 1 = 삽입 위치
}
```

10개 테스트가 모두 통과했고, 랜덤 50,000건에서도 정답과 일치했다.

## 추적표: `[1,3,5,6]`, `target = 2`

| 단계 | left | right | mid | 값 | 이동 |
|---|---|---|---|---|---|
| 1 | 0 | 3 | 1 | 3 (> 2) | `right = 0` |
| 2 | 0 | 0 | 0 | 1 (< 2) | `left = 1` |
| 끝 | **1** | **0** | | | `left > right`라 종료 |

끝났을 때 `right = 0`은 "작은 수 중 최대값(`1`)의 위치", `left = 1`은 "그 다음 칸 = 삽입 위치"다.

## 오늘 배운 내용

**`left <= right` 틀에서 반복이 끝나면 항상 `left = right + 1`이다.** 그 순간 `right`는 "target보다 작은 것 중 마지막", `left`는 "target 이상인 것 중 처음"을 가리킨다. 찾으려는 것이 둘 중 어느 쪽인지만 정하면 반환할 변수가 정해진다.

**출력이 정답과 "정확히 1씩" 차이 나면 경계 변수 하나를 잘못 고른 것이다.** 로직 전체를 의심하기 전에 `left`와 `right` 중 어느 것을 돌려줘야 하는지부터 본다.

## 오답노트

- **틀렸던 지점 1**: `left < right`로 반복. **왜**: 두 가지 틀(`<=` + `mid - 1` / `<` + `mid`)을 섞었다. **다음에**: `right = mid - 1`을 쓰면 `<=`, `right = mid`를 쓰면 `<`. 짝을 맞춘다.
- **틀렸던 지점 2**: `return right`. **왜**: "작은 수 중 최대값의 위치"를 답으로 착각했다. **다음에**: "삽입 위치 = 그 다음 칸"이므로 `left`다. 손으로 한 번 트레이스해서 끝났을 때 두 변수가 어디를 가리키는지 확인한다.
- **좋았던 점**: 틀린 원인을 스스로 "없을 때 작은 수의 최대값 인덱스"로 정확히 짚었다. 이진 탐색 틀 자체(`mid` 식, `+1`, `-1`)는 처음부터 맞았다.

## AI라면 어떻게 풀었을까

"찾으면 바로 반환, 못 찾으면 `left`" 두 갈래 대신 처음부터 **"target 이상인 첫 번째 위치" 하나만** 찾는다(하한 경계, lower bound). target이 있으면 그 위치가 곧 인덱스이고, 없으면 그 위치가 삽입 위치라서 두 경우가 한 번에 끝난다. 전날 LC1351에서 "첫 음수 위치"를 찾은 것과 완전히 같은 모양이다.

```java
public int searchInsertAlt(int[] nums, int target) {
    int left = 0;
    int right = nums.length;               // 끝 바깥까지 후보
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;                // mid는 target보다 작음 → 버림
        } else {
            right = mid;                   // mid는 target 이상 → 후보로 남김
        }
    }
    return left;                           // target 이상인 첫 위치
}
```

트레이드오프: `==` 분기가 없어 조건이 하나 줄고, 모든 원소가 target보다 작을 때도 `right = nums.length`라서 자연스럽게 끝 위치가 나온다. 대신 target이 있을 때도 끝까지 좁혀 가므로 "찾자마자 반환"보다 반복이 몇 번 더 돌 수 있다(둘 다 `O(log n)`). 랜덤 50,000건에서 본인 풀이와 출력이 일치했다.
