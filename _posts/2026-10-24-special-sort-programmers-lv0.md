---
title: "[프로그래머스-Lv0] 특이한 정렬 - 로컬은 통과, 제출은 실패: Integer != 함정"
date: 2026-10-24 19:40:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, comparator, integer-cache]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/120880
- 파일 경로: `src/main/java/coding_test/정렬/Lv0_특이한정렬.java`
- 난이도: Lv.0

## 문제 설명

`numlist`를 **n과 가까운 순서**로 정렬한다. 거리가 같으면 **더 큰 수가 앞**이다. 원소는 1 ~ 10,000이고 중복이 없다.

```
[1,2,3,4,5,6], n = 4                      → [4,5,3,6,2,1]
[10000,20,36,47,40,6,10,7000], n = 30     → [36,40,20,47,10,6,7000,10000]
```

정렬 Comparator만 따로 연습하려고 고른 드릴 문제다. LC1636(빈도 → 같으면 값 내림차순)과 모양이 같다.

## 시행착오

### 1차: 로컬 7개 통과, 프로그래머스 제출은 실패

```java
Arrays.sort(temp, (a, b) -> map.get(a) != map.get(b) ? map.get(a) - map.get(b) : b - a);
```

이번에는 **Comparator 모양이 처음부터 맞았다.** `?` 뒤(거리가 다를 때)도 `:` 뒤(거리가 같을 때)도 a와 b의 비교식이다. 최근 세 문제 연속으로 나왔던 "한쪽에 값 하나만 반환" 실수가 없었다.

그런데 제출하면 틀렸다. 원인은 `!=`였다.

```java
map.put(1, 100); map.put(2, 100);
map.get(1) != map.get(2)   // false (맞음)

map.put(1, 200); map.put(2, 200);
map.get(1) != map.get(2)   // true  (값은 같은데 "다르다")
```

`map.get()`이 돌려주는 건 `Integer` **객체**다. 객체끼리 `!=`는 값이 아니라 같은 객체인지를 본다. 자바는 -128 ~ 127만 같은 객체를 재사용하기 때문에 그 범위에서만 우연히 맞는다. 이 문제는 거리가 최대 9,999라서, `n = 500`, `[300, 700]`처럼 거리가 같은데 128 이상이면 "다르다"로 처리돼 `[300, 700]`이 나온다(정답 `[700, 300]`). 로컬 테스트는 거리가 전부 작아서 걸러지지 않았다.

LC1636에서도 같은 `!=`를 썼는데, 빈도가 최대 100이라 운 좋게 통과했었다.

### 2차: 통과

```java
!map.get(a).equals(map.get(b)) ? map.get(a) - map.get(b) : b - a
```

`.equals()`로 값을 비교하도록 고쳐 통과했다.

## 최종 코드

```java
public int[] solution(int[] numlist, int n) {
    int[] answer = new int[numlist.length];
    HashMap<Integer, Integer> map = new HashMap<>();
    Integer[] temp = new Integer[numlist.length];
    for (int i = 0; i < numlist.length; i++) {
        temp[i] = numlist[i];
        map.put(numlist[i], Math.abs(n - numlist[i]));   // 숫자 → n과의 거리
    }

    Arrays.sort(temp, (a, b) -> !map.get(a).equals(map.get(b))
            ? map.get(a) - map.get(b)     // 거리가 다르면: 가까운 게 앞
            : b - a);                     // 거리가 같으면: 큰 수가 앞

    for (int i = 0; i < temp.length; i++) {
        answer[i] = temp[i];
    }
    return answer;
}
```

랜덤 20,000건(거리 128 이상 포함)에서 아래 AI 풀이와 결과가 같았다.

## 추적표: `[1,2,3,4,5,6], n = 4`

| a | b | 거리 a / b | 어느 쪽 | 결과 | 앞에 오는 것 |
|---|---|---|---|---|---|
| 3 | 5 | 1 / 1 | `:` (같음) | `5 - 3 = 2` | 5 |
| 4 | 6 | 0 / 2 | `?` (다름) | `0 - 2 = -2` | 4 |
| 2 | 6 | 2 / 2 | `:` (같음) | `6 - 2 = 4` | 6 |
| 1 | 2 | 3 / 2 | `?` (다름) | `3 - 2 = 1` | 2 |

→ `[4, 5, 3, 6, 2, 1]`

## 오늘 배운 내용

**`HashMap<Integer, Integer>`에서 꺼낸 값끼리는 `==`/`!=`로 비교하지 않는다.** `map.get()`은 `Integer` 객체를 돌려주고, 객체의 `==`는 값이 아니라 "같은 객체인지"를 본다. -128 ~ 127에서는 우연히 맞아서 작은 예시로는 절대 안 잡힌다. 값을 비교할 때는 `.equals()`를 쓰거나 `int` 변수에 먼저 꺼낸다.

## 오답노트

- **틀렸던 지점**: `map.get(a) != map.get(b)`. **왜**: `map.get()`의 결과를 `int`처럼 생각했다. LC1636에서 같은 코드가 통과해서 문제가 있는 줄 몰랐다. **어떻게 고쳤나**: `!map.get(a).equals(map.get(b))`. **다음에**: 제한사항에서 값 범위가 127을 넘을 수 있는지 보고, `Integer`(대문자)가 보이면 `==` 대신 `.equals()`를 떠올린다. 로컬 테스트에 큰 값 케이스를 하나 넣는다.

## 꿀팁

**로컬 통과·제출 실패면 "작은 값에서만 맞는 코드"부터 의심한다.** `Integer` 캐시(-128 ~ 127), `int` 넘침, 범위 끝값처럼 작은 예시로는 안 드러나는 문제가 대부분이다. 제한사항의 최댓값을 넣은 테스트를 하나 만들어 돌려 본다.

## AI라면 어떻게 풀었을까

**거리를 저장하지 않고 Comparator 안에서 바로 계산한다.** 거리는 `Math.abs(x - n)`으로 언제든 구할 수 있어서 `HashMap`이 필요 없다. 결과가 `int`라 `Integer` `!=` 함정도 생기지 않는다. 두 기준은 `comparing` → `thenComparing`으로 잇는다.

```java
public int[] solutionAlt(int[] numlist, int n) {
    Integer[] temp = new Integer[numlist.length];
    for (int i = 0; i < numlist.length; i++) {
        temp[i] = numlist[i];
    }
    Arrays.sort(temp, Comparator.comparing((Integer x) -> Math.abs(x - n))   // 거리 오름차순
            .thenComparing(Comparator.reverseOrder()));                     // 같으면 큰 수가 앞
    int[] answer = new int[numlist.length];
    for (int i = 0; i < numlist.length; i++) {
        answer[i] = temp[i];
    }
    return answer;
}
```

트레이드오프: 코드가 짧고 객체 비교 함정이 없다. 대신 비교할 때마다 `Math.abs`를 다시 계산한다. 이 문제는 원소가 100개 이하라 차이가 없다. 계산이 무거운 기준(예: 문자열 파싱)이라면 본인 풀이처럼 미리 계산해 두는 편이 낫다.
