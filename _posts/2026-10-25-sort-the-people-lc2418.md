---
title: "[LeetCode-2418] Sort the People - 이름 말고 번호표를 정렬한다"
date: 2026-10-25 17:25:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, leetcode, sort, comparator, index-sort]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/sort-the-people/
- 파일 경로: `src/main/java/coding_test/정렬/LC2418_SortThePeople.java`
- 난이도: Easy

## 문제 설명

이름 배열 `names`와 키 배열 `heights`가 주어진다. 같은 인덱스가 한 사람이다. **키가 큰 사람부터** 이름을 담아 반환한다. 키는 모두 다르지만 **이름은 겹칠 수 있다.**

```
["Mary","John","Emma"], [180,165,170]  → [Mary, Emma, John]
["Alice","Bob","Bob"],  [155,185,150]  → [Bob, Alice, Bob]
```

Comparator 드릴 마지막 문제다. 기준은 하나뿐이고, 새로운 건 **정렬 대상이 값이 아니라 인덱스**라는 점이었다.

## 시행착오

### 1차: 이름 → 키 HashMap, 예시 2만 실패

```java
HashMap<String, Integer> map = new HashMap<>();
for (int i = 0; i < names.length; i++) map.put(names[i], heights[i]);
Arrays.sort(names, (a, b) -> map.get(b) - map.get(a));
```

7개 중 6개는 맞았는데 `["Alice","Bob","Bob"]`만 틀렸다. `map.put("Bob", 185)` 다음에 `map.put("Bob", 150)`이 와서 **185가 덮어써졌다.** 두 Bob이 모두 150이 되니 순서가 깨진다. 이름은 겹칠 수 있으니 key로 쓰면 안 된다.

### 2차: 번호표 정렬, 방향과 덮어쓰기 실수

```java
Arrays.sort(idx, Comparator.comparing((Integer i) -> heights[i]));
for (int i = 0; i < names.length; i++) names[i] = names[idx[i]];
```

출력이 `[John, Emma, John]`, `[Bob, Bob, Bob]`처럼 나왔다. 실수가 두 개였다.

1. `comparing`은 기본이 **오름차순**이라 키 작은 사람이 앞에 왔다.
2. `names`를 읽으면서 **같은 `names`에 덮어썼다.** `names[0] = names[1]`이 실행되는 순간 Mary가 사라지고, 뒤에서 `names[0]`을 읽으면 이미 바뀐 값이 나온다.

### 3차: 통과

내림차순 lambda로 바꾸고, 결과는 새 배열 `answer`에 담아 통과했다.

## 최종 코드

```java
public String[] sortPeople(String[] names, int[] heights) {
    Integer[] idx = new Integer[names.length];          // 사람 번호표
    for (int i = 0; i < names.length; i++) {
        idx[i] = i;
    }

    Arrays.sort(idx, (a, b) -> heights[b] - heights[a]); // a, b는 번호. 키 큰 번호가 앞

    String[] answer = new String[names.length];          // 원본을 덮어쓰지 않게 새 배열
    for (int i = 0; i < names.length; i++) {
        answer[i] = names[idx[i]];
    }
    return answer;
}
```

## 추적표: `["Alice","Bob","Bob"], [155,185,150]`

| 번호 | 이름 | 키 |
|---|---|---|
| 0 | Alice | 155 |
| 1 | Bob | 185 |
| 2 | Bob | 150 |

번호표 `[0, 1, 2]`를 키 내림차순으로 정렬하면 `[1, 0, 2]`이고, 번호로 이름을 꺼내면 `[Bob, Alice, Bob]`이다. 두 Bob은 번호가 달라서 섞이지 않는다.

## 오늘 배운 내용

**같은 인덱스로 묶인 배열 두 개는 "번호표(인덱스) 배열"을 정렬한다.** `Integer[] idx = {0, 1, 2, ...}`를 만들고 lambda 안에서 `heights[a]`처럼 다른 배열을 본다. 이때 lambda의 `a`, `b`는 값이 아니라 번호다. 이름처럼 겹칠 수 있는 값을 HashMap key로 쓰면 덮어써져서 틀린다.

## 오답노트

- **틀렸던 지점 1**: `HashMap<String, Integer>`에 이름을 key로 넣었다. **왜**: 이름이 겹칠 수 있다는 걸 놓쳤다. **다음에**: HashMap key로 쓸 값은 제한사항에서 "중복 없음"인지 먼저 확인한다.
- **틀렸던 지점 2**: `names[i] = names[idx[i]]`로 읽는 배열에 바로 썼다. **왜**: 앞에서 바꾼 칸을 뒤에서 다시 읽는다는 걸 생각하지 못했다. **다음에**: 다른 순서로 다시 담을 때는 결과용 새 배열을 만든다.
- **틀렸던 지점 3**: `Comparator.comparing(...)`만 쓰고 내림차순을 놓쳤다. **다음에**: "큰 게 앞"이면 `.reversed()`를 붙이거나 `b - a` 쪽으로 쓴다.

## 꿀팁

**HashMap key는 "겹치지 않는 쪽"으로 잡는다.** 이 문제에서 이름은 겹치지만 키는 모두 다르다. 그래서 `키 → 이름` 방향이면 HashMap도 안전하다(아래 AI 풀이).

## AI라면 어떻게 풀었을까

번호표 정렬이 정석이고, 본인 최종 코드가 그 방식이다. 다른 방법은 겹치지 않는 **키를 key로** 두는 것이다.

```java
public String[] sortPeopleAlt(String[] names, int[] heights) {
    HashMap<Integer, String> map = new HashMap<>();      // 키 → 이름 (키는 안 겹침)
    for (int i = 0; i < names.length; i++) {
        map.put(heights[i], names[i]);
    }
    int[] sorted = heights.clone();
    Arrays.sort(sorted);                                   // int[]라 Comparator 없이 오름차순
    String[] answer = new String[names.length];
    for (int i = 0; i < names.length; i++) {
        answer[i] = map.get(sorted[names.length - 1 - i]); // 뒤에서부터 = 큰 키부터
    }
    return answer;
}
```

트레이드오프: Comparator 없이 `int[]` 기본 정렬만 쓴다. 대신 "키가 모두 다르다"는 조건이 있어야만 맞다. 번호표 정렬은 값이 겹쳐도 항상 쓸 수 있어서 더 범용적이다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
