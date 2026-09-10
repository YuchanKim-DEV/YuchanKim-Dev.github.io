---
title: "[LeetCode-278] 즉시 리턴 없는 이진 탐색 패턴, 안 보고 다시 짜서 정착 확인하기"
date: 2026-09-10 21:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search, 복습]
---

- 문제 링크: https://leetcode.com/problems/first-bad-version/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0278_FirstBadVersion.java`
- 난이도: Easy

## 문제 설명

버전 `1..n`이 있고, 어떤 버전부터 불량이면 그 이후 버전은 전부 불량이다. `isBadVersion(version)` API를 최소 횟수로 호출해서 첫 번째 불량 버전을 찾는다. (자세한 문제 설명은 [어제 글](/posts/first-bad-version-lc278/) 참고)

어제 이 문제를 풀 때 오답노트에 남긴 한 줄은 이거였다:

> "딱 맞음"이라는 탈출 조건이 없는 이진 탐색은 `right = mid`(후보 유지), `left = mid+1`(확실히 제외), 루프 조건은 `left < right`, 답은 `left`.

오늘은 이 패턴이 진짜 몸에 붙었는지 확인하려고, 어제 코드를 보지 않고 백지에서 다시 짜봤다.

## 시행착오

이번엔 힌트도, 어제 코드도 안 보고 짰다. 결과적으로 로직 자체는 한 번에 맞았지만, 다시 짠 코드에 아래처럼 불필요한 조건이 하나 남았다.

```java
if (isBadVersion(mid)) {
    right = mid;
} else if (!isBadVersion(mid)) {   // else로 충분한데 조건을 다시 씀
    left = mid + 1;
}
```

`if`에서 이미 `isBadVersion(mid)`가 거짓인 경우만 `else`로 넘어오기 때문에 `else if (!isBadVersion(mid))`의 조건은 항상 참이다. 동작에는 문제가 없지만, "이미 걸러진 조건을 왜 또 확인하려 했는가"를 생각해보니 `else`만으로 충분하다는 걸 놓치고 있었다.

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

## 오늘 배운 내용

가장 중요한 확인 사항은 **"즉시 리턴 조건이 없는 이진 탐색" 패턴 자체(`right=mid`, `left=mid+1`, `left<right`, 답은 `left`)는 안 보고도 재현할 수 있을 만큼 정착됐다**는 것이다. 다만 조건문을 "이미 걸러진 경우를 또 조건으로 확인하는" 습관이 아직 남아있다는 걸 새로 발견했다.

## 오답노트

- **여전히 남아있는 버릇**: `if`/`else`로 완전히 나뉘는 이분 조건인데도 `else if (!조건)`처럼 조건을 한 번 더 써버리는 습관. 이미 `if`에서 걸러진 경우이므로 `else`만으로 충분하다.
- **왜 자꾸 이러나 생각해보면**: 조건이 복잡했던 초기 시도(어제 2차 시도의 `isBadVersion(mid) && !isBadVersion(mid-1)`처럼 여러 조건을 AND로 묶던 버릇)가 아직 몸에 남아서, 단순한 이분 조건에도 반사적으로 조건을 다시 쓰게 되는 것 같다.
- **다음에 확인할 것**: 이진 탐색 조건문을 짤 때 "이 분기가 정말 두 가지로 완전히 나뉘는가"를 먼저 확인하고, 그렇다면 `else`만 쓰는 연습을 의식적으로 한다.

## 꿀팁

- 안 보고 다시 짜보는 복습은 "패턴을 기억하는지"뿐 아니라 "코드 스타일의 남은 버릇"까지 드러내준다. 정답을 맞혔다고 끝내지 말고, 코드를 한 번 더 리뷰하는 습관이 도움이 된다.
