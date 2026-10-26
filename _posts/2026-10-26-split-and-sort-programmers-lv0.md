---
title: "[프로그래머스-Lv0] 문자열 잘라서 정렬하기 - split이 남기는 빈 문자열"
date: 2026-10-26 13:04:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, string, split, list]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/181866
- 파일 경로: `src/main/java/coding_test/정렬/Lv0_문자열잘라서정렬하기.java`
- 난이도: Lv.0

## 문제 설명

문자열을 `"x"` 기준으로 잘라 사전순으로 정렬해 반환한다. **빈 문자열은 넣지 않는다.**

```
"axbxcxdx"       → ["a", "b", "c", "d"]
"dxccxbbbxaaaa"  → ["aaaa", "bbb", "cc", "d"]
```

바로 전에 푼 [튜플]({% post_url 2026-10-26-tuple-programmers-lv2 %})에서 `split` 파싱이 약했다. 그래서 쉬운 문제로 `split`을 다시 손에 익히려고 골랐다.

## 시행착오

### 1차: `answer.split(" ")` — 배열에 split

```java
String[] answer = myString.split("x");
answer = answer.split(" ");
```

빈 문자열을 없애려고 한 번 더 쪼개려 했다. 그런데 `split`은 `String`의 메서드라서 `String[]`에는 쓸 수 없다.

먼저 `split`이 실제로 무엇을 돌려주는지 확인했다.

```java
"xxaxx".split("x")   →  ["", "", "a"]
```

`x`가 연달아 있거나 맨 앞에 있으면 그 사이에 빈 문자열 `""`이 생긴다. 맨 뒤쪽 빈 문자열은 `split`이 알아서 지운다.

### 2차: 통과

배열은 크기를 바꿀 수 없다. 그래서 빈 문자열이 아닌 것만 `List`에 담고, `toArray`로 배열로 바꿨다.

## 최종 코드

```java
public String[] solution(String myString) {
    String[] temp = myString.split("x");
    List<String> list = new ArrayList<>();
    for (int i = 0; i < temp.length; i++) {
        if (!temp[i].isEmpty()) {                  // 빈 문자열 거르기
            list.add(temp[i]);
        }
    }
    String[] answer = new String[list.size()];
    answer = list.toArray(new String[0]);          // List → String[]
    Arrays.sort(answer, (a, b) -> a.compareTo(b)); // 사전순

    return answer;
}
```

## 추적표: `"bbxaxxab"`

| 단계 | 결과 |
|---|---|
| `split("x")` | `["bb", "a", "", "ab"]` |
| 빈 문자열 거르기 | `["bb", "a", "ab"]` |
| 사전순 정렬 | `["a", "ab", "bb"]` |

`"a"`와 `"ab"`는 앞글자가 같고, 짧은 쪽이 먼저다.

## 오늘 배운 내용

**`split`은 빈 문자열을 남긴다.** 구분자가 맨 앞에 있거나 연달아 있으면 `""`이 생긴다(맨 뒤쪽 빈 문자열만 자동으로 지워진다). 결과에서 빼야 하면 `isEmpty()`로 걸러 `List`에 담는다.

**`List` → 배열은 `list.toArray(new String[0])`.** `new String[0]`은 어떤 타입의 배열로 바꿀지 알려 주는 용도이고, 크기는 자바가 맞춰 준다.

## 오답노트

- **틀렸던 지점**: `answer.split(" ")`. **왜**: `split`이 배열에도 있다고 생각했다. **어떻게 고쳤나**: 빈 문자열을 `isEmpty()`로 걸러 `List`에 담았다. **다음에**: 메서드를 부르기 전에 변수 타입이 `String`인지 `String[]`인지 본다. `split` 결과는 항상 한 번 찍어 보고 `""`이 끼어 있는지 확인한다.

## AI라면 어떻게 풀었을까

흐름은 같고 두 군데만 줄인다.

```java
public String[] solutionAlt(String myString) {
    List<String> list = new ArrayList<>();
    for (String part : myString.split("x")) {
        if (!part.isEmpty()) {
            list.add(part);
        }
    }
    Collections.sort(list);                   // 사전순
    return list.toArray(new String[0]);
}
```

1. `String`은 기본 정렬이 이미 사전순이라 `(a, b) -> a.compareTo(b)`가 필요 없다.
2. `toArray`가 새 배열을 만들어 주므로 `new String[list.size()]`를 미리 만들 필요가 없다. 본인 코드에서는 만든 배열이 바로 버려진다.

트레이드오프는 없다. 결과는 같고 코드만 짧아진다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
