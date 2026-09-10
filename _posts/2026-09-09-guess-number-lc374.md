---
title: "오늘 처음으로 힌트 없이 이진 탐색을 풀었다 (LeetCode 374)"
date: 2026-09-09 21:00:00 +0900
categories: [코딩테스트, 이진탐색]
tags: [java, leetcode, binary-search]
---

- 문제 링크: https://leetcode.com/problems/guess-number-higher-or-lower/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0374_GuessNumberHigherOrLower.java`
- 난이도: Easy

## 문제 설명

1~n 사이 숫자 하나(`pick`)가 미리 정해져 있다. `guess(num)` API를 호출하면:

- `-1`: 내가 부른 숫자가 정답보다 큼
- `1`: 내가 부른 숫자가 정답보다 작음
- `0`: 정답

정답 숫자를 반환해야 한다.

```
Input: n = 10, pick = 6  -> Output: 6
Input: n = 1, pick = 1   -> Output: 1
Input: n = 2, pick = 1   -> Output: 1
```

## 시행착오 — 이번엔 자력으로 풀었다

오늘 처음으로 힌트 없이 완전히 혼자 풀어낸 문제. 278에서 배운 "즉시 리턴 가능한 조건이 있는 경우엔 704 템플릿을 그대로 쓴다"는 감각을 그대로 적용해서, `nums[mid]` 자리에 `guess(mid)`의 리턴값을 넣는 방식으로 짰다.

풀고 나서 리뷰받은 개선점:

1. **API 호출 중복** — 처음 짠 코드는 `if (guess(mid) == 0)`, `else if (guess(mid) == 1)`처럼 반복마다 `guess()`를 최대 2번 호출했다. `int result = guess(mid);`로 한 번만 호출해서 저장하는 게 맞다 (이 문제도 API 호출을 아끼는 게 중요한 유형).
2. `pick`은 `1 <= pick <= n`이라 `left`를 `0`이 아니라 `1`부터 시작해도 됐다.

## 최종 코드 (자력 구현본)

```java
public int guessNumber(int n) {
    int left = 0;
    int right = n;
    int mid = 0;
    while (left <= right) {
        mid = left + (right - left) / 2;
        if (guess(mid) == 0) {
            return mid;
        } else if (guess(mid) == 1) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return mid;
}
```

## 변수 추적표 (n = 10, pick = 6)

| 반복 | left | right | mid | guess(mid) | 갱신 |
|---|---|---|---|---|---|
| 1 | 0 | 10 | 0+(10-0)/2=**5** | 1 (5<6) | left = 6 |
| 2 | 6 | 10 | 6+(10-6)/2=**8** | -1 (8>6) | right = 7 |
| 3 | 6 | 7 | 6+(7-6)/2=**6** | 0 (6==6) | `return 6` |

## 한 줄 오답노트

> 704 템플릿에서 배열 비교 자리를 API 호출로 바꾸는 감각을 처음으로 자력 적용해봄. 다만 비용이 드는 API는 한 번만 호출해서 결과를 변수에 저장하는 습관을 들여야 한다.
