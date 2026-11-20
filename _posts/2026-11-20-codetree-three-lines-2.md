---
title: "[코드트리] 3개의 선 2 - 점마다는 OR, 점들 사이는 AND"
date: 2026-11-20 14:30:00 +0900
categories: [알고리즘 연습, 완전탐색]
tags: [java, codetree, brute-force, nested-loop, boolean-logic]
---

- 문제 링크: 코드트리 3개의 선 2 (Chapter 8 > Lesson 1)
- 파일 경로: `src/main/java/coding_test/완전탐색/CT_3개의선2.java`
- 난이도: Medium, 30XP
- 제한: 1 ≤ N ≤ 20, 0 ≤ x, y ≤ 10, 모든 점은 서로 다름 (1000ms / 64MiB)

## 문제 설명

좌표평면 위에 서로 다른 N개의 점이 있다. **x축 또는 y축에 평행한 직선 3개**로 모든 점을 지나게 할 수 있으면 1, 아니면 0을 출력한다. 세 직선은 모두 가로여도, 모두 세로여도, 섞여 있어도 된다.

```
6
1 7 / 0 0 / 1 2 / 2 0 / 1 4 / 3 4   → 1   (x=1, x=3, y=0)
```

## 시행착오

처음에는 "직선 3개를 어떻게 고르지?"가 막혔다. 좌표가 0~10이라 직선 후보가 **세로선 11개 + 가로선 11개 = 22개**뿐이다. 그래서 직선에 번호를 붙여 `i, j, k`를 전부 돌리면 된다.

- 번호 `0~10` = x좌표가 그 번호인 세로선, `11~21` = y좌표가 `번호 - 11`인 가로선.
- 22 × 22 × 22 × 점 20개 = 약 21만 번이라 충분히 빠르다.

틀린 곳은 전부 **논리 구조**였다. 같은 곳에서 네 번 고쳤다.

1. `boolean through = true;`를 점 반복문(`q`) **안**에 뒀다. 점마다 `true`로 리셋돼서 마지막 점 하나만 덮이면 1이 나왔다.
2. 가로선 비교를 `y[q] - 11 != i`로 썼다. 가로선의 y좌표는 `i - 11`이다.
3. `if / else if` 체인으로 "i에 안 맞으면 false, j에 안 맞으면 false …"를 썼다. 이건 점이 **세 직선을 전부** 지나야 통과라는 뜻(AND)이다.
4. `onLine`을 `true`로 시작해서 "못 지나면 `false`"로 바꿨다. 같은 AND 실수의 변형이다. **`false`로 시작해서 "지나면 `true`"**로 바꿔야 OR이 된다.

## 최종 코드

```java
for (int i = 0; i <= 21; i++) {
    for (int j = 0; j <= 21; j++) {
        for (int k = 0; k <= 21; k++) {
            boolean through = true;             // 모든 점이 덮였는가 (AND)

            for (int q = 0; q < n; q++) {
                boolean onLine = false;         // 이 점이 세 직선 중 하나라도 지나는가 (OR)
                if (i <= 10 && x[q] == i) onLine = true;
                else if (i > 10 && y[q] == i - 11) onLine = true;
                if (j <= 10 && x[q] == j) onLine = true;
                else if (j > 10 && y[q] == j - 11) onLine = true;
                if (k <= 10 && x[q] == k) onLine = true;
                else if (k > 10 && y[q] == k - 11) onLine = true;

                if (!onLine) through = false;
            }
            if (through) { System.out.print(1); return; }
        }
    }
}
System.out.print(0);
```

## 추적표: 예제 1

직선을 번호순으로 보면 처음 성공하는 조합은 `i=1, j=3, k=11`(x=1, x=3, y=0)이다.

| 점 | x=1 (i) | x=3 (j) | y=0 (k) | onLine |
|---|---|---|---|---|
| (1, 7) | ✅ | | | true |
| (0, 0) | | | ✅ | true |
| (1, 2) | ✅ | | | true |
| (2, 0) | | | ✅ | true |
| (1, 4) | ✅ | | | true |
| (3, 4) | | ✅ | | true |

모든 점의 `onLine`이 true라 `through`가 true로 남는다. 출력 1.

## 오늘 배운 내용

- 직선처럼 **후보가 적은 대상은 번호를 매겨 전부 돌린다.** 후보 수를 먼저 세어 보면(22개) 3중 반복이 안전하다는 걸 알 수 있다.
- 논리 구조는 **"점마다는 OR, 점들 사이는 AND"**다.
  - 한 점이 세 직선 중 **하나라도** 지나면 OK → `false`로 시작해 지나면 `true` (OR)
  - 모든 점이 OK여야 성공 → `true`로 시작해 하나라도 실패하면 `false` (AND)
- 초기값과 바꾸는 방향이 곧 연산을 정한다. `true`로 시작해 `false`로 바꾸면 AND, `false`로 시작해 `true`로 바꾸면 OR이다.
- 상태 변수(`through`)를 어느 반복문 **안**에서 초기화하느냐가 의미를 바꾼다. 조합마다 리셋은 `k` 안, 점마다 리셋은 `q` 안이다.

## 오답노트

- `through`를 `q` 안에서 초기화 → 점마다 리셋돼 마지막 점만 보게 됨.
- 가로선 비교 `y - 11 != i` → 번호에서 11을 빼는 쪽이 번호다 (`y == i - 11`).
- `else if` 체인과 `true` 시작 `onLine`으로 **OR이 필요한 곳에 AND를 만듦** (두 번). "하나라도"라는 말이 나오면 `false`로 시작한다.

## AI라면 어떻게 풀었을까

"점이 직선을 지나는가"를 메서드로 빼면 if-else 6개가 한 줄이 된다.

```java
static boolean on(int line, int px, int py) {
    return line <= 10 ? px == line : py == line - 11;
}

for (int i = 0; i <= 21; i++)
    for (int j = 0; j <= 21; j++)
        for (int k = 0; k <= 21; k++) {
            boolean all = true;
            for (int q = 0; q < n && all; q++) {
                all = on(i, x[q], y[q]) || on(j, x[q], y[q]) || on(k, x[q], y[q]);
            }
            if (all) { System.out.print(1); return; }
        }
System.out.print(0);
```

`||`가 곧 "점마다 OR"이고, `all`에 결과를 쌓는 것이 "점들 사이 AND"다. 못 지나는 점을 만나면 `q` 반복도 바로 멈춘다.
