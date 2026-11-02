---
title: "[LeetCode-1791] Find Center of Star Graph - 37ms에서 1ms로, 간선 2개면 끝"
date: 2026-11-02 09:43:00 +0900
categories: [알고리즘 연습, 그래프]
tags: [java, leetcode, graph, degree]
---

- 문제 링크: https://leetcode.com/problems/find-center-of-star-graph/
- 파일 경로: `src/main/java/coding_test/그래프/LC1791_FindCenterOfStarGraph.java`
- 난이도: Easy

## 문제 설명

별 모양 그래프는 **중심 노드 하나**가 나머지 모든 노드와 간선으로 이어진 그래프다. `edges[i] = [u, v]`는 u와 v가 연결돼 있다는 뜻이다. 중심 노드 번호를 반환한다.

```
[[1,2],[2,3],[4,2]]        → 2
[[1,2],[5,1],[1,3],[1,4]]  → 1
```

그래프 유형의 첫 문제다. 그래프를 "간선 목록"으로 받는 형태에 익숙해지려고 골랐다.

## 시행착오

### 1차: Map으로 개수 세기 → 통과, 그런데 운 좋게

노드마다 몇 개의 간선에 나오는지 `HashMap`으로 셌다. 그다음 가장 많이 나온 노드를 찾았다.

```java
int ans = 0;
for (Map.Entry<Integer, Integer> e : map.entrySet()) {
    if (ans < e.getValue()) {      // ans는 노드 번호, value는 개수
        ans = e.getKey();
    }
}
```

테스트는 다 통과했다. 그런데 **노드 번호(`ans`)와 개수(`value`)를 비교**하고 있었다. 이 문제는 번호가 1~n이고 중심의 개수가 항상 n-1이라서 우연히 맞았다. "가장 많이 나온 것 찾기"에 그대로 쓰면 틀린다.

### 2차: `map.get(ans)` → NullPointerException 위험

개수끼리 비교하려고 `map.get(ans) < value`로 바꿨다. 그런데 처음 `ans = 0`이고 0번 노드는 Map에 없다. `map.get(0)`은 `null`이고, `null`을 int로 바꾸는 순간 터진다. `map.getOrDefault(ans, 0)`으로 고쳤다.

### 3차: 37ms → 1ms

Map 풀이는 37ms가 걸렸는데, 1ms로 끝내는 풀이가 있었다. 핵심은 문제의 모양 자체였다.

- 중심은 모든 노드와 이어져 있다 → **중심은 모든 간선에 들어 있다.**
- 첫 간선 `[1,2]`만 봐도 중심은 1 아니면 2다.
- 두 번째 간선에도 있는 쪽이 중심이다. 바깥 노드는 간선 하나에만 나온다.

**간선이 10만 개여도 2개만 보면 된다.**

## 최종 코드

```java
public int findCenter(int[][] edges) {
    int temp01 = edges[0][0];
    int temp02 = edges[0][1];
    int temp03 = edges[1][0];
    int temp04 = edges[1][1];

    int ans = 0;
    if (temp01 == temp03 || temp01 == temp04) {      // 첫 간선의 앞 노드가 두 번째 간선에도 있으면
        ans = temp01;
    } else if (temp02 == temp03 || temp02 == temp04) {
        ans = temp02;
    }
    return ans;
}
```

## 추적표

| 입력 | 첫 간선 후보 | 두 번째 간선 | 겹치는 노드 |
|---|---|---|---|
| [[1,2],[2,3],[4,2]] | 1, 2 | [2,3] | **2** |
| [[1,2],[5,1],[1,3],[1,4]] | 1, 2 | [5,1] | **1** |
| [[5,3],[3,1],[2,3],[3,4]] | 5, 3 | [3,1] | **3** |

## 오늘 배운 내용

- **그래프에서 "개수 세기" = 차수(degree).** 노드가 몇 개의 간선에 나오는지 센 것이 그 노드의 차수다. 별 그래프의 중심은 차수가 n-1인 노드다. 다음 문제(Find the Town Judge)도 같은 개수 세기다.
- **문제의 모양이 주는 조건을 먼저 본다.** "올바른 별 그래프만 들어온다"는 조건 덕분에 다 셀 필요가 없었다. 전부 세는 풀이를 먼저 짠 뒤, "이 조건이 있으면 덜 봐도 되나?"를 묻는다.
- **37ms vs 1ms**: HashMap은 넣을 때마다 박싱(int → Integer)과 해시 계산이 들어가 느리다. 그래도 둘 다 O(n) 이하라 코테 통과에는 문제가 없다.

## 오답노트

- **실수 1**: 최댓값을 찾을 때 **번호와 개수를 한 변수로 비교**했다. **다음에**: 최댓값 찾기는 변수 2개, `max`(가장 큰 개수)와 `ans`(그때의 번호)를 둔다. 더 크면 둘 다 바꾼다.
- **실수 2**: Map에 없을 수 있는 키를 `map.get()`으로 꺼내 비교했다. **다음에**: 없을 수 있으면 `getOrDefault`를 쓴다.

## AI라면 어떻게 풀었을까

본인 최종 풀이와 같다. 첫 노드가 중심이 아니면 남은 하나는 무조건 중심이라서 `else if`를 한 줄로 줄일 수 있다.

```java
public int findCenterAlt(int[][] edges) {
    int a = edges[0][0];
    return (a == edges[1][0] || a == edges[1][1]) ? a : edges[0][1];
}
```

Map 풀이를 쓴다면 최댓값 찾기는 이렇게 한다.

```java
int max = 0, ans = 0;
for (Map.Entry<Integer, Integer> e : map.entrySet()) {
    if (e.getValue() > max) {
        max = e.getValue();
        ans = e.getKey();
    }
}
```

트레이드오프: 2개만 보는 풀이는 "올바른 별 그래프"라는 조건에 기대므로 조건이 바뀌면 못 쓴다. 개수 세기 풀이는 느리지만 어떤 그래프에도 쓸 수 있다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
