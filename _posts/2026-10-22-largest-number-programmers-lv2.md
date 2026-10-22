---
title: "[프로그래머스-Lv2] 가장 큰 수 - 크기가 아니라 '붙인 결과'로 정렬하기"
date: 2026-10-22 16:08:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, comparator]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/42746
- 파일 경로: `src/main/java/coding_test/정렬/Lv2_가장큰수.java`
- 난이도: Lv.2

## 문제 설명

0 이상의 정수 배열 `numbers`의 **순서를 바꿔 이어 붙였을 때 만들 수 있는 가장 큰 수**를 문자열로 반환한다. 원소는 최대 100,000개, 각 원소는 0 ~ 1,000이다.

```
[6, 10, 2]         → "6210"
[3, 30, 34, 5, 9]  → "9534330"
```

## 시행착오

처음 보는 패턴이라 감이 오지 않았고, 아이디어와 수도코드를 받은 뒤에도 문법에서 여러 번 막혔다. 결국 마지막 비교식 한 줄은 직접 제시받았다.

### 0차: 옆 칸끼리 비교하려다 멈춤

```java
for (int i = 0; i < numbers.length - 1; i++) {
    String a = Integer.toString(numbers[i]);
    String b = Integer.toString(numbers[i + 1]);
    Arrays.sort()
}
```

"두 수를 비교한다"는 감은 있었는데, 옆 칸만 한 번씩 보면 전체 순서가 정해지지 않는다. 그리고 `Arrays.sort`를 반복문 안에 넣었다. **`a`, `b`는 내가 만드는 게 아니라 `Arrays.sort`가 두 개씩 꺼내 와서 규칙에 물어보는 것**이다. 정렬은 반복문 밖에서 한 번만 부르고, 우리는 규칙만 넘긴다.

전체 흐름을 세 덩어리로 나누고 나서야 구조가 보였다.

```
[1] [6, 10, 2]        → ["6", "10", "2"]   문자열로 바꿈
[2] ["6", "10", "2"]  → ["6", "2", "10"]   붙였을 때 가장 크게 되는 순서로 정렬
[3] ["6", "2", "10"]  → "6210"             순서대로 이어 붙이면 답
```

### 1차: `NullPointerException ... "a[runHi]" is null`

```java
for (int i = 0; i < numbers.length - 1; i++) {   // 마지막 칸을 안 채움
    strArr[i] = Integer.toString(numbers[i]);
}
```

0차에서 `numbers[i + 1]` 때문에 붙였던 `- 1`이 남아 있었다. `i + 1`이 없어졌는데 범위만 그대로라 마지막 칸이 `null`이었고, 정렬하다 터졌다.

### 2차: `Arrays.reversed()` — 컴파일 에러 + 규칙도 틀림

```java
Arrays.reversed().sort(strArr, Comparator.comparing((String s) -> s.charAt(0))
        .thenComparing(Comparator.naturalOrder()));
```

- `reversed()`는 `Arrays`가 아니라 **`Comparator`의 메서드**다. `규칙.reversed()`로 쓴다.
- 고쳐도 규칙 자체가 틀렸다. "첫 글자 → 사전순"을 뒤집으면 `3`과 `30` 중 `"30"`이 앞에 와서 `"303"`이 된다. `"330"`이 더 크다. **글자를 따로따로 보는 규칙으로는 못 푼다.**

### 3차: `(a + b) - (b + a)` — 문자열은 뺄 수 없음

```java
Arrays.sort(strArr, (a, b) -> (a + b) - (b + a));
// error: bad operand types for binary operator '-'
```

드디어 "붙여서 비교"가 들어갔다. 하지만 `a + b`는 `"6" + "10"` = `"610"`인 **문자열**이고, `-`는 숫자에만 쓸 수 있다. 문자열은 `X.compareTo(Y)`가 `X - Y` 역할을 한다.

### 4·5차: `compareTo`로 바꾸다가 붙인 문자열이 사라짐

```java
Arrays.sort(strArr, (a, b) -> b.compareTo(a) - a.compareTo(b));   // 4차
Arrays.sort(strArr, (a, b) -> b.compareTo(a));                    // 5차
```

```
나온 값: 6210 9534303 0 0 210 12112 40340 999999 10000 1000100101
정답:    6210 9534330 0 0 210 12121 40403 999999 10000 1101001000
```

둘 다 `a`와 `b`를 **그대로** 비교해서 사실상 "사전순 뒤집기"였다. 2차와 같은 이유로 `3`/`30`, `12`/`121`, `40`/`403`에서 틀렸다. `compareTo`는 비교할 두 값(`b + a`, `a + b`)을 한 번에 받는다는 걸 놓쳤다.

### 6차: 통과

```java
Arrays.sort(strArr, (a, b) -> (b + a).compareTo(a + b));
```

이 한 줄은 직접 제시받았다. 바꾸자마자 11개 테스트가 모두 통과했다.

## 최종 코드

```java
public String solution(int[] numbers) {
    String[] strArr = new String[numbers.length];
    for (int i = 0; i < numbers.length; i++) {
        strArr[i] = Integer.toString(numbers[i]);           // [1] 문자열로
    }

    Arrays.sort(strArr, (a, b) -> (b + a).compareTo(a + b)); // [2] 붙인 결과가 큰 쪽이 앞

    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < strArr.length; i++) {
        sb.append(strArr[i]);                                // [3] 이어 붙이기
    }

    if (sb.charAt(0) == '0') {                               // 전부 0이면 "000"이 아니라 "0"
        sb = new StringBuilder();
        sb.append("0");
    }

    return sb.toString();
}
```

랜덤 30,000건에서 아래 AI 풀이와 결과가 같았고, 원소 100,000개 입력은 169ms였다.

## 추적표: 비교 규칙이 내리는 판단

| a | b | a + b | b + a | 결과 |
|---|---|---|---|---|
| `"6"` | `"10"` | `"610"` | `"106"` | 6이 앞 |
| `"3"` | `"30"` | `"330"` | `"303"` | 3이 앞 |
| `"3"` | `"34"` | `"334"` | `"343"` | 34가 앞 |
| `"12"` | `"121"` | `"12121"` | `"12112"` | 12가 앞 |
| `"9"` | `"5"` | `"95"` | `"59"` | 9가 앞 |

`[3, 30, 34, 5, 9]`는 이 규칙으로 `["9", "5", "34", "3", "30"]`이 되고, 붙이면 `"9534330"`이다.

## 오늘 배운 내용

**정렬 기준은 "값의 크기"가 아니라 "문제가 원하는 결과"로 정한다.** 이 문제의 목표는 붙인 결과가 크게 되는 것이므로, 두 원소의 순서도 **실제로 붙여 본 결과**(`a + b` vs `b + a`)로 정한다. `Comparator`에는 "두 개를 받아 누가 앞인지" 알려 주는 규칙만 넣으면 된다. 전체 순서는 `Arrays.sort`가 그 규칙으로 알아서 맞춘다.

## 오답노트

- **틀렸던 지점 1**: 옆 칸 `a`, `b`를 직접 만들고 `Arrays.sort`를 반복문 안에 넣음. **왜**: `Comparator`가 "규칙을 넘겨주는 것"이라는 감이 없었다. **다음에**: 정렬 기준을 바꾸는 문제는 `Arrays.sort(배열, (a, b) -> 규칙)` 한 줄이 반복문 밖에 있다.
- **틀렸던 지점 2**: 범위에 `- 1`이 남음(NPE). **왜**: 앞 코드에서 `i + 1` 때문에 넣은 `- 1`을, `i + 1`을 지운 뒤에도 그대로 뒀다. **다음에**: 반복문 끝값의 `- 1`은 안에서 `i + 1`을 쓸 때만 필요하다.
- **틀렸던 지점 3**: `Arrays.reversed()`. **왜**: "뒤집기" 메서드가 어디 붙는지 몰랐다. **다음에**: `reversed()`는 `Comparator` 뒤에 붙는다(`Comparator.comparing(...).reversed()`).
- **틀렸던 지점 4**: 첫 글자 + 사전순 규칙. **왜**: 어제 문제(n번째 글자 → 사전순)의 모양을 그대로 가져왔다. **다음에**: `3`/`30`처럼 앞부분이 같은 반례를 먼저 손으로 넣어 본다.
- **틀렸던 지점 5**: 문자열에 `-`. **왜**: 숫자 비교 습관(`a - b`). **다음에**: 문자열 비교는 `X.compareTo(Y)`이고, 이게 `X - Y` 자리를 대신한다.
- **틀렸던 지점 6**: `compareTo`로 바꾸며 `a + b`, `b + a`가 사라짐. **왜**: `compareTo`를 `a`, `b`에 각각 붙여야 한다고 생각했다. **다음에**: `X - Y`를 `X.compareTo(Y)`로 바꿀 때 X, Y를 통째로 그대로 옮긴다.

## 꿀팁

**`(b + a).compareTo(a + b)`에서 `b`가 앞에 있으면 "큰 게 앞"(내림차순)이다.** 기본 `a.compareTo(b)`는 오름차순이고, 두 값의 자리를 바꾸면 내림차순이 된다. 이 문제처럼 비교 대상이 `a + b`여도 같은 규칙이다. "b 쪽이 앞이면 내림차순"으로 외워 두면 방향을 매번 돌려 보지 않아도 된다.

## AI라면 어떻게 풀었을까

**같은 "붙여서 비교" 규칙을 문자열 대신 숫자로 비교한다.** `a + b`와 `b + a`는 길이가 같아서 숫자로 바꿔 비교해도 결과가 같다. 원소가 1,000 이하라 붙여도 최대 8자리이고, `long`이면 충분하다. "0" 처리는 붙인 뒤가 아니라 정렬 직후 맨 앞 칸으로 판단한다. 맨 앞이 `"0"`이면 가장 큰 조각이 0이라는 뜻이라 전부 0이다.

```java
public String solutionAlt(int[] numbers) {
    String[] strs = new String[numbers.length];
    for (int i = 0; i < numbers.length; i++) {
        strs[i] = String.valueOf(numbers[i]);
    }
    Arrays.sort(strs, (a, b) -> Long.compare(Long.parseLong(b + a), Long.parseLong(a + b)));
    if (strs[0].equals("0")) {
        return "0";
    }
    return String.join("", strs);
}
```

트레이드오프: 숫자 비교라 오늘 헷갈렸던 `compareTo`보다 "큰 게 앞"이 눈에 잘 들어온다. 대신 비교할 때마다 `parseLong`을 해서 조금 느리고, 원소가 커서 붙인 길이가 18자리를 넘으면 `long`도 넘친다. 그럴 땐 본인 풀이처럼 문자열 `compareTo`가 안전하다.
