---
title: "[LeetCode-1636] 빈도순 정렬 - Comparator의 ? : 양쪽은 모두 'a와 b의 비교'다"
date: 2026-10-23 21:54:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, leetcode, sort, comparator, hash]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/sort-array-by-increasing-frequency/
- 파일 경로: `src/main/java/coding_test/정렬/LC1636_SortArrayByIncreasingFrequency.java`
- 난이도: Easy

## 문제 설명

정수 배열 `nums`를 **나온 횟수(빈도)가 적은 값부터** 정렬한다. 빈도가 같으면 **값이 큰 것부터** 놓는다. 길이는 최대 100, 값은 -100 ~ 100이다.

```
[1,1,2,2,2,3]           → [3,1,1,2,2,2]
[2,3,1,3,2]             → [1,3,3,2,2]     (2와 3은 둘 다 2번 → 큰 3이 앞)
[-1,1,-6,4,5,-6,1,4,1]  → [5,-1,4,4,-6,-6,1,1,1]
```

## 시행착오

전날(10/22) 저녁에 처음 시도했다가 지쳐서 중단했다. 마지막 비교식은 그때 이미 제시받았기 때문에, 다음 날 메서드를 백지로 비우고 다시 풀었다.

### 1일차 (10/22)

**1차: `int[]`에 Comparator**

```java
Arrays.sort(nums, (a, b) -> b - a);    // nums는 int[] → 컴파일 에러
```

Comparator는 칸에 **객체**가 든 배열(`String[]`, `Integer[]`, `int[][]`)에만 쓸 수 있다. `int[]`는 `Integer[]`로 옮겨서 정렬하고, 끝나면 다시 `int[]`로 되돌린다.

**2차: `map.getValue(a)` — HashMap에 없는 메서드**

```java
Arrays.sort(arr, (a, b) -> map.getValue(b) - map.getValue(a));
```

키로 값을 꺼내는 건 `map.get(키)`다. `getKey()`, `getValue()`는 `entrySet()`을 돌 때 나오는 `Map.Entry`의 메서드다. 이 식은 방향(횟수 많은 게 앞)도 반대였고, 두 번째 기준도 없었고, 정렬한 `arr`이 아니라 `nums`를 그대로 반환하고 있었다.

**3차: `? :` 양쪽에 무엇을 넣는지 헷갈림**

```java
// 횟수가 같을 때 횟수끼리 또 뺌 → 항상 0
map.get(a).equals(map.get(b)) ? map.get(b) - map.get(a) : map.get(a) - map.get(b)
// 맞던 쪽(:)을 고쳐 버림
map.get(a).equals(map.get(b)) ? map.get(b) - map.get(a) : map.get(a)
// 양쪽 다 값 하나만 반환
map.get(a).equals(map.get(b)) ? a : map.get(a) - map.get(b)
```

여기서 지쳐서 중단했다.

### 2일차 (10/23, 백지 재도전)

빈도 세기, `Integer[]` 변환, `int[]`로 되돌리기는 막힘 없이 다시 짰다. 막힌 곳은 또 Comparator 한 줄이었다.

```java
map.get(a) != map.get(b) ? map.get(a) : map.get(a) - map.get(b)       // ① ? 뒤에 a 쪽만
map.get(a) != map.get(b) ? map.get(a) - map.get(b) : map.get(a)       // ② : 뒤에 a 쪽만
map.get(a) != map.get(b) ? map.get(a) - map.get(b) : b - a            // ③ 통과
```

①에서 조건을 `!=`로 바꾸면서 `?`와 `:`의 역할이 뒤바뀌었다. 횟수 비교식을 `?` 뒤로 옮기자 7개가 통과했고(②), 틀린 3개는 모두 **빈도가 같은 값끼리의 순서**였다. `:` 뒤를 값 비교(`b - a`)로 바꾸자 10개가 모두 통과했다.

## 최종 코드

```java
public int[] frequencySort(int[] nums) {
    Map<Integer, Integer> map = new HashMap<>();
    Integer[] arr = new Integer[nums.length];
    for (int i = 0; i < nums.length; i++) {
        map.put(nums[i], map.getOrDefault(nums[i], 0) + 1);   // 빈도 세기
        arr[i] = nums[i];                                     // int → Integer
    }

    Arrays.sort(arr, (a, b) -> map.get(a) != map.get(b)
            ? map.get(a) - map.get(b)     // 빈도가 다르면: 적은 게 앞
            : b - a);                     // 빈도가 같으면: 값이 큰 게 앞

    for (int i = 0; i < arr.length; i++) {
        nums[i] = arr[i];                                     // Integer → int
    }
    return nums;
}
```

랜덤 30,000건에서 아래 AI 풀이와 결과가 같았다.

## 추적표: `[2, 3, 1, 3, 2]`

빈도: `{1=1, 2=2, 3=2}`

| a | b | 빈도 a / b | 어느 쪽 | 결과 | 앞에 오는 것 |
|---|---|---|---|---|---|
| 1 | 2 | 1 / 2 | `?` (다름) | `1 - 2 = -1` | 1 |
| 2 | 3 | 2 / 2 | `:` (같음) | `3 - 2 = 1` | 3 |
| 3 | 1 | 2 / 1 | `?` (다름) | `2 - 1 = 1` | 1 |

→ `[1, 3, 3, 2, 2]`

## 오늘 배운 내용

**Comparator는 무엇을 돌려주든 항상 "a와 b를 비교한 결과"여야 한다.** 음수면 a가 앞, 양수면 b가 앞, 0이면 상관없다. 그래서 `조건 ? X : Y`로 기준을 나눌 때 **X와 Y 둘 다 a와 b가 함께 들어간 비교식**이어야 한다. `map.get(a)`나 `a`처럼 한쪽 값만 돌려주면 정렬은 둘 중 누가 앞인지 알 수 없다.

## 오답노트

- **틀렸던 지점 1**: `int[]`에 Comparator. **왜**: 문자열 문제에서 쓴 모양을 그대로 가져왔다. **다음에**: 기본형 배열(`int[]`, `char[]`, `long[]`)은 `Integer[]` 등으로 옮긴 뒤 정렬한다.
- **틀렸던 지점 2**: `map.getValue(a)`, `map.getKey(a)`. **왜**: `Entry`의 메서드와 `HashMap`의 메서드를 섞었다. **다음에**: Comparator 안의 `a`, `b`는 배열 칸의 숫자 그 자체다. 빈도는 `map.get(a)`, 값은 `a` 그대로다.
- **틀렸던 지점 3**: `?` 또는 `:` 뒤에 한쪽 값만 반환. **왜**: "a의 빈도로 정렬한다"를 "a의 빈도를 돌려준다"로 옮겼다. **다음에**: `?` 뒤와 `:` 뒤에 a와 b가 둘 다 있는지 눈으로 확인한다.
- **틀렸던 지점 4**: 조건을 `==` ↔ `!=`로 바꾸면서 양쪽 역할이 뒤바뀜. **왜**: 조건만 바꾸고 `?`, `:` 뒤의 식은 그대로 뒀다. **다음에**: `조건 ? (참일 때) : (거짓일 때)`를 쓰기 전에 "참이면 무슨 기준?"을 한 번 말해 본다.
- **틀렸던 지점 5**: 빈도가 같은데 빈도끼리 또 뺌. **왜**: 두 번째 기준이 "값"이라는 걸 놓쳤다. **다음에**: 2기준 정렬에서 두 번째 식은 첫 번째 기준을 다시 쓰지 않는다.

## 꿀팁

**`Integer`끼리 `!=`, `==`는 128 이상에서 틀릴 수 있다.** `Integer`는 객체라서 `==`가 값이 아니라 "같은 객체인지"를 비교한다. -128 ~ 127은 자바가 같은 객체를 재사용해서 우연히 맞는다. 이 문제는 빈도가 최대 100이라 통과했지만, 범위가 크면 `map.get(a).equals(map.get(b))`를 쓰거나 `int`로 꺼내서 비교한다.

## AI라면 어떻게 풀었을까

**같은 두 기준을 `? :` 대신 `Comparator.comparing` + `thenComparing`으로 이어 쓴다.** "1기준 → 같으면 2기준"이 코드 모양 그대로 읽혀서, 오늘 헷갈렸던 `?`와 `:` 자리 문제가 사라진다. 값 범위가 -100 ~ 100으로 작아서 빈도는 `HashMap` 대신 크기 201짜리 배열에 센다(값 + 100을 칸 번호로 씀).

```java
public int[] frequencySortAlt(int[] nums) {
    int[] freq = new int[201];                     // -100 ~ 100 → 0 ~ 200
    for (int x : nums) {
        freq[x + 100]++;
    }
    Integer[] arr = new Integer[nums.length];
    for (int i = 0; i < nums.length; i++) {
        arr[i] = nums[i];
    }
    Arrays.sort(arr, Comparator.comparing((Integer x) -> freq[x + 100])   // 빈도 오름차순
            .thenComparing(Comparator.reverseOrder()));                   // 같으면 값 내림차순
    for (int i = 0; i < nums.length; i++) {
        nums[i] = arr[i];
    }
    return nums;
}
```

트레이드오프: 기준이 늘어나도 `.thenComparing(...)`만 이어 붙이면 되고, `Integer` `!=` 함정도 없다. 대신 `comparing`에 넘기는 람다는 "비교식"이 아니라 "**정렬 기준이 될 값 하나**"를 돌려준다. `(a, b) -> ...` 람다와 반환하는 것이 다르니 섞어 쓰지 않도록 주의한다. 배열로 빈도를 세는 건 값 범위가 작을 때만 쓸 수 있다.
