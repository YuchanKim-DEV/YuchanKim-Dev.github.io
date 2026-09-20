---
title: "[LeetCode-1502] 정렬 후 인접 원소 비교로 등차수열 판별하기"
date: 2026-09-15 20:20:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, leetcode, sorting]
---

- 문제 링크: https://leetcode.com/problems/can-make-arithmetic-progression-from-sequence/
- 파일 경로: `src/main/java/coding_test/정렬/LC1502_CanMakeArithmeticProgressionFromSequence.java`
- 난이도: Easy

## 문제 설명

정수 배열 `arr`이 주어질 때, 순서를 재배치해서 등차수열(연속된 원소 차이가 전부 같은 수열)로 만들 수 있으면 `true`, 없으면 `false`를 반환한다.

```
Input: arr = [3,5,1]
Output: true  // [1,3,5]로 재배치하면 차이가 전부 2
```

## 풀이 과정

등차수열인지 확인하려면 순서를 알아야 하는데, 문제는 "재배치해서" 만들 수 있는지를 묻는다. 그래서 순서 자체를 고민할 필요 없이, 일단 정렬부터 하면 재배치 문제가 "정렬된 상태에서 인접한 차이가 전부 같은가"로 단순해진다는 걸 알아챘다.

정렬 후에는 첫 번째 차이(`arr[1] - arr[0]`)를 기준값으로 잡고, 나머지 인접 쌍들이 전부 같은 차이를 갖는지 반복문으로 확인하면 된다. 반복문 범위를 `i < arr.length - 1`로 잡아서 `arr[i+1]`이 항상 배열 안에 있도록 처음부터 신경 썼고, 크래시 없이 한 번에 통과했다.

## 최종 코드

```java
public static boolean canMakeArithmeticProgression(int[] arr) {
    Arrays.sort(arr);
    int commonDiff = arr[0] - arr[1];

    for (int i = 1; i < arr.length - 1; i++) {
        int diff = arr[i] - arr[i + 1];
        if (commonDiff != diff) {
            return false;
        }
    }

    return true;
}
```

## 오늘 배운 내용

**"재배치해서 조건을 만족시킬 수 있는가"를 묻는 문제는, 정렬로 순서를 하나로 고정하면 훨씬 단순한 문제로 바뀐다.** 그리고 인접한 두 인덱스(`i`, `i+1`)를 비교하는 반복문에서는 끝 조건(`i < length - 1`)을 먼저 확정해야 배열 밖을 건드리지 않는다.

## 오답노트

이번엔 딱히 틀린 게 없었다. 코드를 짜기 전에 "이 반복문에서 배열 밖을 건드릴 수 있는 인덱스가 있는가"를 먼저 점검하고 시작했기 때문이다.
