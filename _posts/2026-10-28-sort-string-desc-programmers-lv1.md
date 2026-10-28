---
title: "[프로그래머스-Lv1] 문자열 내림차순으로 배치하기 - 글자는 글자 그대로 붙인다"
date: 2026-10-28 09:08:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, char, string]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/12917
- 파일 경로: `src/main/java/coding_test/정렬/Lv1_문자열내림차순으로배치하기.java`
- 난이도: Lv.1

## 문제 설명

문자열의 글자를 **큰 것부터** 정렬해 반환한다. 영문 대소문자만 있고, **대문자는 소문자보다 작다.**

```
"Zbcdefg" → "gfedcbZ"
```

바로 전에 푼 [정수 내림차순으로 배치하기]({% post_url 2026-10-27-sort-digits-desc-programmers-lv1 %})와 같은 모양이다. 이번에는 "첫 15분은 힌트 없이" 시도했다.

## 시행착오

### 1차: `answer += temp[i] - '0'`, 앞에서부터

```java
char[] temp = String.valueOf(s).toCharArray();
Arrays.sort(temp);
for (int i = 0; i < temp.length; i++) {
    answer += temp[i] - '0';
}
```

`toCharArray` → `Arrays.sort`까지는 혼자 바로 짰다. 틀린 곳은 두 개였다.

1. **`- '0'`을 그대로 가져왔다.** 앞 문제에서 `- '0'`을 쓴 건 숫자 문자를 계산용 숫자로 바꾸기 위해서였다. 이번에는 글자를 그대로 붙여야 한다. `'b' - '0'`은 50이라 `"50"`이 붙는다.
2. **앞에서부터 붙였다.** `Arrays.sort`는 오름차순이라 작은 글자부터 붙는다.

### 2차: 통과

`- '0'`을 지우고, `i`를 뒤에서부터 돌려 큰 글자부터 붙였다.

## 최종 코드

```java
public String solution(String s) {
    char[] temp = String.valueOf(s).toCharArray();
    Arrays.sort(temp);                              // 오름차순 (대문자가 앞)
    String answer = "";

    for (int i = temp.length - 1; i >= 0; i--) {   // 뒤에서부터 = 큰 글자부터
        answer += temp[i];
    }
    return answer;
}
```

`s`는 이미 `String`이라 `String.valueOf(s)` 없이 `s.toCharArray()`만 써도 된다.

## 추적표: `"Zbcdefg"`

| 단계 | 결과 |
|---|---|
| toCharArray | `['Z','b','c','d','e','f','g']` |
| 오름차순 정렬 | `['Z','b','c','d','e','f','g']` (`'Z'` = 90 < `'b'` = 98) |
| 뒤에서부터 붙이기 | `"gfedcbZ"` |

## 오늘 배운 내용

**`char`의 기본 정렬은 문자 코드순이다.** 대문자는 65 ~ 90, 소문자는 97 ~ 122라서 "대문자가 소문자보다 작다"는 조건이 따로 처리하지 않아도 저절로 맞는다.

**`- '0'`은 "숫자 문자 → 계산용 숫자"일 때만 쓴다.** 글자를 문자열에 그대로 이어 붙일 때는 `char`를 그대로 `+=` 한다.

## 오답노트

- **틀렸던 지점**: `answer += temp[i] - '0'`. **왜**: 직전 문제 코드를 모양만 보고 따라 썼다. **다음에**: `- '0'`을 쓰기 전에 "이 글자를 숫자로 계산해야 하나, 글자 그대로 써야 하나"를 먼저 묻는다.

## AI라면 어떻게 풀었을까

흐름은 같고, 붙이는 부분만 바꾼다. `String`에 `+=`로 한 글자씩 붙이면 붙일 때마다 새 문자열이 만들어진다. 그래서 정렬된 `char[]`를 한 번에 문자열로 만들고 `StringBuilder.reverse()`로 뒤집는다.

```java
public String solutionAlt(String s) {
    char[] chars = s.toCharArray();
    Arrays.sort(chars);                                                // 오름차순 (대문자가 앞)
    return new StringBuilder(new String(chars)).reverse().toString(); // 뒤집기 = 내림차순
}
```

트레이드오프: 길이가 길어질수록 `+=` 반복보다 빠르다. 이 문제는 길이 제한이 작아서 차이가 없다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
