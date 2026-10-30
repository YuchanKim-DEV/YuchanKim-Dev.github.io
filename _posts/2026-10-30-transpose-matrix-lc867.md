---
title: "[LeetCode-867] Transpose Matrix - 어느 배열 기준으로 도는가"
date: 2026-10-30 10:13:00 +0900
categories: [알고리즘 연습, 구현]
tags: [java, leetcode, matrix, implementation]
---

- 문제 링크: https://leetcode.com/problems/transpose-matrix/
- 파일 경로: `src/main/java/coding_test/구현/LC0867_TransposeMatrix.java`
- 난이도: Easy

## 문제 설명

2차원 배열의 **행과 열을 바꾼** 배열(전치 행렬)을 반환한다. 정사각형이 아닐 수도 있다.

```
[[1,2,3],        [[1,4],
 [4,5,6]]   →     [2,5],
                  [3,6]]
```

원래 `[행][열]`에 있던 값은 결과에서 `[열][행]`으로 간다. 그래서 2×3 배열은 3×2가 된다.

정렬 유형을 마치고 금요일 구현 슬롯으로 골랐다. 다음 주부터 시작하는 그래프·격자 BFS에 앞서 2차원 인덱스 감각을 잡으려는 목적이다.

## 시행착오

### 1차: 크기는 맞음, 읽는 쪽 인덱스를 뒤집음

```java
int[][] ans = new int[matrix[0].length][matrix.length];
for (int i = 0; i < matrix.length; i++) {
    for (int j = 0; j < matrix[i].length; j++) {
        ans[i][j] = matrix[j][i];
    }
}
```

가장 놓치기 쉬운 **결과 배열 크기**(행·열 길이 바꾸기)는 처음부터 맞았다.

틀린 곳은 대입 한 줄이었다. 루프는 `matrix` 기준으로 돈다(`i`는 행 0 ~ m-1, `j`는 열 0 ~ n-1). 그런데 정작 `matrix`를 `[j][i]`로 읽었다. 정사각형에서는 우연히 맞지만, 2×3에서는 `matrix[2][0]`을 읽으려다 범위를 벗어난다.

### 2차: 통과

`matrix`는 루프 기준 그대로 `[i][j]`로 읽고, 쓰는 쪽 `ans`를 `[j][i]`로 뒤집어 통과했다.

## 최종 코드

```java
public int[][] transpose(int[][] matrix) {
    int[][] ans = new int[matrix[0].length][matrix.length];   // 행·열 길이를 바꿔서

    for (int i = 0; i < matrix.length; i++) {                 // matrix 기준으로 돈다
        for (int j = 0; j < matrix[i].length; j++) {
            ans[j][i] = matrix[i][j];                         // 읽기는 그대로, 쓰기는 뒤집어서
        }
    }
    return ans;
}
```

## 추적표: `[[1,2,3],[4,5,6]]`

| i | j | matrix[i][j] | 쓰는 칸 |
|---|---|---|---|
| 0 | 0 | 1 | ans[0][0] |
| 0 | 1 | 2 | ans[1][0] |
| 0 | 2 | 3 | ans[2][0] |
| 1 | 0 | 4 | ans[0][1] |
| 1 | 1 | 5 | ans[1][1] |
| 1 | 2 | 6 | ans[2][1] |

→ `[[1,4],[2,5],[3,6]]`

## 오늘 배운 내용

**2차원 배열을 두 개 다룰 때는 "어느 배열 기준으로 도는가"부터 정한다.** 루프 범위를 정한 배열은 `[i][j]` 그대로 쓰고, 다른 배열만 인덱스를 바꾼다. 정사각형 테스트만으로는 이 실수가 드러나지 않으니, 직사각형 예시를 꼭 하나 돌려 본다.

## 오답노트

- **틀렸던 지점**: `ans[i][j] = matrix[j][i]`. **왜**: 행·열을 바꾼다는 생각만 하고, 루프가 어느 배열 범위로 도는지와 맞춰 보지 않았다. **다음에**: `for` 조건에 쓴 배열(`matrix.length`, `matrix[i].length`)을 보고, 그 배열은 `[i][j]`로 쓴다.

## AI라면 어떻게 풀었을까

본인 풀이가 정석이다. 반대로 **결과 배열 기준**으로 돌아도 된다. 이때는 쓰는 쪽이 `[r][c]`, 읽는 쪽이 `[c][r]`이다.

```java
public int[][] transposeAlt(int[][] matrix) {
    int rows = matrix[0].length, cols = matrix.length;   // 결과 크기: 행·열이 바뀜
    int[][] ans = new int[rows][cols];
    for (int r = 0; r < rows; r++) {                     // 결과 배열 기준
        for (int c = 0; c < cols; c++) {
            ans[r][c] = matrix[c][r];
        }
    }
    return ans;
}
```

트레이드오프는 없다. 어느 쪽이든 "기준 배열은 `[i][j]` 그대로"라는 규칙만 지키면 된다. 랜덤 크기 20,000건에서 두 풀이의 결과가 같았다.
