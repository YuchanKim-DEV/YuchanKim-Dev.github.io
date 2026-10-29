---
title: "[LeetCode-1331] Rank Transform of an Array - 같은 값은 같은 순위"
date: 2026-10-29 10:15:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, leetcode, sort, rank, index-sort]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/rank-transform-of-an-array/
- 파일 경로: `src/main/java/coding_test/정렬/LC1331_RankTransformOfAnArray.java`
- 난이도: Easy

## 문제 설명

각 원소를 **순위**로 바꾼다. **작은 값이 1등**이고, **같은 값은 같은 순위**다. 순위는 빈 번호 없이 1, 2, 3...으로 붙인다. 값의 범위는 -10억 ~ 10억이다.

```
[40,10,20,30]               → [4,1,2,3]
[100,100,100]               → [1,1,1]
[37,12,28,9,100,56,80,5,12] → [5,3,4,2,8,6,7,1,3]
```

아침에 푼 [진료순서 정하기]({% post_url 2026-10-29-treatment-order-programmers-lv0 %})와 같은 순위 패턴이다. 처음 본 패턴은 같은 패턴을 하나 더 푼다는 규칙에 따라 이어서 풀었다. 이번에는 **중복값**이 추가됐다.

## 시행착오

이번에는 아침에 AI 풀이로 봤던 **번호표 정렬 + `ans[idx[i]] = rank`**를 처음부터 직접 썼다. 구조는 맞았고, 세부 세 군데를 고쳤다.

### 1차: 정렬 방향 + 원래 배열에서 이웃 비교

```java
Arrays.sort(idx, (a, b) -> arr[b] - arr[a]);   // 큰 값이 앞
...
if (arr[i - 1] == arr[i]) { ... }             // 원래 배열의 옆 칸
```

- 작은 값이 1등인데 내림차순으로 정렬했다.
- 같은 값끼리 붙어 있는 건 **정렬된 순서**에서다. 그런데 원래 배열의 옆 칸끼리 비교했다. 아침과 같은 "정렬된 배열 vs 원래 배열" 혼동이다.
- 덤으로, 값이 ±10억이라 `arr[b] - arr[a]`는 `int` 범위를 넘을 수 있다.

### 2차: 비교는 고침, 순위를 거꾸로 셈

비교를 `arr[idx[i - 1]] == arr[idx[i]]`로, 정렬은 `Integer.compare`로 고쳤다. 다만 내림차순을 유지한 채 `rank`를 `arr.length`부터 하나씩 줄였다. 중복이 없으면 맞지만 `[100,100,100]`은 `[3,3,3]`이 나온다. 서로 다른 값의 개수가 길이보다 적으니 시작 숫자가 너무 크다.

### 3차: 통과

오름차순 정렬 + `rank = 1`부터 `+= 1`로 바꿔 통과했다. 가장 작은 값은 항상 1등이라 시작점이 틀릴 일이 없다. (중간에 `rank`만 바꾸고 정렬 방향을 안 바꿔 한 번 더 뒤집힌 결과가 나왔다.)

## 최종 코드

```java
public int[] arrayRankTransform(int[] arr) {
    Integer[] idx = new Integer[arr.length];               // 번호표
    for (int i = 0; i < arr.length; i++) {
        idx[i] = i;
    }

    Arrays.sort(idx, (a, b) -> Integer.compare(arr[a], arr[b]));   // 작은 값의 번호가 앞
    int[] ans = new int[arr.length];
    int rank = 1;
    for (int i = 0; i < arr.length; i++) {
        if (i == 0) {
            ans[idx[i]] = rank;
        } else {
            if (arr[idx[i - 1]] == arr[idx[i]]) {          // 정렬된 순서의 바로 앞 값과 같으면
                ans[idx[i]] = rank;                        // 같은 순위
            } else {
                rank += 1;
                ans[idx[i]] = rank;
            }
        }
    }
    return ans;
}
```

## 추적표: `[3,1,3,2]`

번호표 정렬 결과 `idx = [1, 3, 0, 2]` (값 1, 2, 3, 3)

| i | idx[i] | 값 | 앞 값과 같음? | rank | ans |
|---|---|---|---|---|---|
| 0 | 1 | 1 | - | 1 | `[_, 1, _, _]` |
| 1 | 3 | 2 | 아니오 | 2 | `[_, 1, _, 2]` |
| 2 | 0 | 3 | 아니오 | 3 | `[3, 1, _, 2]` |
| 3 | 2 | 3 | **예** | 3 | `[3, 1, 3, 2]` |

## 오늘 배운 내용

**순위를 셀 때는 시작이 확실한 쪽부터 센다.** 가장 작은 값의 순위는 항상 1이다. 반면 가장 큰 값의 순위는 중복이 몇 개인지 다 봐야 알 수 있다.

**값 범위가 ±10억이면 Comparator에서 빼기를 쓰지 않는다.** `Integer.compare(x, y)`를 쓴다.

## 오답노트

- **틀렸던 지점**: `arr[i - 1] == arr[i]`. **왜**: 아침 문제와 같이, 정렬된 순서로 봐야 할 곳에서 원래 배열을 봤다. **다음에**: 번호표 정렬 뒤에는 값을 꺼낼 때 항상 `arr[idx[i]]` 모양인지 확인한다.
- **틀렸던 지점**: 순위를 `arr.length`부터 줄여 감. **왜**: 정렬 방향을 바꾸지 않고 순위 쪽을 맞추려 했다. **다음에**: 시작값을 확실히 알 수 있는 방향으로 정렬한다.

## AI라면 어떻게 풀었을까

값을 복사해 정렬하고 **"값 → 순위" map**을 만든다. 처음 보는 값에만 다음 순위를 붙인다.

```java
public int[] arrayRankTransformAlt(int[] arr) {
    int[] sorted = arr.clone();
    Arrays.sort(sorted);                               // 오름차순 (정렬됨)
    HashMap<Integer, Integer> rank = new HashMap<>();  // 값 → 순위
    for (int x : sorted) {
        if (!rank.containsKey(x)) {
            rank.put(x, rank.size() + 1);              // 처음 보는 값만 다음 순위
        }
    }
    int[] ans = new int[arr.length];
    for (int i = 0; i < arr.length; i++) {
        ans[i] = rank.get(arr[i]);                     // 원래 순서대로 꺼내기
    }
    return ans;
}
```

같은 값은 map에 한 번만 들어가므로 같은 순위가 저절로 된다. `int[]` 기본 정렬이라 Comparator와 넘침 걱정도 없다. 아침에 본인이 진료순서 정하기를 푼 방식(값 → 순위 map)에 "처음 보는 값만"이라는 조건 하나를 더한 모양이다. ±10억 극단값을 섞은 랜덤 20,000건에서 두 풀이의 결과가 같았다.
