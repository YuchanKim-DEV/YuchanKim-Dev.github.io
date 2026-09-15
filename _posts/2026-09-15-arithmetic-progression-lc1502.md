---
title: "[LeetCode-1502] 이진 탐색 말고, 배열 경계 감각만 따로 떼서 연습하기"
date: 2026-09-15 20:20:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, leetcode, sorting]
---

- 문제 링크: https://leetcode.com/problems/can-make-arithmetic-progression-from-sequence/
- 파일 경로: `src/main/java/coding_test/정렬/LC1502_CanMakeArithmeticProgressionFromSequence.java`
- 난이도: Easy

[1385번]({% post_url 2026-09-15-distance-value-lc1385 %})에서 배열 인덱스 경계 처리 때문에 크게 헤맨 뒤, 일부러 이진 탐색이 아닌 문제를 하나 골랐다. "인접한 두 인덱스(`i`, `i+1`)를 안전하게 비교하는 감각"만 따로 연습하고 싶어서다.

## 문제 설명

정수 배열 `arr`이 주어질 때, 순서를 재배치해서 등차수열(연속된 원소 차이가 전부 같은 수열)로 만들 수 있으면 `true`, 없으면 `false`를 반환한다.

```
Input: arr = [3,5,1]
Output: true  // [1,3,5]로 재배치하면 차이가 전부 2
```

## 풀이

정렬하면 재배치 문제가 "정렬된 상태에서 인접한 차이가 전부 같은가"로 단순해진다. 그래서 정렬 후, 첫 번째 차이를 기준값으로 잡고 나머지 인접 쌍들과 비교하면 된다.

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

이번엔 크래시 없이 한 번에 통과했다. 반복문 범위를 `i < arr.length - 1`로 잡아서 `arr[i+1]`이 항상 배열 안에 있도록 처음부터 신경 썼다.

## 오늘 배운 내용

**"인접한 두 인덱스를 비교하는 반복문"에서는 반복문의 끝 조건(`i < length - 1`)이 핵심이다.** `i+1`을 참조하는 순간, 반복문이 어디까지 돌아야 안전한지부터 확정하고 시작해야 한다. 1385번에서는 이걸 놓쳐서 여러 번 크래시가 났는데, 이번엔 처음부터 그 부분을 먼저 생각하고 짜서 문제없이 통과했다.

## 오답노트

이번엔 딱히 틀린 게 없었다. 대신 "왜 이번엔 안 틀렸는가"를 남긴다 — 코드를 짜기 전에 "이 반복문에서 배열 밖을 건드릴 수 있는 인덱스가 있는가"를 먼저 점검했기 때문이다. 1385번의 경험이 바로 다음 문제에 적용된 셈이다.
