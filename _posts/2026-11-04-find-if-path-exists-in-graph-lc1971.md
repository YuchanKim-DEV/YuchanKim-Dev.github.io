---
title: "[LeetCode-1971] Find if Path Exists in Graph - 간선 목록을 이웃 목록으로"
date: 2026-11-04 11:01:00 +0900
categories: [알고리즘 연습, DFS_BFS]
tags: [java, leetcode, bfs, graph, adjacency-list]
---

- 문제 링크: https://leetcode.com/problems/find-if-path-exists-in-graph/
- 파일 경로: `src/main/java/coding_test/DFS_BFS_그래프/LC1971_FindIfPathExistsInGraph.java`
- 난이도: Easy

## 문제 설명

0번부터 n-1번까지 정점이 있는 **양방향** 그래프다. `edges[i] = [u, v]`는 u와 v를 잇는 간선이다. `source`에서 `destination`까지 가는 경로가 있으면 `true`를 반환한다.

```
n=3, [[0,1],[1,2],[2,0]], 0→2          → true
n=6, [[0,1],[0,2],[3,5],[5,4],[4,3]], 0→5  → false  (끊어진 그래프)
```

어제 [Flood Fill]({% post_url 2026-11-03-flood-fill-lc733 %})에서 BFS 틀을 처음 썼지만 "새 요소가 너무 많다"는 문제가 있었다. 그래서 오늘은 **같은 틀에 요소를 하나씩만 바꾸는 사다리**로 갔다.

## 시행착오

### 0차: 사다리 3단계 (틀 고정, 바뀌는 것은 하나씩)

| 단계 | 바뀐 것 | 결과 |
|---|---|---|
| BFS 방문 순서 | 틀 익히기 | 통과 |
| BFS 도달 가능 | 꺼낸 정점이 target인가 | 통과 (처음엔 `start`로 되돌아오는지를 봄) |
| BFS 거리 구하기 | 거리 기록(`Arrays.fill(-1)` 배열이 방문표를 겸함) | 통과 |

사다리 중에 **`graph[i]`의 `i`는 노드 번호**이고 `graph[i]`는 그 노드의 이웃 목록이라는 것을 확인했다. 줄 하나 = 정점 하나라서, 줄 수 = 정점 수다.

### 1차: `edges`를 `graph`처럼 씀 → ArrayIndexOutOfBounds

```java
for (int next : edges[current]) { ... }   // edges[5] 에러
```

`edges`와 `graph`는 둘 다 `int[][]`지만 줄의 뜻이 다르다.

| | `graph` | `edges` |
|---|---|---|
| 줄 하나 | 정점 하나의 **이웃 목록** | **간선 하나** `[u, v]` |
| 줄 수 | 정점 수 | 간선 수 |

그래서 BFS 전에 `edges`로 `graph`를 먼저 만들어야 했다. 정점마다 길이가 다르니 `List<List<Integer>>`를 썼다.

### 2차: 간선 번호 `i`와 정점 번호를 섞음 → 또 ArrayIndexOutOfBounds

```java
graph.get(i).add(edges[i][i]);   // edges[2][2] → 에러
graph.get(edges[i][1]).add(i);
```

이 `for`의 `i`는 **몇 번째 간선인지**이고 정점 번호가 아니다. 간선 한 개에서 양 끝 정점은 `edges[i][0]`, `edges[i][1]`로 꺼내야 한다. 안쪽 `for j`도 간선을 두 번 처리하게 만들어서 지웠다.

## 최종 코드

```java
public boolean validPath(int n, int[][] edges, int source, int destination) {
    Queue<Integer> queue = new LinkedList<>();
    boolean[] visited = new boolean[n];

    List<List<Integer>> graph = new ArrayList<>();
    for (int i = 0; i < n; i++) {
        graph.add(new ArrayList<>());          // 정점마다 빈 이웃 목록
    }

    for (int i = 0; i < edges.length; i++) {
        int u = edges[i][0];
        int v = edges[i][1];
        graph.get(u).add(v);                   // 양방향이라 둘 다 추가
        graph.get(v).add(u);
    }

    queue.add(source);
    visited[source] = true;
    while (!queue.isEmpty()) {
        int cur = queue.poll();
        if (cur == destination) {
            return true;                       // 꺼낸 정점이 도착지면 끝
        }

        for (int next : graph.get(cur)) {
            if (!visited[next]) {
                visited[next] = true;
                queue.add(next);
            }
        }
    }

    return false;                              // 큐가 비도록 못 찾음
}
```

## 추적표: `n=6, [[0,1],[0,2],[3,5],[5,4],[4,3]]`, 0 → 5

`graph`는 이렇게 만들어진다: `0:{1,2}, 1:{0}, 2:{0}, 3:{5,4}, 4:{5,3}, 5:{3,4}`

| 단계 | 꺼낸 정점 | 새로 넣은 정점 | 큐 |
|---|---|---|---|
| 시작 | | 0 | [0] |
| 1 | 0 | 1, 2 | [1, 2] |
| 2 | 1 | 없음 (0은 방문함) | [2] |
| 3 | 2 | 없음 | [] |

큐가 비었는데 5를 못 꺼냈으니 `false`다. 0이 속한 덩어리 {0,1,2}에는 5가 없다.

## 오늘 배운 내용

- **간선 목록 → 이웃 목록 바꾸기:** 간선 하나 `[u, v]`마다 `u`의 이웃에 `v`, `v`의 이웃에 `u`를 넣는다(양방향). 그 뒤는 평소 BFS다.
- **`graph[i]`의 `i`는 정점 번호, `edges[i]`의 `i`는 간선 번호.** 둘을 섞으면 `ArrayIndexOutOfBounds`가 난다.
- **BFS 틀은 하나다.** 방문 순서, 도달 여부, 거리, 경로 존재가 전부 같은 틀에서 "꺼낸 뒤 무엇을 하는가"만 다르다.
- **`List<List<Integer>>` 문법:** 만들 때 정점 수만큼 `new ArrayList<>()`를 먼저 채우고(`graph.add(new ArrayList<>())`), 이웃은 `graph.get(a).add(b)`로 넣는다.

## 오답노트

- **막혔던 지점**: 간선 번호와 정점 번호를 같은 `i`로 섞었다. **다음에**: `for (int i ...)`의 `i`가 무엇을 가리키는지 주석으로 적어 둔다("i번째 간선").
- **실수**: `edges`를 `graph`처럼 `edges[정점번호]`로 읽었다. **다음에**: 입력의 `[i]`가 정점인지 간선인지 문제를 읽자마자 확인한다.

## AI라면 어떻게 풀었을까

이 문제는 "경로가 있는가"만 묻는다. 경로 자체나 거리는 필요 없다. 그러면 BFS 대신 **"source와 destination이 같은 덩어리인가"**만 알면 되고, 이웃 목록을 만들지 않는 Union-Find로 풀 수 있다.

```java
public boolean validPathAlt(int n, int[][] edges, int source, int destination) {
    int[] parent = new int[n];
    for (int i = 0; i < n; i++) {
        parent[i] = i;                                   // 처음엔 각자 자기 자신이 대표
    }
    for (int[] e : edges) {
        int a = find(parent, e[0]), b = find(parent, e[1]);
        if (a != b) {
            parent[a] = b;                               // 두 덩어리를 하나로 합침
        }
    }
    return find(parent, source) == find(parent, destination);
}

private int find(int[] parent, int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]];                   // 경로를 줄이며 대표를 찾음
        x = parent[x];
    }
    return x;
}
```

트레이드오프: 둘 다 시간은 O(n + 간선 수)다. BFS는 이웃 목록 메모리가 필요하고, Union-Find는 부모 배열 하나면 된다. 하지만 Union-Find는 아직 배우지 않았고, BFS 틀은 거리나 경로가 필요한 문제로 그대로 이어지니 지금은 BFS가 맞다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
