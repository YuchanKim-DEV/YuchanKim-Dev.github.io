---
title: "[코드트리] 마라톤 중간에 택시타기 2 - 건너뛴 점은 prev를 갱신하지 않고 continue"
date: 2026-11-13 09:20:00 +0900
categories: [알고리즘 연습, 완전탐색]
tags: [java, codetree, brute-force, manhattan-distance]
---

- 문제 링크: 코드트리 Trail 2. Novice Mid > Chapter 6 > Lesson 1 (자리 수 단위로 완전탐색)
- 파일 경로: `src/main/java/coding_test/완전탐색/CT_마라톤중간에택시타기2.java`
- 난이도: 50XP (정답률 76%)

## 문제 설명

체크포인트 N개를 1번부터 N번까지 순서대로 방문한다. 1번과 N번을 뺀 **중간 체크포인트 하나**를 건너뛸 때, 이동 거리의 최솟값을 구한다. 거리는 택시 거리 `|x1-x2| + |y1-y2|`이고, 좌표가 겹쳐도 건너뛰는 것은 그 번호 하나뿐이다.

```
4
0 0
8 3
11 -1
10 0   → 14   (2번을 건너뛰면 (0,0)→(11,-1)→(10,0) = 12 + 2)
```

## 시행착오

N이 작아서 "건너뛸 점을 하나씩 정해 놓고 처음부터 다시 더하기"로 바로 정했다. 걸린 부분은 하나였다.

### 건너뛴 점 다음의 거리는 누구와 재야 하나

건너뛴 점이 있으면 `j - 1`번과 재면 안 된다. 건너뛴 점 앞의 점과 재야 한다. 그래서 `j - 1` 대신 **직전에 실제로 방문한 점 `prev`**를 따로 두고, 건너뛸 때는 `prev`를 갱신하지 않고 `continue`했다.

```java
if (j == skip) continue;          // prev는 그대로 → 다음 점이 앞의 점과 이어짐
total += |x[j]-x[prev]| + |y[j]-y[prev]|;
prev = j;
```

건너뛰기 후보는 `1 ~ n-2`(0-indexed)로만 돌려서 처음과 끝은 건드리지 않는다.

## 최종 코드

```java
long best = Long.MAX_VALUE;
for (int skip = 1; skip < n - 1; skip++) {
    long total = 0;
    int prev = 0;
    for (int j = 1; j < n; j++) {
        if (j == skip) {
            continue;
        }

        total += Math.abs(x[j] - x[prev]) + Math.abs(y[j] - y[prev]);
        prev = j;
    }

    best = Math.min(best, total);
}
```

## 추적표: 예제 (skip = 1)

| j | prev | 동작 | total |
|---|---|---|---|
| 1 | 0 | skip → continue | 0 |
| 2 | 0 | (0,0)→(11,-1) = 12 | 12 |
| 3 | 2 | (11,-1)→(10,0) = 2 | 14 |

skip = 2일 때는 (0,0)→(8,3)→(10,0) = 11 + 5 = 16이라 최솟값은 14.

## 오늘 배운 내용

- 완전탐색의 틀: **"무엇을 하나 정한다" 바깥 루프 + "그 상태로 전부 계산한다" 안쪽 루프**.
- "빼고 이어 붙이기"는 `prev`(직전에 실제로 쓴 값)를 따로 들고 가면 된다. 건너뛸 때 `prev`를 안 바꾸는 것이 핵심.
- 처음과 끝은 못 건너뛰므로 후보 범위는 `1 ~ n-2`.

## 오답노트

- 이번에는 틀린 곳이 없었다. 다만 `prev`가 없이 `j - 1`로 쟀다면 예제에서 바로 틀렸을 것이다. 인덱스가 "직전 번호"인지 "직전에 실제로 쓴 번호"인지 구분해서 읽는다.

## AI라면 어떻게 풀었을까

매번 처음부터 더하지 않고, 전체 거리에서 **바뀌는 조각만 고친다**. `i`번을 건너뛰면 `(i-1→i)`, `(i→i+1)`이 빠지고 `(i-1→i+1)`이 생긴다.

```java
long total = 0;
for (int i = 1; i < n; i++) total += dist(i - 1, i);

long best = Long.MAX_VALUE;
for (int i = 1; i < n - 1; i++) {
    long changed = total - dist(i - 1, i) - dist(i, i + 1) + dist(i - 1, i + 1);
    best = Math.min(best, changed);
}
```

시간이 O(N²) → O(N)이다. 이 문제는 N이 작아 둘 다 통과하지만, 다음에 N이 큰 문제가 나오면 이 방식이 필요하다.
