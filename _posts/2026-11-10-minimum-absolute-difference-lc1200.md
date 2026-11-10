---
title: "[LeetCode-1200] Minimum Absolute Difference - 차이는 abs(a - b), abs(a) - abs(b)가 아니다"
date: 2026-11-10 09:31:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, leetcode, sort, array, abs]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/minimum-absolute-difference/
- 파일 경로: `src/main/java/coding_test/정렬/LC1200_MinimumAbsoluteDifference.java`
- 난이도: Easy

## 문제 설명

서로 다른 정수 배열 `arr`에서, 두 원소의 차이가 가장 작은 쌍 `[a, b]`(a < b)를 오름차순으로 모두 반환한다.

```
[4,2,1,3]                  → [[1,2],[2,3],[3,4]]
[1,3,6,10,15]              → [[1,3]]
[3,8,-10,23,19,-4,-14,27]  → [[-14,-10],[19,23],[23,27]]
```

11/09 기준점 테스트(90분, 힌트 없음)의 세 번째 문제였다. 시험에서는 4/6으로 끝났고, 다음 날 두 줄을 고쳐 통과했다.

## 시행착오

### 시험 때 구조는 맞았다

정렬 → 인접한 두 수의 차이로 최솟값 구하기 → 다시 돌면서 최솟값과 같은 쌍 모으기. 이 틀은 처음부터 정확했다. 틀린 건 두 줄이다.

### 1. 최솟값 시작값을 `arr[마지막]`으로 잡음

```java
int minAbs = arr[arr.length - 1];   // [-1000000, 1000000]이면 시작값 1000000
```

배열의 가장 큰 값이 차이보다 작을 수 있다. `[-1000000, 1000000]`은 차이가 2000000인데 시작값이 1000000이라 최솟값이 갱신되지 않는다. 시작값은 **어떤 차이보다도 큰 값**이어야 한다.

### 2. 차이를 `abs(a) - abs(b)`로 계산

```java
if (Math.abs(arr[i]) - Math.abs(arr[i - 1]) == minAbs)   // 틀림
if (Math.abs(arr[i] - arr[i - 1]) == minAbs)             // 맞음
```

| a | b | abs(b) - abs(a) | abs(b - a) |
|---|---|---|---|
| -14 | -10 | -4 | **4** |
| -4 | 3 | -1 | **7** |

음수가 섞이면 "절댓값끼리 뺀 값"은 거리가 아니다. 위 줄(최솟값 구하기)에서는 맞게 썼는데 아래 줄에서만 다르게 써서, 예시 3에서 `[-14,-10]`이 빠졌다.

## 최종 코드

```java
public List<List<Integer>> minimumAbsDifference(int[] arr) {
    List<List<Integer>> ans = new ArrayList<>();

    Arrays.sort(arr);

    int minAbs = 10000000;
    for (int i = 1; i < arr.length; i++) {
        minAbs = Math.min(minAbs, Math.abs(arr[i] - arr[i - 1]));
    }

    for (int i = 1; i < arr.length; i++) {
        if (Math.abs(arr[i] - arr[i - 1]) == minAbs) {
            List<Integer> list = new ArrayList<>();
            list.add(arr[i - 1]);
            list.add(arr[i]);
            ans.add(list);
        }
    }

    return ans;
}
```

## 추적표: `[10,0,5,20]`

정렬하면 `[0,5,10,20]`.

| i | 쌍 | 차이 | minAbs | 담김? |
|---|---|---|---|---|
| 1 | (0,5) | 5 | 5 | ✅ |
| 2 | (5,10) | 5 | 5 | ✅ |
| 3 | (10,20) | 10 | 5 | ❌ |

결과 `[[0,5],[5,10]]`.

## 오늘 배운 내용

- 정렬하면 가장 가까운 두 수는 **반드시 서로 이웃**이다. 모든 쌍을 볼 필요가 없다.
- 차이(거리)는 `Math.abs(a - b)`. 정렬 뒤에는 `arr[i] - arr[i-1]`이 항상 양수라 `abs`도 필요 없다.
- 최솟값 시작값은 데이터 안의 값이 아니라 `Integer.MAX_VALUE`처럼 확실히 큰 값으로.

## 오답노트

- 같은 계산을 두 곳에 쓰면 두 곳이 같은 식인지 확인한다. (위는 맞고 아래는 틀렸음)
- 예시 3처럼 **음수가 섞인 예시를 손으로 한 번 돌려 보면** 바로 잡히는 실수였다. 시험 때 예시 검증을 건너뛴 게 원인.

## AI라면 어떻게 풀었을까

반복문 하나로 줄인다. 더 작은 차이를 만나면 지금까지 모은 쌍을 비우고 새로 모은다.

```java
Arrays.sort(arr);
int min = Integer.MAX_VALUE;
for (int i = 1; i < arr.length; i++) {
    int diff = arr[i] - arr[i - 1];
    if (diff < min) {
        min = diff;
        ans.clear();
    }
    if (diff == min) ans.add(Arrays.asList(arr[i - 1], arr[i]));
}
```
