---
title: "[코드트리] 팰린드롬 수 찾기 - reverse()는 같은 객체를 뒤집는다"
date: 2026-11-18 15:30:00 +0900
categories: [알고리즘 연습, 완전탐색]
tags: [java, codetree, brute-force, palindrome, stringbuilder]
---

- 문제 링크: 코드트리 팰린드롬 수 찾기
- 파일 경로: `src/main/java/coding_test/완전탐색/CT_팰린드롬수찾기.java`
- 난이도: Medium, 50XP (정답률 88%)
- 제한: 10 ≤ X ≤ Y ≤ 1,000,000 (2.5초, 512MiB)

## 문제 설명

X 이상 Y 이하의 수 중 **팰린드롬**(거꾸로 읽어도 같은 수)의 개수를 센다.

```
98 132   → 5   (99, 101, 111, 121, 131)
```

## 시행착오

X부터 Y까지 하나씩 확인하는 완전탐색이다. 최대 100만 번이라 충분하다. 막힌 곳은 알고리즘이 아니라 **Java 문자열 API**였다.

### 1. `reverse()`는 새 객체를 만들지 않는다

```java
StringBuilder sb = new StringBuilder(String.valueOf(i));
StringBuilder reversed = sb.reverse();   // sb 자신을 뒤집고, 같은 객체를 돌려줌
if (sb.equals(reversed)) count++;        // 항상 같은 객체 → 전부 팰린드롬으로 셈
```

`sb`와 `reversed`는 이름만 다른 **같은 객체**다. 비교하면 항상 같다고 나온다.

### 2. `StringBuilder.equals()`는 내용을 비교하지 않는다

`StringBuilder`는 `equals()`를 "같은 객체인가"로만 본다. 내용 비교는 `String`의 `equals()`가 한다. 그래서 **뒤집기 전에 원본을 `String`으로 따로 저장**하고, 뒤집은 결과도 `toString()`으로 `String`을 만들어 비교한다.

### 3. `new StringBuilder(int)`는 내용이 아니라 용량이다

`new StringBuilder(121)`은 `"121"`이 아니라 용량 121인 빈 객체를 만든다. 숫자를 넣으려면 `String.valueOf(i)`를 거친다. (이 부분은 질문으로 먼저 확인하고 썼다.)

## 최종 코드

```java
int count = 0;
for (int i = x; i <= y; i++) {
    StringBuilder sb = new StringBuilder(String.valueOf(i));
    String sb_temp = sb.toString();            // 뒤집기 전에 원본을 String으로 보관
    StringBuilder reversed = sb.reverse();

    if (sb_temp.equals(reversed.toString())) {
        count++;
    }
}
return count;
```

## 추적표: 121과 123

| i | 원본(String) | reverse 후 | 같은가? | count |
|---|---|---|---|---|
| 121 | "121" | "121" | ✅ | +1 |
| 123 | "123" | "321" | ❌ | 그대로 |

## 오늘 배운 내용

- `StringBuilder.reverse()`는 **원본을 바꾸고 같은 객체를 돌려준다.** 원본이 필요하면 뒤집기 전에 `toString()`으로 복사해 둔다.
- 내용 비교는 `String.equals()`. `StringBuilder.equals()`와 `==`는 내용을 비교하지 않는다.
- `new StringBuilder(정수)`는 용량이다. 숫자를 문자열로 바꿔서 넣는다.

## 오답노트

- 처음에는 `reversed`가 새 객체라고 가정해서 비교가 항상 참이 되었다. 메서드가 **새 객체를 돌려주는지, 자신을 바꾸는지**는 처음 쓰는 API라면 문서나 작은 출력으로 확인한다.

## AI라면 어떻게 풀었을까

문자열 없이 **숫자만으로 뒤집는다.**

```java
int rev = 0, t = i;
while (t > 0) {
    rev = rev * 10 + t % 10;   // 일의 자리를 떼어 뒤에 붙인다
    t /= 10;
}
if (rev == i) count++;
```

객체를 만들지 않아 빠르고 `equals`/`==` 혼동도 없다. 이 문제는 100만 번이라 둘 다 통과한다.
