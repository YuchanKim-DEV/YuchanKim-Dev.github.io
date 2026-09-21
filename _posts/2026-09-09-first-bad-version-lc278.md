---
title: "[LeetCode-278] \"찾자마자 리턴\"이 안 되는 이진 탐색 패턴 - right=mid, left<right로 후보를 좁힌다"
date: 2026-09-09 21:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
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

## 오늘 배운 내용 — "즉시 리턴이 없는" 이진 탐색 패턴

이 문제에서 가장 중요한 내용은, 이진 탐색에 두 가지 다른 유형이 있다는 것이다. 704, 35와 달리 이 문제는 "딱 맞다"는 조건 자체가 없다 (`isBadVersion`은 true/false 둘 뿐). 이럴 때는:

- `isBadVersion(mid)`가 **true** → mid는 불량. 첫 불량은 mid 자신이거나 더 왼쪽. **오른쪽은 볼 필요 없음**, 그러나 mid 자신은 후보로 남겨야 함 → `right = mid` (mid-1 아님!)
- `isBadVersion(mid)`가 **false** → mid는 확실히 답이 아님 → `right = mid - 1`이 아니라 `left = mid + 1`

루프 조건도 `left <= right`가 아니라 `left < right`로 바꿔야 한다 — `right`가 mid 자신을 후보로 계속 남기고 있어서, `left == right`가 될 때까지만 좁히면 된다.

## 오답노트

- **1차 시도의 문제**: "조건을 만족하면 그게 곧 답"이라고 착각했다. 하지만 이 문제의 조건(`isBadVersion(mid) == true`)은 "mid가 답 후보"라는 뜻이지 "mid가 정답"이라는 뜻이 아니다. 더 왼쪽에도 만족하는 값이 있을 수 있으므로 즉시 리턴하면 안 된다.
- **2차 시도의 문제**: 정답 자체는 맞았지만 접근 방식이 비효율적이었다. `mid`와 `mid-1`을 동시에 비교하는 방식은 매 반복 API를 최대 2번 호출하게 되어, "API 호출 최소화"라는 이 문제의 진짜 요구사항을 놓쳤다.
- **최종적으로 정착된 패턴**: "즉시 리턴 조건이 없는" 이진 탐색은 `right = mid`(후보를 유지한 채 좁히기), `left = mid + 1`(확실히 아닌 것만 제외), 루프 조건은 `left < right`, 답은 `left`. First Bad Version이 이 패턴의 대표 문제이고, 앞으로 "예/아니오만 주는 API"나 "조건을 만족하는 첫 위치를 찾는" 유형을 만나면 이 템플릿부터 떠올린다.
- **디버깅 팁**: `bad`를 다양한 값(1, n, 중간값)으로 바꿔가며 변수 추적표를 직접 그려본 게 이 패턴이 왜 성립하는지 이해하는 데 제일 도움이 됐다.

## 꿀팁

- 이 문제처럼 "이 값 이상은 전부 조건을 만족한다"는 이분법적 구조를 가진 문제는 대부분 `right = mid` / `left = mid + 1` / `left < right` 템플릿으로 풀린다. 문제를 읽을 때 "만족/불만족이 어떤 경계를 기준으로 완전히 갈리는가"부터 확인하는 습관을 들이면 유형 판별이 빨라진다.
- 비용이 큰(또는 호출 횟수 제한이 있는) API를 다루는 문제는 "한 반복에 API를 몇 번 부르는가"를 항상 세어보자. 정답이어도 호출 횟수 조건 때문에 틀리는 경우가 있다.

## AI라면 어떻게 풀었을까

이 문제는 API 호출 기반이라 라이브러리로 대체할 수 없지만, 템플릿 자체를 하나 더 알아두면 좋다. 지금 짠 코드는 `left < right`로 좁혀가다가 `left`(==`right`)를 반환하는 "즉시 리턴 없음" 템플릿인데, 같은 문제를 `left <= right`를 유지하면서 정답 후보를 별도 변수(`ans`)에 저장해두는 방식으로도 풀 수 있다.

```java
int left = 1, right = n, ans = n;
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (isBadVersion(mid)) {
        ans = mid;
        right = mid - 1;
    } else {
        left = mid + 1;
    }
}
return ans;
```

두 템플릿은 결과가 같지만, `ans` 변수를 쓰는 쪽이 "정답 후보를 계속 갱신한다"는 의도가 코드에 더 명시적으로 드러난다. 나중에 복잡한 파라메트릭 서치를 짤 때는 이 `ans` 변수 방식이 디버깅하기 더 쉬운 경우가 많다.
