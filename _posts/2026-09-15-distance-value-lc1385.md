---
title: "[LeetCode-1385] \"이진 탐색을 모르겠다\"의 진짜 정체는 배열 경계값 처리였다"
date: 2026-09-15 20:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search]
---

> 이진 탐색 개념과 패턴 정리는 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/find-the-distance-value-between-two-arrays/
- 파일 경로: `src/main/java/coding_test/이진탐색_투포인터/LC1385_FindTheDistanceValueBetweenTwoArrays.java`
- 난이도: Easy

## 문제 설명

정수 배열 `arr1`, `arr2`와 정수 `d`가 주어진다. `arr2`의 **어떤** 원소와도 차이(절댓값)가 `d` 이하가 되지 않는 `arr1`의 원소 개수("거리 값")를 구하라.

```
Input: arr1 = [4,5,8], arr2 = [10,9,1,8], d = 2
Output: 2
```

정렬된 배열에서 "가까운 값이 있는지"를 확인하는 문제라 이진 탐색 카테고리에 들어있다.

## 시행착오

### 1차 — 브루트포스, 근데 조건 방향이 반대

처음엔 "된다/안 된다"의 기준을 반대로 잡았다. 문제는 "arr2 어디에도 가까운 값이 **없어야**" 센다인데, "arr2에 가까운 값이 있으면" 세는 걸로 착각했다. 게다가 `arr2[0]`만 특별 취급하는 이상한 분기까지 있었다. 조건문을 말로 다시 풀어서("어떤 원소와도 <=d가 되지 않아야 카운트") 방향을 바로잡고, 불필요한 특이 케이스 분기를 지우고 나서야 브루트포스로 통과했다.

### 2차 — 이진 탐색으로 재도전, 여기서부터가 진짜였다

`arr2`를 정렬하고, `arr1[i]`가 들어갈 "삽입 위치"를 LC35(Search Insert Position) 방식으로 찾은 다음, 그 위치 근처(`arr2[left]`, `arr2[left-1]`)만 확인하면 될 거라 생각했다. 근데 여기서 크래시가 연달아 났다.

- **버그 A**: `arr1[left]`라고 잘못 써서 엉뚱한 배열을 인덱싱함 (`arr2[left]`여야 했다)
- **버그 B**: `arr2[left]` **하나만** 확인해서, `left`가 `arr2.length`(배열 끝 밖)가 되는 경우 `ArrayIndexOutOfBoundsException`

이때 크게 좌절했다. "이진 탐색 개념을 하나도 모르겠다"고 느꼈는데, 사실 막힌 지점은 개념이 아니라 **"삽입 위치의 양쪽 이웃을 안전하게 확인하는 것"**이었다. `left`와 `left-1` 둘 다 범위 안에 있는지(`left < arr2.length`, `left - 1 >= 0`) 먼저 확인하고 나서 값을 비교하는 걸로 고치고 나서야 통과했다.

```java
boolean close = false;
if (left < arr2.length && Math.abs(arr1[i] - arr2[left]) <= d) {
    close = true;
}
if (left - 1 >= 0 && Math.abs(arr1[i] - arr2[left - 1]) <= d) {
    close = true;
}
if (!close) {
    ans++;
}
```

## 최종 코드

```java
public static int distanceValue(int[] arr1, int[] arr2, int d) {
    int ans = 0;
    Arrays.sort(arr2);

    for (int i = 0; i < arr1.length; i++) {
        int left = 0;
        int right = arr2.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (arr2[mid] == arr1[i]) {
                left = mid;
                break;
            } else if (arr2[mid] < arr1[i]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }

        boolean close = false;
        if (left < arr2.length && Math.abs(arr1[i] - arr2[left]) <= d) {
            close = true;
        }
        if (left - 1 >= 0 && Math.abs(arr1[i] - arr2[left - 1]) <= d) {
            close = true;
        }
        if (!close) {
            ans++;
        }
    }

    return ans;
}
```

## 오늘 배운 내용

**"이진 탐색을 모르겠다"고 느껴지는 순간이, 실제로는 이진 탐색 개념이 아니라 "배열 인덱스 경계 처리" 문제인 경우가 많다.** 삽입 위치를 찾는 것까지는 LC35에서 이미 해봤던 거였고, 실제로 막힌 건 그 결과값(`left`)의 앞/뒤 인덱스가 배열 범위를 벗어날 수 있다는 걸 놓친 것뿐이었다. 개념과 "정확하게 구현하는 손끝 감각"은 다른 능력이라는 걸 체감했다.

## 오답노트

- **틀렸던 패턴 1**: 카운트 조건의 방향을 반대로 잡음("가까우면 센다" vs 실제로는 "안 가까우면 센다"). **다음에 떠올릴 시점**: 조건문을 짜기 전에 항상 "카운트하는 상황"을 문제 문장 그대로 한 번 따라 읽어본다.
- **틀렸던 패턴 2**: `arr1[left]`처럼 엉뚱한 배열을 인덱싱. **다음에 떠올릴 시점**: 인덱스 변수(`left`, `right`, `mid`)가 어느 배열에 속하는 인덱스인지 변수명 옆에 항상 명확히 하고 쓴다.
- **틀렸던 패턴 3**: 이웃을 하나만 확인하고 범위 체크 없이 접근. **다음에 떠올릴 시점**: 인덱스로 "앞/뒤"를 확인할 때는 값을 보기 **전에** 항상 `0 <= idx < length`부터 확인하는 습관을 들인다.

## 꿀팁

- 정렬된 배열에서 "가장 가까운 값"을 찾을 때는, 이진 탐색으로 삽입 위치(`left`)를 구한 다음 **그 위치와 바로 앞(`left-1`) 두 곳만** 확인하면 된다. 둘 다 배열 범위 안에 있는지 먼저 체크하고 나서 비교한다.
- (사고 기록) 작업 중 이 파일이 실수로 빈 템플릿으로 되돌아간 적이 있었는데, VSCode의 로컬 히스토리(파일 우클릭 → Timeline/Local History)에서 되찾았다. 저장이 꼬였다 싶으면 되돌리기 전에 로컬 히스토리부터 확인해볼 것.
