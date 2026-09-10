---
title: "찾자마자 리턴이 안 되는 이진 탐색도 있다 - First Bad Version에서 제일 오래 헤맨 이유"
date: 2026-09-09 21:00:00 +0900
categories: [코딩테스트, 이진탐색]
tags: [java, leetcode, binary-search]
---

- 문제 링크: https://leetcode.com/problems/first-bad-version/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0278_FirstBadVersion.java`
- 난이도: Easy

## 문제 설명

버전 `1..n`이 있고, 어떤 버전부터 불량이면 그 이후 버전은 전부 불량이다. `isBadVersion(version)`이라는 API(예/아니오만 반환)를 최소 횟수로 호출해서 **첫 번째 불량 버전**을 찾아야 한다.

```
Input: n = 5, bad = 4  -> Output: 4
Input: n = 1, bad = 1  -> Output: 1
```

이 문제는 실제 채점 환경에서 `isBadVersion`이 미리 주어지는 API라, 로컬 테스트를 위해 별도로 흉내 낸 버전을 만들어야 했다:

```java
static int bad = 4;
public static boolean isBadVersion(int version) {
    return version >= bad;
}
```

## 시행착오

오늘 세션에서 가장 오래 걸린 문제. 실수가 여러 겹으로 있었다.

### 1차 시도 — 발견하자마자 바로 리턴

```java
if (isBadVersion(mid)) {
    return mid;
}
```

`bad = 1, n = 5`로 테스트하니 `mid = 2`에서 `isBadVersion(2)`가 true라서 바로 `2`를 리턴했다. 정답은 `1`인데. `mid`가 불량이라는 게 "이게 첫 불량"이라는 뜻은 아니었다 — 더 왼쪽에 더 이른 불량이 있을 수 있는데 확인도 안 하고 리턴해버린 것.

### 2차 시도 — mid와 mid-1을 같이 확인

```java
if (isBadVersion(mid) && !isBadVersion(mid - 1)) {
    return mid;
} else if (isBadVersion(mid) && isBadVersion(mid - 1)) {
    n = mid - 1;
} else if (!isBadVersion(mid) && !isBadVersion(mid - 1)) {
    prev = mid + 1;
}
```

이건 통과는 했지만, 반복마다 `isBadVersion`을 최대 2번씩 호출해서 문제가 요구하는 "API 호출 최소화" 조건에 안 맞았다.

## 깨달은 것 — "즉시 리턴이 없는" 이진 탐색 패턴

704, 35와 달리 이 문제는 "딱 맞다"는 조건 자체가 없다 (`isBadVersion`은 true/false 둘 뿐). 이럴 때는:

- `isBadVersion(mid)`가 **true** → mid는 불량. 첫 불량은 mid 자신이거나 더 왼쪽. **오른쪽은 볼 필요 없음**, 그러나 mid 자신은 후보로 남겨야 함 → `right = mid` (mid-1 아님!)
- `isBadVersion(mid)`가 **false** → mid는 확실히 답이 아님 → `right = mid - 1`이 아니라 `left = mid + 1`

루프 조건도 `left <= right`가 아니라 `left < right`로 바꿔야 한다 — `right`가 mid 자신을 후보로 계속 남기고 있어서, `left == right`가 될 때까지만 좁히면 된다.

## 최종 코드

```java
static int bad = 4;

public static boolean isBadVersion(int version) {
    return version >= bad;
}

public static int firstBadVersion(int n) {
    int left = 0;
    int right = n;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (isBadVersion(mid)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```

## 변수 추적표 (n = 5, bad = 1 — 첫 버전부터 불량)

| 반복 | left | right | mid | isBadVersion(mid) | 갱신 |
|---|---|---|---|---|---|
| 1 | 0 | 5 | 0+(5-0)/2=**2** | true (2≥1) | right = 2 |
| 2 | 0 | 2 | 0+(2-0)/2=**1** | true (1≥1) | right = 1 |
| 3 | 0 | 1 | 0+(1-0)/2=**0** | false (0<1) | left = 1 |
| — | 1 | 1 | `left(1) < right(1)`? 거짓 → 종료 | | |

루프 종료 시 `left == right == 1` → `return left` = `1` (정답)

## 한 줄 오답노트

> "딱 맞음"이라는 탈출 조건이 없는 이진 탐색은 `right = mid`(후보 유지), `left = mid+1`(확실히 제외), 루프 조건은 `left < right`, 답은 `left`. First Bad Version이 이 패턴의 대표 문제.
