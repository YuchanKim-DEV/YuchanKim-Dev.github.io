---
title: "2차원 행렬도 결국 1차원 배열이다 - Search a 2D Matrix로 인덱스 변환 감 잡기"
date: 2026-09-09 21:00:00 +0900
categories: [코딩테스트, 이진탐색]
tags: [java, leetcode, binary-search]
---

- 문제 링크: https://leetcode.com/problems/search-a-2d-matrix/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0074_SearchA2DMatrix.java`
- 난이도: Medium

## 문제 설명

`m x n` 정수 행렬이 주어진다. 각 행은 오름차순 정렬돼 있고, **각 행의 첫 값이 이전 행의 마지막 값보다 크다.** `target`이 행렬에 있는지 O(log(m*n))에 판별해야 한다.

```
matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]
target = 3   -> true
target = 13  -> false
```

## 시행착오

### mid 계산 부호 오타

```java
int mid = left - (right - left) / 2;   // 오타: - 여야 할 게 아니라 +
```

`left=0, right=11`로 계산하면 `mid = 0 - 5 = -5`, 음수 인덱스가 나오는 걸 직접 확인하고 `+`로 수정했다.

## 깨달은 것 — 2차원을 1차원처럼 다루기

문제의 두 조건("각 행 정렬" + "다음 행 첫 값이 이전 행 마지막 값보다 큼")을 합치면, 행렬 전체를 한 줄로 쭉 이어붙인 게 정렬된 1차원 배열과 완전히 같다.

```
[[1,3,5,7],[10,11,16,20],[23,30,34,60]]
→ [1,3,5,7, 10,11,16,20, 23,30,34,60]
```

그래서 `left`, `right`를 "행 번호"가 아니라 **전체 원소 개수 기준 1차원 인덱스**로 잡고, 704 템플릿을 그대로 쓴 다음 `mid`(1차원 인덱스)를 실제 행/열로 변환하면 된다:

- `row = mid / n` (한 행의 길이로 나눈 몫)
- `col = mid % n` (나머지)

## 최종 코드

```java
public static boolean searchMatrix(int[][] matrix, int target) {
    int left = 0;
    int right = (matrix.length * matrix[0].length) - 1;
    int n = matrix[0].length;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int row = mid / n;
        int col = mid % n;

        if (matrix[row][col] == target) {
            return true;
        } else if (matrix[row][col] > target) {
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }

    return false;
}
```

## 변수 추적표 (target = 3)

`matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `n = 4`

| 반복 | left | right | mid | row=mid/n | col=mid%n | matrix[row][col] | 비교 | 갱신 |
|---|---|---|---|---|---|---|---|---|
| 1 | 0 | 11 | 0+(11-0)/2=**5** | 5/4=**1** | 5%4=**1** | matrix[1][1]=**11** | 11>3 | right=4 |
| 2 | 0 | 4 | 0+(4-0)/2=**2** | 2/4=**0** | 2%4=**2** | matrix[0][2]=**5** | 5>3 | right=1 |
| 3 | 0 | 1 | 0+(1-0)/2=**0** | 0/4=**0** | 0%4=**0** | matrix[0][0]=**1** | 1<3 | left=1 |
| 4 | 1 | 1 | 1+(1-1)/2=**1** | 1/4=**0** | 1%4=**1** | matrix[0][1]=**3** | 3==3 | `return true` |

## 한 줄 오답노트

> 2차원 배열이 "행마다 정렬 + 다음 행이 이전 행보다 큼" 조건이면, 1차원 배열로 편 것과 동일하다. `row = idx/n`, `col = idx%n`로 변환하면 704 템플릿을 그대로 재사용할 수 있다.
