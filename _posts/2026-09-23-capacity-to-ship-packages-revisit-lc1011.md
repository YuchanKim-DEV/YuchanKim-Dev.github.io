---
title: "[LeetCode-1011] 짐 하나가 용량보다 크면, 그 용량은 무조건 탈락시켜야 한다"
date: 2026-09-23 12:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search]
---

> 이진 탐색 개념과 패턴 정리는 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다. 이 문제의 첫 풀이 글은 [9/14 글]({% post_url 2026-09-14-capacity-to-ship-packages-lc1011 %})에 있다 — 이번은 백지 재작성 검증 글이다. [LC875 재작성 글]({% post_url 2026-09-22-koko-eating-bananas-revisit-lc875 %})에 이어지는 이진 탐색 복귀의 두 번째 문제다.

- 문제 링크: https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC1011_CapacityToShipPackagesWithinDDays.java`
- 난이도: Medium

## 문제 설명

컨베이어 벨트 위에 짐이 순서대로 놓여 있고, days일 안에 전부 배로 실어 날라야 한다. 매일 짐을 순서대로 배에 싣되, 그날 실을 수 있는 무게 합은 배의 최대 적재 용량을 넘을 수 없다. days일 안에 모든 짐을 다 실을 수 있는 **최소 적재 용량**을 구한다.

```
Input: weights = [1,2,3,1,1], days = 4
Output: 3
```

## 시행착오

875보다 훨씬 오래 걸렸다. 여러 라운드에 걸쳐 하나씩 고쳤다.

**1. 탐색 상한(`right`)을 잘못 잡음.** 처음엔 짐 하나의 최댓값(제약 조건에 있는 `500`)으로 잡았는데, `days=1`이면 짐 전체를 하루에 다 실어야 한다는 걸 생각하면 상한은 **모든 짐 무게의 합**(`sum(weights)`)이어야 한다는 걸 깨닫고 고쳤다.

**2. 용량 초과를 확인하는 시점이 늦음.** `total += weights[i]`로 일단 더한 다음 `total >= mid`를 확인했는데, 이러면 이미 용량을 넘겨서 더한 뒤에야 알아챈다. `total + weights[i] > mid`(더하기 전에 미리 확인)로 바꿔서, 넘칠 짐은 애초에 오늘 몫에 넣지 않도록 고쳤다.

**3. 루프 종료 후 남은 마지막 날 처리가 계속 빠짐.** 배열을 다 순회하고 나서도 `total`에 아직 하루로 안 세어진 짐이 남아있을 수 있는데, 이 처리(`if (total > 0) numDays++`)를 몇 번이나 썼다 지웠다 했다.

**4. 이진 탐색 갱신 조건의 경계가 반대로 나뉨.** `numDays < days`(용량이 넉넉해서 필요한 날짜보다 여유 있음)인데 `left = mid + 1`(용량을 더 키우는 방향)로 잘못 갔다. 판단 기준을 "필요한 날짜가 허용 날짜를 **넘는가**"로 다시 정리해서 `numDays > days`(불가능, 용량 키움) / `numDays <= days`(가능, 용량 줄여봄)로 고쳤다.

**5. 가장 크게 발목 잡힌 지점 — 9/14에 이미 배웠던 교훈이 재발했다.** 오답노트에 "짐 하나(`weights[i]`)가 그 자체로 `mid`보다 크면 그 용량은 절대 불가능"이라고 적어뒀었는데, 이번에 그 검사를 빼먹었다. 짐 하나가 용량보다 크면, 새 날을 시작해도 그 짐 자체가 하루 용량에 못 들어가는데 시뮬레이션은 그냥 "새 날 시작"으로 처리해버려서 불가능한 용량이 "가능하다"고 잘못 판정됐다. 루프 맨 앞에 `if (weights[i] > mid) { numDays = days + 1; break; }`를 추가해서, 짐 하나라도 용량을 넘으면 그 즉시 이 용량을 확실히 탈락시키도록 고쳤다.

## 최종 코드

```java
public static int shipWithinDays(int[] weights, int days) {
    int left = 1;
    int right = 0;
    for (int w : weights) {
        right += w;
    }

    while (left <= right) {
        int mid = left + (right - left) / 2;

        int total = 0;
        int numDays = 0;
        for (int i = 0; i < weights.length; i++) {
            if (weights[i] > mid) {
                numDays = days + 1;
                break;
            }
            if (total + weights[i] > mid) {
                numDays++;
                total = weights[i];
            } else {
                total += weights[i];
            }
        }

        if (total > 0) {
            numDays++;
        }

        if (numDays <= days) {
            right = mid - 1;
        } else if (numDays > days) {
            left = mid + 1;
        }
    }

    return left;
}
```

## 오늘 배운 내용

**한 번 오답노트에 적어둔 교훈이라도, 백지에서 다시 짤 때 저절로 튀어나오는 건 아니다.** "짐 하나가 용량보다 크면 무조건 불가능"이라는 규칙은 9/14에 이미 명시적으로 적어뒀던 내용인데, 이번엔 그 검사 자체를 빼먹고 시작했다. 오답노트는 "다시 안 틀리게 해주는 보험"이 아니라 "이 지점에서 또 틀릴 수 있다는 걸 미리 아는 지도"에 가깝다 — 백지 재작성을 할 때는 지난 오답노트를 체크리스트처럼 옆에 두고 하나씩 확인하는 게 나을 수도 있겠다.

## 오답노트

- **틀렸던 패턴 1**: `right`를 짐 하나의 최댓값으로 잡음. **왜**: `days=1`인 극단적 케이스를 생각 안 했다.
- **틀렸던 패턴 2**: `total >= mid`(더한 뒤 확인). **왜**: "더하기 전에 미리 확인"과 "더한 뒤 사후 확인"이 다른 결과를 낸다는 걸 놓쳤다.
- **틀렸던 패턴 3**: 루프 종료 후 남은 `total` 처리 누락(반복적으로). **왜**: 시뮬레이션 코드를 여러 번 고치는 과정에서 이 처리가 계속 같이 삭제됐다.
- **틀렸던 패턴 4**: 이진 탐색 갱신 조건의 경계(`<` vs `<=`)가 반대로 나뉨. **왜**: "가능/불가능"을 나누는 정확한 경계값을 말로 먼저 확인하지 않고 코드부터 짰다.
- **틀렸던 패턴 5 (핵심)**: 짐 하나가 용량보다 큰 경우를 걸러내는 검사 누락. **왜**: 지난 풀이의 오답노트에 이미 적어뒀던 교훈인데도, 백지 상태에서 새로 짜다 보니 자동으로 떠오르지 않았다.
- **다음에 떠올릴 시점**: 파라메트릭 서치 문제에서 "원소 하나가 그 자체로 후보값보다 큰 경우"는 거의 항상 별도로 걸러줘야 하는 특수 상황이다. 판정 함수를 짤 때 이 경우부터 먼저 체크하는 습관을 들인다.

## AI라면 어떻게 풀었을까

`right` 초기값과 판정 함수를 스트림으로 짧게 쓸 수도 있다.

```java
public static int shipWithinDays(int[] weights, int days) {
    int left = Arrays.stream(weights).max().getAsInt();
    int right = Arrays.stream(weights).sum();

    while (left <= right) {
        int mid = left + (right - left) / 2;
        int numDays = 1;
        int total = 0;
        for (int w : weights) {
            if (total + w > mid) {
                numDays++;
                total = 0;
            }
            total += w;
        }

        if (numDays <= days) {
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```

여기서는 `left`의 시작값 자체를 `Arrays.stream(weights).max()`(가장 무거운 짐)로 잡았다 — 어차피 용량은 가장 무거운 짐보다는 커야 하니, 이 값 미만은 애초에 탐색할 필요가 없다. 이렇게 하면 지금 코드에서 썼던 "짐 하나가 mid보다 크면 탈락" 검사 자체가 필요 없어진다 — 탐색 범위 자체가 이미 그 조건을 만족하는 구간으로 좁혀져 있기 때문이다. 문제를 푸는 방식이 아니라 **탐색 범위를 잡는 방식**으로 같은 버그를 원천적으로 막은 셈이다.
