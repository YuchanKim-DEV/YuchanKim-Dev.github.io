---
title: "[프로그래머스-Lv2] 최솟값 만들기 - 작은 것은 큰 것과 짝짓는다"
date: 2026-10-26 10:29:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, greedy]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/12941
- 파일 경로: `src/main/java/coding_test/정렬/Lv2_최솟값만들기.java`
- 난이도: Lv.2

## 문제 설명

길이가 같은 배열 A, B에서 하나씩 뽑아 곱하고, 그 곱을 누적해 더한다. 한 번 뽑은 칸은 다시 못 뽑는다. 누적 합이 **최소**가 되도록 짝을 지었을 때의 합을 구한다.

```
A = [1, 4, 2], B = [5, 4, 4]  → 29   (1×5 + 2×4 + 4×4)
A = [1, 2],    B = [3, 4]     → 10   (1×4 + 2×3)
```

Comparator 드릴을 마치고 정렬 Lv2로 돌아온 첫 문제다.

## 시행착오

### 1차: `int[]`에 Comparator, lambda 안에서 `B[b]`

방향은 처음부터 맞았다. **A는 오름차순, B는 내림차순으로 정렬해 같은 칸끼리 곱한다.** 그런데 컴파일 오류가 났다.

```java
Arrays.sort(B, (a, b) -> B[b] - B[a]);
```

틀린 곳이 두 개였다.

1. `B`는 `int[]`다. Comparator를 받는 `Arrays.sort`는 객체 배열(`Integer[]`)에서만 쓸 수 있다.
2. 어제 [LC2418]({% post_url 2026-10-25-sort-the-people-lc2418 %})에서는 **번호표**를 정렬해서 `heights[b]`처럼 다른 배열을 봤다. 이번에는 B의 **값**을 직접 정렬하니까 `a`, `b`가 이미 값이다. 어제 패턴이 그대로 따라 나왔다.

### 2차: 통과

`Integer[]`로 옮기고 `(a, b) -> b - a`로 정렬해 7개 테스트를 모두 통과했다.

## 최종 코드

```java
public int solution(int[] A, int[] B) {
    Arrays.sort(A);                                   // A 오름차순
    Integer[] b_temp = new Integer[B.length];
    for (int i = 0; i < B.length; i++) {
        b_temp[i] = B[i];
    }
    Arrays.sort(b_temp, (a, b) -> b - a);             // B 내림차순

    int answer = 0;
    for (int i = 0; i < A.length; i++) {
        answer += A[i] * b_temp[i];                   // 작은 A × 큰 B
    }
    return answer;
}
```

## 추적표: `A = [1, 4, 2], B = [5, 4, 4]`

| i | A (오름차순) | B (내림차순) | 곱 | 누적 |
|---|---|---|---|---|
| 0 | 1 | 5 | 5 | 5 |
| 1 | 2 | 4 | 8 | 13 |
| 2 | 4 | 4 | 16 | 29 |

## 오늘 배운 내용

**합을 최소로 만들려면 큰 수끼리 만나지 않게 한다.** 큰 수끼리 곱하면 값이 확 커진다. 그래서 A의 가장 큰 수를 B의 가장 작은 수와 짝지어야 한다. 한쪽은 오름차순, 다른 쪽은 내림차순으로 정렬하면 같은 칸끼리 자동으로 그렇게 짝지어진다.

## 오답노트

- **틀렸던 지점**: `Arrays.sort(B, (a, b) -> B[b] - B[a])`. **왜**: ① `int[]`에 Comparator를 넣음 ② 어제 인덱스 정렬처럼 lambda 안에서 배열을 다시 봄. **어떻게 고쳤나**: `Integer[]`로 옮기고 `(a, b) -> b - a`로 고쳤다. **다음에**: lambda를 쓰기 전에 두 가지를 본다. "정렬할 배열이 `Integer[]`인가?", "`a`, `b`가 값인가 번호인가?"

## 꿀팁

**내림차순이 필요해도 꼭 내림차순으로 정렬할 필요는 없다.** 오름차순으로 정렬해 두고 **뒤에서부터** 읽으면 똑같다. `int[]` 그대로 `Arrays.sort`를 쓸 수 있어 `Integer[]` 변환도 필요 없다.

**버블·퀵·머지 정렬을 직접 짜야 할까?** 코딩테스트에서는 거의 직접 짜지 않는다. `Arrays.sort`를 쓴다. 다만 면접에서는 다음 정도를 물어볼 수 있다.
- 시간복잡도: 버블 O(n²), 퀵 평균 O(n log n)·최악 O(n²), 머지 O(n log n)
- 자바 `Arrays.sort`가 쓰는 방식: `int[]`는 Dual-Pivot Quicksort, `Integer[]`·`String[]`은 TimSort(머지 계열이라 **같은 값의 순서가 유지됨 = stable**)

## AI라면 어떻게 풀었을까

**둘 다 오름차순으로 정렬하고 B는 뒤에서부터 꺼낸다.** Comparator와 `Integer[]` 변환이 모두 사라진다.

```java
public int solutionAlt(int[] A, int[] B) {
    Arrays.sort(A);
    Arrays.sort(B);
    int n = A.length;
    int answer = 0;
    for (int i = 0; i < n; i++) {
        answer += A[i] * B[n - 1 - i];   // A 작은 것 × B 큰 것
    }
    return answer;
}
```

트레이드오프: 코드가 짧고, 박싱이 없어 더 빠르다. n이 1,000 이하라 이 문제에서는 둘 다 충분히 빠르지만, n이 커지면 `int[]` 쪽이 유리하다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
