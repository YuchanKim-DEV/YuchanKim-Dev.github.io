---
title: "[프로그래머스-Lv2] 전화번호 목록 - 앞부분을 잘라서 HashSet에 물어본다"
date: 2026-10-20 11:03:00 +0900
categories: [알고리즘 연습, 해시]
tags: [java, programmers, hash, hashset, prefix]
---

> 해시 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-20-hash-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/42577
- 파일 경로: `src/main/java/coding_test/해시/Lv2_전화번호목록.java`
- 난이도: Lv.2

## 문제 설명

전화번호부에서 어떤 번호가 **다른 번호의 접두어(앞부분)**이면 `false`, 아니면 `true`를 반환한다. 번호는 최대 **100만 개**다.

```
["119", "97674223", "1195524421"]   → false  ("119"가 "1195524421"의 앞부분)
["123", "456", "789"]               → true
["12", "123", "1235", "567", "88"]  → false
```

## 시행착오

해시 카테고리의 마지막 문제였다. 100만 개라 모든 쌍을 비교하는 이중 `for`문은 안 되고, **각 번호의 앞부분을 잘라서 `HashSet`에 있는지 물어보는** 방향을 받고 시작했다.

### 0차: "contains 하면 자기 자신이 걸리지 않나?"

맞는 걱정이었다. 그래서 앞부분을 자를 때 **자기 길이보다 짧은 데까지만** 자른다(`i < 번호.length()`). `"1195524421"`이면 `"1"`, `"11"`, `"119"`(여기서 걸림), … `"119552442"`까지만 확인하고, 자기 자신 전체는 확인하지 않는다.

### 1차: `s.contains(s)` → 전부 false

```java
for (String s : set) {
    if (s.contains(s)) {
        return false;
    }
}
```

이 `contains`는 `HashSet`의 것이 아니라 **문자열 `s`의 `contains`**(부분 문자열 포함 여부)였다. "문자열 `s` 안에 `s`가 들어 있나?"라서 항상 `true`였고, 그래서 전부 `false`가 나왔다. 물어봐야 할 대상은 **`set`**, 물어볼 값은 **`s`의 앞부분**이었다.

## 최종 코드

```java
public boolean solution(String[] phone_book) {
    boolean answer = true;
    Set<String> set = new HashSet<>();
    for (int i = 0; i < phone_book.length; i++) {
        set.add(phone_book[i]);                                  // 모든 번호를 넣는다
    }

    for (int i = 0; i < phone_book.length; i++) {
        for (int j = 0; j < phone_book[i].length(); j++) {       // 자기 자신은 제외
            if (set.contains(phone_book[i].substring(0, j))) {   // 앞부분이 번호부에 있나?
                return false;
            }
        }
    }
    return answer;
}
```

9개 테스트가 통과했고, 30만 개는 446ms, 100만 개(14자리)는 305ms였다. 랜덤 30,000건에서도 정답과 일치했다.

## 추적표: `"1195524421"`의 앞부분

| j | `substring(0, j)` | set에 있나? |
|---|---|---|
| 0 | `""` | 없음 |
| 1 | `"1"` | 없음 |
| 2 | `"11"` | 없음 |
| 3 | `"119"` | **있음 → false** |

## 오늘 배운 내용

**"다른 것과 겹치나?"를 모든 쌍으로 비교하면 `O(n²)`이지만, 한쪽을 `HashSet`에 넣으면 `O(n × 길이)`다.** 이 문제는 번호 길이가 최대 20이라, 100만 개여도 앞부분 확인은 최대 2,000만 번이다. 이중 `for`문(1조 번)과 비교가 안 된다.

**`contains`는 누가 부르느냐에 따라 뜻이 다르다.** `set.contains(x)`는 "집합에 `x`가 있나", `str.contains(x)`는 "문자열 안에 `x`가 부분으로 들어 있나"다.

## 오답노트

- **틀렸던 지점 1**: `s.contains(s)`. **왜**: `contains`라는 이름이 같아서 `HashSet`의 메서드를 부른다고 착각했다. **다음에**: 메서드 앞의 **변수 타입**(`Set`인지 `String`인지)을 먼저 본다.
- **확인할 점**: `j`가 0부터라 `substring(0, 0) = ""`도 확인한다. 빈 번호는 없어서 결과는 같지만, 1부터 시작하면 쓸데없는 확인이 하나 줄어든다.
- **좋았던 점**: "자기 자신이 걸리지 않나?"를 코드 전에 스스로 물었다. 경계를 먼저 따져 본 것이다.

## AI라면 어떻게 풀었을까

방향을 뒤집는다. **모든 번호의 "앞부분(자기 자신 제외)"을 `prefixes`에 넣어 두고**, 어떤 번호가 통째로 `prefixes`에 있으면 그 번호는 누군가의 접두어다. 같은 해시 확인이지만 "무엇을 넣고 무엇을 찾나"가 반대다.

```java
public boolean solutionAlt(String[] phone_book) {
    Set<String> prefixes = new HashSet<>();
    for (String p : phone_book) {
        for (int i = 1; i < p.length(); i++) {
            prefixes.add(p.substring(0, i));        // 자기 자신은 빼고 앞부분만
        }
    }
    for (String p : phone_book) {
        if (prefixes.contains(p)) {
            return false;                           // p가 누군가의 앞부분
        }
    }
    return true;
}
```

트레이드오프: "누가 누구의 앞부분인가"의 방향이 코드에 그대로 드러난다. 대신 앞부분을 전부 저장하니 메모리를 더 쓴다(번호마다 최대 19개). 시간 복잡도는 같다. 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
