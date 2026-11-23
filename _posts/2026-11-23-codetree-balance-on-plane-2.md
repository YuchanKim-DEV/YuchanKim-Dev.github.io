---
title: "[코드트리] 좌표평면 위의 균형 2 - max는 조합마다, min은 전체에서"
date: 2026-11-23 10:30:00 +0900
categories: [알고리즘 연습, 완전탐색]
tags: [java, codetree, brute-force, nested-loop, min-max]
---

- 문제 링크: 코드트리 좌표평면 위의 균형 2 (Chapter 8 > Lesson 1)
- 파일 경로: `src/main/java/coding_test/완전탐색/CT_좌표평면위의균형2.java`
- 난이도: Medium, 50XP
- 제한: 1 ≤ N ≤ 100, 1 ≤ x, y ≤ 100, x와 y는 모두 홀수

## 문제 설명

좌표평면 위에 N개의 점이 있다. **x축에 평행한 직선 1개와 y축에 평행한 직선 1개**를 그어 4구역으로 나눈다. 직선은 항상 **짝수 좌표**에 긋는다. 4구역 중 점이 가장 많은 구역의 점 수를 M이라 할 때, **M의 최솟값**을 구한다.

```
7
7 3 / 5 5 / 9 7 / 3 1 / 7 7 / 5 3 / 9 1   → 2   (x=6, y=4에 그으면 1, 2, 2, 2)
```

## 시행착오

"이것도 완전탐색이냐"는 질문에서 시작했다. 직선을 그을 수 있는 위치가 짝수 좌표뿐이라 후보가 적다. **x선 후보 × y선 후보**를 전부 돌리면 된다.

- 점은 모두 홀수 좌표라 짝수 직선 위에 놓일 일이 없다. 그래서 "작다/크다" 둘로만 나눠도 4구역이 빠짐없이 갈린다.
- 처음 `i`, `j` 범위를 `0 ~ 10`으로 썼다. 좌표가 100까지라 `0 ~ 100`이어야 한다.
- **`max` 변수를 반복문 밖에서 한 번만 `0`으로 만들었다.** 조합이 바뀌어도 이전 조합의 최댓값이 이어졌다.
- **`min` 갱신을 `j` 반복문 밖(`i` 하나가 끝날 때)에 뒀다.** 조합마다 M이 하나씩 나오므로 `j` 안에서 갱신해야 한다.

## 최종 코드

```java
int min = Integer.MAX_VALUE;
int max = 0;

for (int i = 0; i <= 100; i += 2) {
    for (int j = 0; j <= 100; j += 2) {
        int first = 0, second = 0, third = 0, fourth = 0;
        max = 0;                                   // 조합마다 초기화

        for (int k = 0; k < n; k++) {
            if (x[k] < i && y[k] < j) first++;
            else if (x[k] > i && y[k] < j) second++;
            else if (x[k] < i && y[k] > j) third++;
            else fourth++;
        }
        max = Math.max(max, first);
        max = Math.max(max, second);
        max = Math.max(max, third);
        max = Math.max(max, fourth);
        min = Math.min(max, min);                  // 조합마다 갱신
    }
}
System.out.print(min);
```

## 추적표: 예제 1에서 `(i, j) = (6, 4)`

| 점 | x < 6? | y < 4? | 구역 |
|---|---|---|---|
| (7, 3) | 아니오 | 예 | second |
| (5, 5) | 예 | 아니오 | third |
| (9, 7) | 아니오 | 아니오 | fourth |
| (3, 1) | 예 | 예 | first |
| (7, 7) | 아니오 | 아니오 | fourth |
| (5, 3) | 예 | 예 | first |
| (9, 1) | 아니오 | 예 | second |

`first=2, second=2, third=1, fourth=2` → 이 조합의 M은 2. 모든 조합 중 최솟값도 2.

## 오늘 배운 내용

- 후보가 한정된 직선 문제도 **후보 × 후보 이중 반복**이다. 후보 개수(51 × 51)를 세어 보면 안전하다는 걸 안다.
- 최댓값/최솟값을 함께 쓰는 문제는 **변수의 수명**을 구분해야 한다.
  - `max`는 "이번 조합의 값" → 조합마다 `0`으로 초기화
  - `min`은 "지금까지의 최소" → 맨 처음 한 번만 큰 값으로 초기화, 조합마다 갱신
- 반복문 범위는 예제 크기가 아니라 **제한 조건**으로 정한다.

## 오답노트

- `max`를 조합마다 초기화하지 않음 → 이전 조합의 값이 이어짐.
- `min`을 `j` 반복문 밖에서 갱신 → `i` 하나당 마지막 `j`의 값만 반영됨.
- 반복문 범위를 `10`으로 씀 → 제한 조건이 `100`.

## AI라면 어떻게 풀었을까

구역 4개를 변수 4개 대신 배열로 두고, 구역 번호를 식으로 구한다.

```java
int best = Integer.MAX_VALUE;
for (int a = 0; a <= 100; a += 2) {
    for (int b = 0; b <= 100; b += 2) {
        int[] cnt = new int[4];
        for (int q = 0; q < n; q++) {
            int zone = (x[q] > a ? 1 : 0) * 2 + (y[q] > b ? 1 : 0);
            cnt[zone]++;
        }
        int m = Math.max(Math.max(cnt[0], cnt[1]), Math.max(cnt[2], cnt[3]));
        best = Math.min(best, m);
    }
}
```

`m`은 조합 안에서 새로 만들고, `best`는 바깥에서 한 번만 초기화한다. 변수 수명이 선언 위치에 그대로 드러난다.
