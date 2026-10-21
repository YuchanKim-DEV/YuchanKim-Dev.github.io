---
title: "[프로그래머스-Lv1] 문자열 내 마음대로 정렬하기 - Comparator로 기준 두 개를 잇는다"
date: 2026-10-21 11:27:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, comparator]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/12915
- 파일 경로: `src/main/java/coding_test/정렬/Lv1_문자열내마음대로정렬하기.java`
- 난이도: Lv.1

## 문제 설명

문자열 배열 `strings`를 **각 문자열의 n번째 글자(0부터)** 기준으로 오름차순 정렬한다. n번째 글자가 같으면 **사전순**으로 앞선 문자열이 앞에 온다.

```
["sun","bed","car"], n = 1    → ["car","bed","sun"]   (u, e, a 기준)
["abce","abcd","cdx"], n = 2  → ["abcd","abce","cdx"] (c, c, x → c가 같은 둘은 사전순)
```

## 시행착오

**틀린 곳 없이 한 번에 통과했다.** 정렬 카테고리에서 처음으로 `Comparator`를 쓰는 문제였는데, 정렬 총정리 글에 정리해 둔 "기준이 같으면 다음 기준으로(`thenComparing`)" 모양을 그대로 응용했다. 힌트는 받지 않았다.

## 최종 코드

```java
public String[] solution(String[] strings, int n) {
    Arrays.sort(strings, Comparator.comparing((String s) -> s.charAt(n))   // 1순위: n번째 글자
            .thenComparing(Comparator.naturalOrder()));                    // 2순위: 사전순
    return strings;
}
```

7개 테스트가 통과했고, 랜덤 30,000건에서도 정답과 일치했다.

## 추적표: `["abce","abcd","cdx"]`, n = 2

| 문자열 | 2번째 글자 | 1순위 결과 | 같은 글자끼리 2순위(사전순) |
|---|---|---|---|
| abce | c | c 그룹 | abcd 다음 |
| abcd | c | c 그룹 | **맨 앞** |
| cdx | x | x 그룹(뒤) | - |
| 결과 | | | **[abcd, abce, cdx]** |

## 오늘 배운 내용

**정렬 기준이 여러 개면 `Comparator.comparing(1순위).thenComparing(2순위)`로 잇는다.** 1순위가 같을 때만 2순위를 본다. 람다 `(String s) -> s.charAt(n)`처럼 **타입을 적어 주는** 이유는, 뒤에 `.thenComparing`을 붙이면 Java가 `s`의 타입을 스스로 추론하지 못하기 때문이다.

## 오답노트

- **틀린 곳은 없었다.**
- **좋았던 점**: 직전에 정리한 총정리 글의 도구를 바로 꺼내 썼다. "어떤 도구가 있는지 아는 것"이 이 유형의 절반이다.
- **알아 둘 점**: `Arrays.sort`는 반환값 없이 **배열 자체를 바꾼다.** 그래서 정렬한 뒤 `strings`를 그대로 반환하면 된다.

## AI라면 어떻게 풀었을까

기준 두 개를 한 `Comparator`에 넣는 대신 **"안정 정렬"을 이용해 정렬을 두 번** 한다. 객체 배열의 `Arrays.sort`는 안정 정렬이라 기준이 같은 원소들은 정렬 전 순서를 그대로 유지한다. 그래서 먼저 사전순으로 정렬해 두고, 그다음 n번째 글자만으로 정렬하면 글자가 같은 문자열끼리는 사전순이 그대로 남는다.

```java
public String[] solutionAlt(String[] strings, int n) {
    Arrays.sort(strings);                                         // 1) 사전순 (보조 기준)
    Arrays.sort(strings, Comparator.comparing(s -> s.charAt(n))); // 2) n번째 글자 (주 기준)
    return strings;
}
```

트레이드오프: `thenComparing` 없이 쉬운 정렬 두 번으로 끝나고, 람다 타입 표기도 필요 없다. 대신 **"보조 기준을 먼저, 주 기준을 나중에"**라는 순서와 안정 정렬이라는 성질을 알아야 한다. 기본형 배열(`int[]`)의 정렬은 안정 정렬이 보장되지 않으니 이 방법은 객체 배열에서만 쓴다. 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
