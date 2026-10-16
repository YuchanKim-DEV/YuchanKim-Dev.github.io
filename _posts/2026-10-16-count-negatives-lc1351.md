---
title: "[LeetCode-1351] 행마다 '첫 음수 위치'를 찾고 길이에서 뺀다"
date: 2026-10-16 09:30:00 +0900
categories: [알고리즘 연습, 이진탐색]
tags: [java, leetcode, binary-search, lower-bound, matrix]
---

> 이진 탐색 개념과 패턴은 [카테고리 총정리 글]({% post_url 2026-09-09-binary-search-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/count-negative-numbers-in-a-sorted-matrix/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC1351_CountNegativeNumbersInASortedMatrix.java`
- 난이도: Easy

## 문제 설명

각 행과 각 열이 **내림차순**(같은 값 허용)으로 정렬된 `m x n` 행렬에서 **음수의 개수**를 센다.

```
grid = [[4,3,2,-1],[3,2,1,-1],[1,1,-1,-2],[-1,-1,-2,-3]]  → 8
grid = [[3,2],[1,0]]                                       → 0
```

## 시행착오

스택·큐에 오래 있다가 이진 탐색으로 돌아온 첫 문제였다. "이진 탐색 감각을 다 잃어버린 것 같다"고 해서 입국심사(Lv3) 대신 Easy로 바꿨고, 그래도 **오타성 실수**가 여러 번 나왔다. 늦은 시간(21시 30분~22시)이기도 했다.

### 0차: 범위가 뭔지 감이 안 잡힘

"범위를 찾는 게 맞나?"를 먼저 물었다. 답은 맞는데 **범위가 아주 작다**는 것이었다. 행 하나의 칸 번호 `0 ~ 열 끝`이다. `[4,3,2,-1]`이면 첫 음수가 3번 칸이고, 음수 개수는 `행의 길이 4 - 3 = 1`이다.

### 1차: `mid` 계산식

```java
int mid = left - (left + right) / 2;     // → mid가 음수 → ArrayIndexOutOfBoundsException: Index -1
```

`+`가 아니라 `-`였고 괄호 안도 `(right - left)`가 아니라 `(left + right)`였다. 익숙한 식 `left + (right - left) / 2`를 한 번에 못 썼다.

### 2차: 수도코드를 받아서 옮기는 중의 네 가지 차이

수도코드를 받아 한 줄씩 옮겼지만 네 곳이 달랐고, **한 번에 하나씩** 고쳤다.

| 위치 | 수도코드 | 틀린 코드 |
|---|---|---|
| `mid` | `left + (right - left) / 2` | `left + (left - right) / 2` (뺄셈 순서 반대) |
| `right` 시작 | 행의 길이 | `grid.length - 1` (행 개수, 게다가 `-1`) |
| 양수일 때 | `left = mid + 1` | `left = mid` (`+1` 빠짐) |
| 개수 | `행의 길이 - left` | `left - mid` → `grid.length - mid` |

### 3차: 마지막 한 줄

```java
counter += grid.length - mid;        // grid.length는 "행 개수", mid는 "마지막에 본 칸"
counter += grid[i].length - left;    // 올바른 식
```

반복이 끝나면 **`left`가 첫 음수의 위치**다. 음수 개수는 그 행의 칸 수 `grid[i].length`에서 `left`를 뺀 값이다. `mid`는 마지막에 본 칸일 뿐이라 의미가 없다. 그리고 `grid.length`는 행 개수라서 열 개수(`grid[i].length`)와 다르다. 크레인 인형뽑기에서 배운 `[행][열]` 구분이 다시 나왔다.

## 최종 코드

```java
public int countNegatives(int[][] grid) {
    int counter = 0;
    for (int i = 0; i < grid.length; i++) {
        int left = 0;
        int right = grid[i].length;              // 음수가 없으면 "첫 음수 위치"가 길이가 되도록
        int mid = 0;
        while (left < right) {
            mid = left + (right - left) / 2;
            if (grid[i][mid] >= 0) {
                left = mid + 1;                  // 이 칸은 음수 아님 → 왼쪽 버림
            } else {
                right = mid;                     // 이 칸도 후보 → 오른쪽 버림
            }
        }
        counter += grid[i].length - left;        // left = 첫 음수 위치
    }
    return counter;
}
```

9개 테스트가 모두 통과했고, 직사각형 포함 랜덤 20,000건에서도 일치했다.

## 추적표: 행 `[4, 3, 2, -1]`

| 단계 | left | right | mid | 칸 값 | 이동 |
|---|---|---|---|---|---|
| 1 | 0 | 4 | 2 | 2 (≥0) | `left = 3` |
| 2 | 3 | 4 | 3 | -1 (<0) | `right = 3` |
| 끝 | 3 | 3 | | | 첫 음수 위치 = 3, 개수 = `4 - 3 = 1` |

## 오늘 배운 내용

**"첫 번째 ○○의 위치"를 찾는 이진 탐색(하한 경계)에서는 `right`를 길이로 시작하고, 끝난 뒤 `left`가 답이다.** 조건에 맞는 칸이 하나도 없으면 `left`가 길이가 되어 "그 뒤로 몇 개인가 = 길이 - left = 0"이 자연스럽게 나온다. 이 모양이 LC35(삽입 위치), LC2529, LC1351 모두에 같다.

## 오답노트

- **틀렸던 지점 1**: `mid = left - (left + right) / 2`. **왜**: 외운 식의 기호를 한 군데씩 틀렸다(밤 늦은 시간). **다음에**: `mid`는 항상 `left + (right - left) / 2`로 쓴다. `right - left`가 양수이므로 `mid`가 음수가 될 수 없다.
- **틀렸던 지점 2**: `right`를 행 개수로 둠. **왜**: "행의 길이"를 `grid.length`로 읽었다. **다음에**: 한 행 안에서 탐색하면 범위는 **그 행의 칸 수** `grid[i].length`다.
- **틀렸던 지점 3**: `left = mid`(`+1` 없음). **왜**: "이 칸은 아니니 버린다"를 코드로 옮기면서 `+1`을 놓쳤다. **다음에**: 버리는 쪽은 항상 `mid ± 1`이고, 후보로 남기는 쪽만 `mid`다.
- **틀렸던 지점 4**: 결과를 `mid`로 계산. **왜**: 마지막에 본 칸이 답이라고 착각했다. **다음에**: 하한 경계 이진 탐색은 반복이 끝난 뒤 `left`(= `right`)가 답이다.
- **참고**: 수도코드를 옮길 때는 **한 줄씩 대조**하면 이런 차이를 바로 찾는다.

## AI라면 어떻게 풀었을까

행마다 매번 `[0, 열 길이)` 전체에서 이진 탐색하는 대신, **"열도 내림차순"이라는 조건을 활용해 탐색 범위를 줄여 나간다.** 윗 행의 첫 음수가 `p`번 칸이면, 아래 행의 `p`번 칸은 윗 행보다 작거나 같으므로 이미 음수다. 그래서 아래 행의 첫 음수 위치는 `p`보다 오른쪽일 수 없다. `right`를 이전 행의 결과로 계속 줄인다.

```java
public int countNegativesAlt(int[][] grid) {
    int n = grid[0].length;
    int right = n;                          // 첫 음수 위치의 상한 (행을 내려갈수록 줄어든다)
    int count = 0;
    for (int[] row : grid) {
        int left = 0, hi = right;
        while (left < hi) {
            int mid = left + (hi - left) / 2;
            if (row[mid] >= 0) {
                left = mid + 1;
            } else {
                hi = mid;
            }
        }
        right = left;                       // 다음 행은 이 위치를 넘지 못한다
        count += n - left;
    }
    return count;
}
```

트레이드오프: 행마다 탐색 범위가 점점 줄어서 더 빠르다. 대신 "열이 내림차순이라 아래 행의 첫 음수 위치가 오른쪽으로 갈 수 없다"는 성질을 알아야 읽힌다. 후속 질문인 O(n + m) 풀이는 오른쪽 위 모서리에서 출발하는 계단식 탐색이지만 이진 탐색이 아니라서, 이 카테고리 연습 취지에 맞춰 이진 탐색을 유지한 버전을 골랐다. 랜덤 20,000건에서 본인 풀이와 출력이 일치했다.
