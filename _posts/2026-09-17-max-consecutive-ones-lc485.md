---
title: "[LeetCode-485] 반복문이 끝난 뒤 남은 구간을 놓치지 않으려면"
date: 2026-09-17 20:00:00 +0900
categories: [알고리즘 연습, 배열]
tags: [java, leetcode, array]
---

- 문제 링크: https://leetcode.com/problems/max-consecutive-ones/
- 파일 경로: `src/main/java/coding_test/배열_리스트/LC0485_MaxConsecutiveOnes.java`
- 난이도: Easy

## 문제 설명

0과 1로만 이루어진 배열 `nums`가 주어진다. 이 배열에서 1이 연속으로 나오는 구간 중 가장 긴 것의 길이를 반환한다.

```
Input: nums = [1,1,0,1,1,1]
Output: 3
```

## 시행착오

1을 만날 때마다 `count`를 늘리고, 0을 만나면 그때까지의 `count`를 `max`와 비교해서 갱신한 뒤 `count`를 0으로 리셋하는 구조로 짰다.

짜는 도중에 "배열이 1로 끝나버리면 어떻게 되지?"라는 질문이 스스로 떠올랐다. 0을 만나야 `max` 갱신이 일어나는데, 배열 마지막이 1이라면 0을 영영 안 만나고 루프가 끝나버려서 `max` 갱신이 한 번도 안 일어날 수 있다. 그래서 루프가 끝난 다음에 남은 `count`를 마지막으로 한 번 더 `max`와 비교하는 코드를 추가했다.

다만 처음엔 `Math.max(max, count);`만 써놓고 반환값을 `max`에 대입하지 않아서 값이 그대로였다 — `Math.max`는 새 값을 만들어서 반환할 뿐, 대입해야 실제로 반영된다는 걸 다시 확인했다.

## 최종 코드

```java
public int findMaxConsecutiveOnes(int[] nums) {
    int count = 0;
    int max = 0;
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] == 1) {
            count += 1;
        } else {
            max = Math.max(max, count);
            count = 0;
        }
    }
    if (max < count) {
        max = count;
    }
    return max;
}
```

## 오늘 배운 내용

**루프 안에서만 갱신되는 값은, 루프가 끝난 뒤에도 한 번 더 처리해줘야 하는 경우가 있다.** 이 문제에서는 "0을 만났을 때"만 `max`를 갱신했기 때문에, 배열이 1로 끝나면 마지막 구간이 누락된다. 루프 종료 직후에 남은 값을 한 번 더 반영하는 패턴을 여기서 처음 의식적으로 챙겼다.

## 오답노트

- **틀렸던 패턴**: `Math.max(max, count);`만 쓰고 반환값을 변수에 대입하지 않음.
- **왜 틀렸나**: `Math.max`는 순수 함수라 인자를 바꾸지 않고 결과를 반환만 한다. `max = Math.max(max, count);`처럼 대입해야 값이 실제로 바뀐다.
- **고친 패턴**: 모든 `Math.max` 호출을 대입문으로 감싸도록 확인.
- **다음에 떠올릴 시점**: 반환값이 있는 메서드를 호출할 때마다 "이 결과를 어딘가에 저장하고 있는가"를 습관적으로 점검한다.
