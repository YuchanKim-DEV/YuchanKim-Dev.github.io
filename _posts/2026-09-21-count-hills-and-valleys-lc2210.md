---
title: "[LeetCode-2210] 연속된 중복값은 압축해서 하나로 묶고 시작한다"
date: 2026-09-21 16:00:00 +0900
categories: [알고리즘 연습, 배열]
tags: [java, leetcode, array]
---

- 문제 링크: https://leetcode.com/problems/count-hills-and-valleys-in-an-array/
- 파일 경로: `src/main/java/coding_test/배열_리스트/LC2210_CountHillsAndValleysInAnArray.java`
- 난이도: Easy

## 문제 설명

정수 배열 `nums`가 주어진다. 인덱스 `i`의 "가장 가까운, 값이 다른 양쪽 이웃"이 둘 다 `nums[i]`보다 작으면 언덕(hill), 둘 다 크면 골짜기(valley)다. 인접한 값이 같으면 같은 언덕/골짜기로 취급한다. 배열 안의 언덕과 골짜기 총 개수를 반환한다.

```
Input: nums = [2,4,1,1,6,5]
Output: 3  // 인덱스1(언덕), 인덱스2-3(골짜기), 인덱스4(언덕)
```

## 시행착오

처음엔 "바로 옆 원소가 같으면 무시하고, 다르면 몇 칸 떨어진 이웃까지 봐야 하는지"를 `if` 조건으로 직접 풀어보려다가 분기가 너무 늘어나서 막혔다. 그래서 배열을 순회하기 전에, **연속으로 같은 값이 나오면 하나로 압축한 배열을 먼저 만드는** 방법을 썼다 — 압축하고 나면 인접한 두 값은 항상 서로 다르다는 게 보장되니까, 그 위에서는 양옆 이웃과 그냥 부등호로 비교만 하면 된다.

```java
List<Integer> compressed = new ArrayList<>();
for (int n : nums) {
    if (compressed.isEmpty() || compressed.get(compressed.size() - 1) != n) {
        compressed.add(n);
    }
}
```

이 압축까지는 잘 짰는데, 제출 후 `[44,44,...,44,40,40]`(44가 28개, 40이 2개)에서 `IndexOutOfBoundsException`이 났다. 이 입력을 압축하면 `[44, 40]`, 길이 2짜리 배열이 되는데, 루프 시작 전에 미리 써둔

```java
int prev = compressed.get(0);
int next = compressed.get(2);
```

이 두 줄에서 `compressed.get(2)`가 존재하지 않는 인덱스를 요청해버린 것이다. 원인은 "압축한 배열의 길이가 3보다 짧을 수도 있다"는 걸 고려하지 않고, 원본 배열 길이(`nums.length`)만 검사한 뒤 안심하고 있었던 것 — 압축하면 길이가 줄어드니까, 압축 후에도 길이를 다시 확인해야 했다. `compressed.size() <= 2`일 때 바로 `0`을 반환하는 가드를 추가해서 해결했다.

## 최종 코드

```java
public int countHillValley(int[] nums) {
    if (nums.length <= 2) {
        return 0;
    }
    int ans = 0;

    List<Integer> compressed = new ArrayList<>();
    for (int n : nums) {
        if (compressed.isEmpty() || compressed.get(compressed.size() - 1) != n) {
            compressed.add(n);
        }
    }

    if (compressed.size() <= 2) {
        return 0;
    }

    for (int i = 1; i < compressed.size() - 1; i++) {
        int prev = compressed.get(i - 1);
        int next = compressed.get(i + 1);
        if (prev > compressed.get(i) && next > compressed.get(i)) {
            ans++;
        } else if (prev < compressed.get(i) && next < compressed.get(i)) {
            ans++;
        }
    }

    return ans;
}
```

## 오늘 배운 내용

**입력 길이를 검사할 땐, 그 검사가 "지금 이 시점의" 길이를 보고 있는지 확인해야 한다.** `nums.length`로 한 번 검사했다고 해서, 그 뒤에 가공(압축, 필터링 등)을 거친 배열의 길이까지 안전하다고 착각하면 안 된다. 원본 배열과 가공된 배열은 길이가 다를 수 있고, 가공 이후에 다시 접근하는 코드가 있다면 그 지점에서 다시 한번 길이를 검사해야 한다.

## 오답노트

- **틀렸던 패턴**: `nums.length <= 2` 검사만 하고, 압축된 `compressed` 배열의 길이는 따로 검사하지 않은 채 `compressed.get(2)`를 호출.
- **왜 틀렸나**: 원본 배열이 충분히 길어도, 중복값이 많으면 압축 후 배열이 훨씬 짧아질 수 있다는 걸 놓쳤다. "이미 길이를 확인했다"는 느낌만 갖고, 그게 지금 접근하려는 배열(압축본)과 같은 배열인지 확인하지 않았다.
- **고친 패턴**: 압축을 마친 직후 `compressed.size() <= 2`를 다시 확인하는 가드 추가.
- **다음에 떠올릴 시점**: 배열을 가공(압축/필터링/변형)하는 코드가 있으면, 그 가공된 배열에 인덱스로 접근하기 전에 "이 배열의 길이를 검사한 적이 있는가"를 원본이 아니라 가공된 버전 기준으로 다시 확인한다.

## AI라면 어떻게 풀었을까

압축(`ArrayList` 생성)이라는 별도 자료구조를 만들지 않고, 인덱스 포인터만으로 "다음/이전의 다른 값"을 즉석에서 찾아내는 방식도 가능하다.

```java
public int countHillValley(int[] nums) {
    int n = nums.length;
    int count = 0;
    int left = nums[0];

    for (int i = 1; i < n - 1; i++) {
        if (nums[i] == nums[i + 1]) {
            continue;
        }
        if ((left < nums[i] && nums[i] > nums[i + 1]) ||
            (left > nums[i] && nums[i] < nums[i + 1])) {
            count++;
        }
        left = nums[i];
    }
    return count;
}
```

`left`라는 변수 하나에 "가장 최근에 본 다른 값"만 기억해두고, `nums[i] == nums[i+1]`(오른쪽이 같은 값)이면 그 자리는 건너뛴다. `ArrayList`를 새로 만들지 않아서 메모리를 덜 쓰고, 원본 배열의 인덱스를 그대로 쓸 수 있다는 장점이 있다. 다만 "왼쪽의 다른 값"과 "오른쪽 경계 판단"을 동시에 다루다 보니 코드를 처음 읽을 땐 압축 방식보다 더 헷갈릴 수 있다 — 압축 방식은 "먼저 문제를 단순하게 만들고 그다음 푼다"는 전략이고, 이 방식은 "단순화 없이 한 번에 처리한다"는 전략의 차이다.
