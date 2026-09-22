---
title: "[LeetCode-875] 이진 탐색이 어려워서 배열로 돌아갔다가, 다시 백지로 재도전"
date: 2026-09-22 14:20:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search]
---

> 이진 탐색 개념과 패턴 정리는 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다. 이 문제의 첫 풀이 글은 [9/10 글]({% post_url 2026-09-10-koko-eating-bananas-lc875 %})에 있다 — 이번은 백지 재작성 검증 글이다.

- 문제 링크: https://leetcode.com/problems/koko-eating-bananas/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0875_KokoEatingBananas.java`
- 난이도: Medium

## 배경 — 왜 다시 풀었나

9/17에 LeetCode 1539(Kth Missing Positive Number)에서 완전히 막혔다. 감독관 진단은 "이진 탐색을 모르는 게 아니라, 반복문을 스스로 설계하는 근육 자체가 없다"였다. 그래서 이진 탐색을 일시 중단하고, `배열_리스트`/`구현` 폴더의 Easy 문제로 우회해서 그 근육부터 다졌다(485, 1550, 674, 896, 228, 941, 1752, 2016, 1909, 665 — 총 10문제).

9/22, 배열 배치를 다 끝내고 원래 계획대로 이진 탐색으로 복귀했다. 875(Koko Eating Bananas)와 1011(Capacity To Ship Packages)은 9/10, 9/14에 통과는 했지만 힌트를 매우 많이 받아서 "백지 재작성 검증"이 안 된 상태로 남아있었던 문제들이다. 기존 풀이 코드를 완전히 지우고 문제 설명만 남긴 채로 다시 도전했다.

## 문제 설명

바나나 더미가 n개 있고, i번째 더미에는 `piles[i]`개가 있다. h시간 안에 전부 먹어야 하고, 시간당 먹는 속도 k를 정하면 매 시간 한 더미에서 k개씩 먹는다(부족하면 그 더미를 다 먹고 그 시간은 끝). h시간 안에 다 먹을 수 있는 **최소 속도 k**를 구한다.

```
Input: piles = [3,6,7,11], h = 8
Output: 4
```

## 시행착오

**탐색 범위 갱신**: 처음엔 `right = right - 1`, `left = left + 1`처럼 `mid`를 전혀 쓰지 않고 경계를 그냥 1씩 옮기는 실수를 했다. `mid`를 확인해서 가능/불가능을 판정했으면, 그 판정을 반영해서 `right = mid - 1` / `left = mid + 1`로 갱신해야 하는데, 판정 결과와 갱신이 연결이 안 되어 있었다. 이건 다행히 스스로 문제를 알아채고 고쳤다.

**시간 계산(올림 나눗셈)**: `hours += Math.ceil(piles[i] / mid);`라고 썼는데, 통과하지 못했다. `piles[i]`와 `mid`가 둘 다 `int`라서 나누는 순간 이미 정수 나눗셈으로 소수점이 사라지고, 그 뒤에 `Math.ceil`을 씌워도 이미 버려진 소수점은 못 살린다는 걸 놓쳤다. `(double) piles[i] / mid`로 캐스팅해야 `Math.ceil`이 의미가 있다는 걸 지적받고 고쳤다.

**특수 케이스가 일반 로직을 가림**: 캐스팅을 고쳤는데도 계속 틀렸다. 원인은 `piles.length == 1`이거나 `piles.length == h`일 때 답을 바로 계산해서 반환하는 특수 케이스 코드를 따로 만들어뒀는데, 그 특수 케이스 안에도 똑같이 정수 나눗셈(올림 처리 안 된) 버그가 들어있었던 것 — 게다가 이 특수 케이스 자체가 애초에 필요 없었다. 일반 로직(이진 탐색 + 올림 나눗셈)이 제대로 짜여 있으면 이 케이스들도 알아서 같은 답을 낸다. 특수 케이스 두 개를 통째로 지우고 나서야 통과했다.

## 최종 코드

```java
public static int minEatingSpeed(int[] piles, int h) {
    int left = 1;
    int right = 0;
    for (int n : piles) {
        right = Math.max(n, right);
    }

    while (left <= right) {
        int mid = left + (right - left) / 2;
        int hours = 0;
        for (int i = 0; i < piles.length; i++) {
            hours += Math.ceil((double) piles[i] / mid);
        }

        if (hours <= h) {
            right = mid - 1;
        } else if (hours > h) {
            left = mid + 1;
        }
    }
    return left;
}
```

## 오늘 배운 내용

**"일부 입력에서만 맞는 지름길"을 먼저 만들면, 그 지름길 안에 있는 버그가 일반 로직의 버그와 겹쳐서 원인 파악이 더 어려워진다.** 특수 케이스를 추가하고 싶어지는 순간, "일반 로직이 이 케이스를 이미 처리할 수 있는가"부터 확인하는 게 먼저다. 이번엔 일반 로직만으로 충분했는데도 특수 케이스를 만들었고, 그 특수 케이스 자체에 원래 로직과 똑같은 버그(정수 나눗셈)가 복제돼 있어서 디버깅이 더 꼬였다.

## 오답노트

- **틀렸던 패턴 1**: `right = right - 1`, `left = left + 1`처럼 `mid`를 쓰지 않고 경계를 그냥 옮김.
- **왜 틀렸나**: `mid`로 판정한 결과를 다음 탐색 범위 갱신에 반영하지 않았다.
- **고친 패턴**: `hours <= h`면 `right = mid - 1`, `hours > h`면 `left = mid + 1`.
- **틀렸던 패턴 2**: `Math.ceil(piles[i] / mid)` — 정수 나눗셈이 먼저 실행돼 소수점이 이미 사라진 뒤에 `ceil`을 적용.
- **왜 틀렸나**: `Math.ceil`이 받는 인자가 이미 정수라는 걸 놓쳤다. 연산 순서(나눗셈이 정수 연산인지 실수 연산인지)를 먼저 확인해야 했다.
- **고친 패턴**: `(double) piles[i] / mid`로 캐스팅해서 나눗셈 자체를 실수 연산으로 만듦.
- **틀렸던 패턴 3**: `piles.length == 1`, `piles.length == h` 특수 케이스를 따로 만들었는데, 그 안에도 캐스팅 안 된 버그가 그대로 있었고, 애초에 이 특수 케이스 자체가 불필요했음.
- **다음에 떠올릴 시점**: 일반 로직을 다 짜기 전에 특수 케이스부터 추가하고 싶어지면, "일반 로직이 완성되면 이 케이스가 저절로 맞을까?"부터 먼저 확인한다. 대부분의 이진 탐색/파라메트릭 서치 문제에서는 경계값이 특수 케이스가 아니라 일반 로직의 자연스러운 결과다.

## AI라면 어떻게 풀었을까

`right`의 초기값과 판정 함수 안의 시간 합산을 스트림으로 짧게 쓸 수도 있다.

```java
public static int minEatingSpeed(int[] piles, int h) {
    int left = 1;
    int right = Arrays.stream(piles).max().getAsInt();

    while (left <= right) {
        int mid = left + (right - left) / 2;
        long hours = Arrays.stream(piles).mapToLong(p -> (p + mid - 1) / mid).sum();

        if (hours <= h) {
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```

여기서는 `Math.ceil((double) piles[i] / mid)` 대신 **`(p + mid - 1) / mid`라는 정수 전용 올림 나눗셈 공식**을 썼다. `double`로 캐스팅했다가 다시 정수로 변환하는 과정에서 부동소수점 오차가 생길 여지가 아예 없고, `int`만으로 계산이 끝나서 더 안전하다. 지금 코드의 `double` 캐스팅이 이 문제(`piles[i] <= 10^9`) 규모에서는 문제없지만, 값이 더 커지는 문제에서는 이 정수 공식이 더 안전한 선택이 될 수 있다.

## 이번 백지 재작성에서 확인된 것

9/17에 진단받았던 "반복문 설계 근육 부족"은 이번엔 안 나타났다 — 탐색 범위 갱신을 `mid` 없이 짰던 실수는 스스로 알아채고 고쳤고, 코드 구조 자체(이진 탐색 골격, 판정 함수 분리)는 힌트 없이 다시 짤 수 있었다. 이번에 막힌 지점(캐스팅, 불필요한 특수 케이스)은 배열 반복문 설계와는 다른 종류의 실수라, 배열 배치로 우회했던 목적은 달성된 것으로 보인다.
