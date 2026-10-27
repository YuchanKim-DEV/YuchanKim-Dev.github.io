---
title: "[프로그래머스-Lv1] 정수 내림차순으로 배치하기 - char는 숫자가 아니다"
date: 2026-10-27 16:26:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, char, long]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/12933
- 파일 경로: `src/main/java/coding_test/정렬/Lv1_정수내림차순으로배치하기.java`
- 난이도: Lv.1

## 문제 설명

정수 `n`의 각 자릿수를 **큰 것부터** 다시 늘어놓은 수를 반환한다. `n`은 최대 80억이라 `int`가 아니라 `long`이다.

```
118372 → 873211
```

원래는 Lv2 테이블 해시 함수를 풀 차례였다. 하지만 아직 그 레벨은 무리고 시간도 적어서, 어제 연습한 문자열 다루기를 이어 가는 짧은 문제로 바꿨다.

## 시행착오

### 1차: `n.length()`

`long`은 기본 타입이라 메서드가 없다. `String.valueOf(n)`으로 문자열로 바꾼 다음 `.length()`를 쓴다.

### 2차: `Character[] temp = ....toCharArray()`

`toCharArray()`는 `char[]`(기본 타입 배열)를 돌려준다. `Character[]`에 바로 담을 수 없다. `int[]`를 `Integer[]`에 못 담는 것과 같은 이유다.

여기서 [최솟값 만들기]({% post_url 2026-10-26-make-minimum-programmers-lv2 %})에서 배운 방법을 썼다. **오름차순으로 정렬하고 뒤에서부터 읽는다.** 그러면 `char[]` 그대로 `Arrays.sort`를 쓸 수 있다.

### 3차: 출력이 `5445706`

```java
for (int i = temp.length - 1; i >= 0; i--) {
    answer += num * temp[i];
    num *= 10;
}
```

틀린 곳이 두 개였다.

1. `temp[i]`는 숫자 1이 아니라 **문자 `'1'`**이다. 계산에 쓰면 문자 코드값(`'1'` = 49)이 들어간다. `n = 1`을 넣으면 49가 나왔다.
2. `num = 1`(일의 자리)부터 곱하면서 뒤(가장 큰 수)부터 읽었다. 그래서 **가장 큰 수가 일의 자리**로 갔다.

### 4차: 통과

`temp[i] - '0'`으로 숫자로 바꿨다. 그리고 `i`를 0부터 돌려서, 작은 수가 일의 자리로 가고 큰 수가 앞자리로 가게 했다.

## 최종 코드

```java
public long solution(long n) {
    char[] temp = String.valueOf(n).toCharArray();
    Arrays.sort(temp);                          // 오름차순

    long answer = 0;
    long num = 1;                               // 자릿값: 1, 10, 100, ...
    for (int i = 0; i < temp.length; i++) {
        answer += num * (temp[i] - '0');        // 작은 수부터 낮은 자리에
        num *= 10;
    }
    return answer;
}
```

## 추적표: `118372`

정렬 후 `['1','1','2','3','7','8']`

| i | temp[i] - '0' | num | answer |
|---|---|---|---|
| 0 | 1 | 1 | 1 |
| 1 | 1 | 10 | 11 |
| 2 | 2 | 100 | 211 |
| 3 | 3 | 1,000 | 3,211 |
| 4 | 7 | 10,000 | 73,211 |
| 5 | 8 | 100,000 | 873,211 |

## 오늘 배운 내용

**`char` 숫자는 `- '0'`을 해야 진짜 숫자다.** `'8'`은 코드값 56이고, `'0'`(48)을 빼야 8이 된다. 반대로 숫자 하나를 `char`로 바꿀 때는 `(char) ('0' + d)`다.

**자릿값을 곱해 숫자를 만들 때는 "일의 자리에 무엇이 오는가"부터 정한다.** `num = 1`부터 시작하면 처음 더하는 수가 일의 자리가 된다.

## 오답노트

- **틀렸던 지점 1**: `num * temp[i]`. **왜**: `char`를 숫자처럼 썼다. **다음에**: `char`로 계산하는 줄에는 `- '0'`이 있는지 본다. `n = 1`처럼 한 자리 입력을 넣어 보면 바로 드러난다.
- **틀렸던 지점 2**: 큰 수부터 읽으면서 일의 자리부터 채움. **왜**: "뒤에서부터 읽는다"는 생각만 하고 자릿값 방향은 따로 확인하지 않았다. **다음에**: 루프 첫 바퀴에서 어떤 수가 어느 자리로 가는지 한 줄만 손으로 따라가 본다.

## AI라면 어떻게 풀었을까

정렬까지는 같다. 숫자로 되돌릴 때 자릿값을 직접 곱하지 않고 **뒤집은 문자열을 `Long.parseLong`으로 한 번에 바꾼다.**

```java
public long solutionAlt(long n) {
    char[] digits = String.valueOf(n).toCharArray();
    Arrays.sort(digits);                                                          // 오름차순
    String desc = new StringBuilder(new String(digits)).reverse().toString();   // 뒤집기 = 내림차순
    return Long.parseLong(desc);
}
```

트레이드오프: 자릿값과 `- '0'` 실수가 생길 여지가 없다. 대신 `StringBuilder`, `new String(char[])`, `parseLong` 세 가지를 알아야 한다. 본인 풀이는 숫자를 직접 만들어서 자릿값 감각을 익히기에 좋다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
