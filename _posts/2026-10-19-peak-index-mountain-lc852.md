---
title: "[LeetCode-852] 산의 꼭대기 - 오른쪽 이웃이 더 크면 아직 올라가는 중이다"
date: 2026-10-19 12:55:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search, peak]
---

> 이진 탐색 개념과 패턴은 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/peak-index-in-a-mountain-array/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0852_PeakIndexInAMountainArray.java`
- 난이도: Medium

## 문제 설명

값이 꼭대기까지 **증가했다가 감소하는** 산 모양 배열에서 꼭대기의 인덱스를 `O(log n)`으로 찾는다.

```
arr = [0,1,0]     → 1
arr = [0,2,1,0]   → 1
arr = [0,10,5,2]  → 1
```

## 시행착오

**없었다.** 같은 날 LC33에서 크게 고생한 뒤 "하나만 더" 풀어 본 문제였는데, 9개 테스트가 한 번에 통과했다. 받은 건 "가운데와 바로 오른쪽만 비교한다"는 규칙과 "올라가는 중이면 꼭대기는 어느 쪽인가?"라는 질문 두 개뿐이었다.

스스로 설명한 전략은 이랬다.

> left가 점점 올라가니까, 마지막에 `arr[mid]`가 `arr[mid + 1]`보다 크면 `right`를 바로 줄여서 `left`랑 같게 하려는 전략이다. 그래서 `left`가 답일 것이다.

정확한 직관이다. **올라가는 중이면 꼭대기는 확실히 오른쪽**이라 `left = mid + 1`로 버리고, **내려가는 중이면 `mid` 자신이 꼭대기일 수도 있어서** `right = mid`로 후보를 남긴다. 그러면 두 포인터가 꼭대기 한 점으로 모인다.

## 최종 코드

```java
public int peakIndexInMountainArray(int[] arr) {
    int left = 0;
    int right = arr.length - 1;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] < arr[mid + 1]) {
            left = mid + 1;          // 올라가는 중 → 꼭대기는 mid 오른쪽
        } else {
            right = mid;             // 내려가는 중(또는 꼭대기) → mid도 후보
        }
    }
    return left;                     // left == right: 꼭대기
}
```

9개 테스트가 통과했고, 랜덤 50,000건에서도 정답과 일치했다.

## 추적표: `[1, 3, 5, 4, 2]`

| 단계 | left | right | mid | arr[mid] vs arr[mid+1] | 이동 |
|---|---|---|---|---|---|
| 1 | 0 | 4 | 2 | 5 > 4 (내려감) | `right = 2` |
| 2 | 0 | 2 | 1 | 3 < 5 (올라감) | `left = 2` |
| 끝 | 2 | 2 | | | **2 반환** |

## 오늘 배운 내용

**이진 탐색의 "버릴 근거"는 정렬이 아니라 "기울기"여도 된다.** `arr[mid] < arr[mid + 1]`이면 오르막이라 꼭대기는 오른쪽에 있다. LC153(가운데와 오른쪽 끝 비교), LC33(정렬된 쪽 확인)과 같은 계열로, 이번 주 이진 탐색 응용은 모두 **"한쪽을 확실히 버릴 수 있는 비교 하나"**를 찾는 문제였다.

**`left < right` + `right = mid` 틀에서 `mid + 1`은 항상 안전하다.** `mid`는 `right`보다 항상 작아서 `mid + 1`이 배열 밖으로 나가지 않는다.

## 오답노트

- **틀린 곳은 없었다.**
- **좋았던 점**: LC153에서 쓴 반열린 틀(`left < right`, `left = mid + 1` / `right = mid`, `return left`)을 이번엔 설명 없이 그대로 꺼내 썼다. 같은 틀이 세 문제(LC35 대안, LC153, LC852)에 반복되면서 손에 붙고 있다.
- **확인할 점**: "감으로" 맞힌 경우에도 왜 맞는지를 한 줄로 말할 수 있어야 다음에 재현된다. 이번엔 직접 설명까지 했다.

## AI라면 어떻게 풀었을까

한 점으로 모일 때까지 좁히는 대신 **`mid`가 꼭대기인지 그 자리에서 바로 확인**한다. 꼭대기는 양옆 이웃보다 큰 칸이다. 산 모양이 보장되어 양 끝은 꼭대기가 될 수 없으므로 범위를 `1 ~ n-2`로 두면 `mid - 1`, `mid + 1`이 항상 안전하다.

```java
public int peakIndexAlt(int[] arr) {
    int left = 1;
    int right = arr.length - 2;                  // 양 끝은 꼭대기가 아님
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid - 1] < arr[mid] && arr[mid] > arr[mid + 1]) {
            return mid;                          // 양옆보다 크면 꼭대기
        } else if (arr[mid - 1] < arr[mid]) {
            left = mid + 1;                      // 올라가는 중 → 꼭대기는 오른쪽
        } else {
            right = mid - 1;                     // 내려가는 중 → 꼭대기는 왼쪽
        }
    }
    return -1;                                   // 산 모양이면 오지 않음
}
```

트레이드오프: 꼭대기를 만나면 바로 끝나서 직관적이다. 대신 비교가 두 번(왼쪽 이웃, 오른쪽 이웃) 필요하고, 양 끝을 범위에서 빼는 이유를 알아야 한다. 랜덤 50,000건에서 본인 풀이와 출력이 일치했다.
