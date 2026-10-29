---
title: "[프로그래머스-Lv0] 진료순서 정하기 - 정렬한 배열과 원래 배열을 구분한다"
date: 2026-10-29 09:30:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, hashmap, rank]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/120835
- 파일 경로: `src/main/java/coding_test/정렬/Lv0_진료순서정하기.java`
- 난이도: Lv.0

## 문제 설명

응급도가 높은 환자부터 진료한다. 각 환자가 **몇 번째로 진료받는지**를 **원래 자리에** 적어 반환한다. 응급도는 겹치지 않는다.

```
[3, 76, 24]          → [3, 1, 2]
[30, 10, 23, 6, 100] → [2, 4, 3, 5, 1]
```

정렬은 하지만, 반환하는 건 정렬된 배열이 아니라 **원래 자리 기준의 순위**다.

## 시행착오

Lv0인데 네 번 고쳤다. 실수가 모두 같은 뿌리에서 나왔다. **지금 들고 있는 게 값인지 번호인지, 정렬된 배열인지 원래 배열인지**를 섞었다.

### 1차: `temp`에는 값, `map`의 key는 번호

```java
map.put(i, emergency[i]);   // 번호 → 응급도
temp[i] = emergency[i];     // 값
Arrays.sort(temp, (a, b) -> map.get(b) - map.get(a));
```

lambda의 `a`, `b`는 값(3, 76, 24)인데 `map`의 key는 번호(0, 1, 2)다. `map.get(76)`은 `null`이라 오류가 났다.

### 2차: `temp`를 번호표로 → 정렬은 성공, 출력이 `[1, 2, 0]`

`temp[i] = i`로 바꾸자 정렬은 맞았다. 그런데 이 결과는 "**몇 등이 몇 번 환자인지**"다(1등은 1번, 2등은 2번, 3등은 0번). 문제가 원하는 건 반대로 "**몇 번 환자가 몇 등인지**"다.

### 3차: 같은 루프에서 `put`하고 `get` → NullPointerException

```java
map.put(temp[i], i);
answer[i] = map.get(emergency[i]);
```

key로 넣은 건 번호인데 꺼낼 때는 값을 썼다. 게다가 아직 다 넣기도 전에 꺼냈다.

### 4차: 값 → 순위 map으로 전환, 결과가 항상 `[1, 2, 3, ...]`

방향을 바꿔 값을 내림차순 정렬하고 `map.put(값, 순위)`로 만들었다. 응급도가 겹치지 않아서 값을 key로 써도 안전하다. 그런데 마지막에 꺼낼 때 **정렬된 `temp`**를 돌아서, 정렬된 순서대로 1, 2, 3...이 나왔다.

### 5차: 통과

마지막 루프에서 원래 순서가 남아 있는 `emergency`를 돌도록 고쳐 통과했다.

## 최종 코드

```java
public int[] solution(int[] emergency) {
    Map<Integer, Integer> map = new HashMap<>();
    Integer[] temp = new Integer[emergency.length];
    for (int i = 0; i < emergency.length; i++) {
        temp[i] = emergency[i];
    }

    Arrays.sort(temp, (a, b) -> b - a);              // 응급도 내림차순

    int[] answer = new int[temp.length];
    for (int i = 0; i < temp.length; i++) {
        map.put(temp[i], i + 1);                     // 응급도 → 순위
    }

    for (int i = 0; i < temp.length; i++) {
        answer[i] = map.get(emergency[i]);           // 원래 순서대로 순위 꺼내기
    }
    return answer;
}
```

## 추적표: `[3, 76, 24]`

| 단계 | 결과 |
|---|---|
| 내림차순 정렬 | `[76, 24, 3]` |
| map | `{76: 1, 24: 2, 3: 3}` |
| `emergency` 순서로 꺼내기 | 3 → 3, 76 → 1, 24 → 2 → `[3, 1, 2]` |

## 오늘 배운 내용

**순위 문제는 배열이 두 개다.** "누가 몇 등인지"를 알려면 정렬된 배열(순위 계산용)과 원래 배열(답을 적는 순서)이 둘 다 필요하다. 결과를 꺼낼 때는 **원래 배열**을 돈다.

## 오답노트

- **틀렸던 지점**: map에 넣을 때와 꺼낼 때 key 종류가 달랐고(번호 vs 값), 마지막 루프에서 정렬된 배열을 돌았다. **왜**: 변수마다 "무엇이 들어 있는지"를 정하지 않고 코드를 고쳤다. **다음에**: 변수를 만들 때 옆에 `// 번호` 또는 `// 값`, `// 정렬됨` 또는 `// 원래 순서`를 주석으로 적는다. `map.get(x)`를 쓸 때는 x가 put할 때의 key와 같은 종류인지 본다.

## AI라면 어떻게 풀었을까

**번호표(인덱스)를 정렬하고, 그 환자 칸에 순위를 바로 적는다.** [LC2418]({% post_url 2026-10-25-sort-the-people-lc2418 %})과 같은 번호표 정렬이다.

```java
public int[] solutionAlt(int[] emergency) {
    Integer[] idx = new Integer[emergency.length];
    for (int i = 0; i < idx.length; i++) {
        idx[i] = i;                                              // 환자 번호표
    }
    Arrays.sort(idx, (a, b) -> emergency[b] - emergency[a]);     // 응급도 높은 번호가 앞
    int[] answer = new int[emergency.length];
    for (int i = 0; i < idx.length; i++) {
        answer[idx[i]] = i + 1;                                  // 그 환자 칸에 순위
    }
    return answer;
}
```

정렬 결과 `idx[i]`는 "i+1등인 환자 번호"다. 그래서 `answer[idx[i]] = i + 1`이면 "그 환자는 i+1등"이 된다. 2차 시도에서 `[1, 2, 0]`이 나왔을 때 이 한 줄만 있으면 끝났다.

트레이드오프: map이 필요 없다. 또 응급도가 겹쳐도 쓸 수 있다. 본인 풀이는 "응급도 중복 없음" 조건이 있어야 맞는다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
