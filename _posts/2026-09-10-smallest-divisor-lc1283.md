---
title: "[LeetCode-1283] 파라메트릭 서치 응용 - divisor를 후보값으로 이진 탐색하기"
date: 2026-09-10 21:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search, parametric-search]
---

- 문제 링크: https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC1283_FindTheSmallestDivisorGivenAThreshold.java`
- 난이도: Medium

## 문제 설명

정수 배열 `nums`와 `threshold`가 주어진다. 양의 정수 `divisor`를 하나 골라 배열의 모든 원소를 그 값으로 나눈 뒤(몫은 올림 처리) 다 더한 값이 `threshold` 이하가 되도록 하는 **가장 작은 `divisor`**를 구한다.

```
Input: nums = [1,2,5,9], threshold = 6      -> Output: 5
Input: nums = [44,22,33,11,1], threshold = 5 -> Output: 44
```

## 시행착오

이 문제도 875와 마찬가지로 이미 완성된 코드로 확인해서, 실제 겪은 시행착오 기록은 없다. 다만 875를 막 풀고 난 직후라 어떤 부분을 그대로 재사용했는지는 코드에서 뚜렷하게 보인다.

- 875에서 "속도 k"였던 후보값이 여기서는 "divisor"로 바뀌었을 뿐, 구조는 동일하다.
- 판정 함수도 "올림 나눗셈의 합이 threshold 이하인가"로 875의 "올림 나눗셈의 합이 h 이하인가"와 같은 모양이다.
- 다만 루프 종료 조건과 반환값을 875와 다르게 짰다 — 875는 `left <= right` / `right = mid - 1`인데, 이 문제는 `left < right` / `right = mid`로, 278에서 쓴 "즉시 리턴 없는" 템플릿 쪽을 가져다 썼다.

## 최종 코드

```java
public static int smallestDivisor(int[] nums, int threshold) {
    int left = 1;
    int right = nums[0];
    for (int n : nums) {
        if (n > right) {
            right = n;
        }
    }

    while (left < right) {
        int mid = left + (right - left) / 2;
        int idx = 0;
        long sums = 0;
        while (!(idx == nums.length)) {
            sums += (nums[idx] + mid - 1) / mid;
            idx++;
        }

        if (sums > threshold) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }

    return left;
}
```

## 변수 추적표 (nums = [1,2,5,9], threshold = 6)

| 반복 | left | right | mid | 나눗셈 합 | 비교 | 갱신 |
|---|---|---|---|---|---|---|
| 1 | 1 | 9 | 1+(9-1)/2=**5** | 1+1+1+2=**5** | 5 <= 6 | right = 5 |
| 2 | 1 | 5 | 1+(5-1)/2=**3** | 1+1+2+3=**7** | 7 > 6 | left = 4 |
| 3 | 4 | 5 | 4+(5-4)/2=**4** | 1+1+2+3=**7** | 7 > 6 | left = 5 |
| — | 5 | 5 | `left(5) < right(5)`? 거짓 → 종료 | | | |

루프 종료 시 `left = 5` → 정답 `5`

## 오늘 배운 내용

이 문제에서 가장 중요한 내용은 **파라메트릭 서치 패턴이 "판정 함수만 바꾸면" 다른 문제에도 그대로 재사용된다**는 것이다. 875에서 익힌 틀("후보값을 이진 탐색 + 후보값마다 배열 전체를 순회해 조건 계산")을 이 문제에서는 `divisor`에 그대로 적용했다. 다만 종료 조건 템플릿은 875(`left<=right`, `right=mid-1`)가 아니라 278(`left<right`, `right=mid`)의 것을 썼는데, 이는 "이 문제도 조건을 만족하는 가장 작은 값을 찾는" 유형이라 즉시 리턴 조건이 없는 패턴과 궁합이 맞기 때문이다.

## 오답노트

- **두 이진 탐색 템플릿을 섞어 쓸 때 주의할 점**: 같은 파라메트릭 서치라도 "종료 조건이 있는 템플릿"(`left<=right`, `right=mid-1`/`left=mid+1`, 875 방식)과 "즉시 리턴 조건이 없는 템플릿"(`left<right`, `right=mid`, 278 방식)이 있다. 어떤 걸 쓰든 결과는 같지만, 한 문제 안에서 두 스타일을 섞으면 `right`를 `mid`로 둘지 `mid-1`로 둘지 헷갈리기 쉽다 — 템플릿을 고르면 끝까지 일관되게 써야 한다.
- **판정 함수 재사용은 좋지만 자동 검증이 필요하다**: "875랑 구조가 같으니 그대로 쓰면 되겠지"라고 넘어가지 말고, 예제 입력으로 직접 손 계산(변수 추적표)을 해서 실제로 맞는지 검증하는 과정이 필요하다.

## 꿀팁

- 파라메트릭 서치 문제를 연달아 풀 때는 "이번 문제의 후보값은 무엇이고, 판정 함수는 무엇인가"부터 먼저 적어보면 이전에 푼 문제의 템플릿을 빠르게 재사용할 수 있다.
- 이진 탐색 템플릿은 한 가지로 통일해서 익혀두는 편이 헷갈림을 줄인다. 지금까지는 "즉시 리턴 있음"(704/35 방식)과 "즉시 리턴 없음"(278 방식) 두 가지를 상황에 맞게 골라 썼는데, 앞으로는 파라메트릭 서치에는 기본적으로 "즉시 리턴 없음" 템플릿을 쓰는 쪽으로 통일해볼 계획이다.
