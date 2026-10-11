---
title: "[LeetCode-387] 세는 것과 찾는 것은 다른 순서로 읽는다"
date: 2026-10-11 15:20:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, queue, counting, string]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/first-unique-character-in-a-string/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC0387_FirstUniqueCharacterInAString.java`
- 난이도: Easy

## 문제 설명

소문자로만 이루어진 문자열 `s`에서 **한 번만 나오는 문자 중 가장 앞에 있는 것**의 인덱스를 반환한다. 없으면 -1이다.

```
Input: s = "loveleetcode"
Output: 2     // 'l'과 'o'는 두 번씩 나오고, 'v'(인덱스 2)가 처음으로 한 번뿐
```

## 시행착오

"변하는 것과 변하지 않는 것이 뭘까?"라는 질문으로 시작했다. 처음 답은 "알파벳의 위치가 변하고 인덱스는 안 변한다"였는데, 이 문제는 큐 시뮬레이션이 아니라서 문자열이 처음부터 끝까지 그대로다. 실제로 변하는 건 **훑는 동안 쌓여 가는 문자별 등장 횟수**다.

### 1차: 횟수 세기는 바로 됐지만 "찾기"를 알파벳 배열로 돌림

문자를 정수 칸에 대응시키는 `c - 'a'` 공식은 이번에 처음 배웠고, 세는 부분은 바로 적용했다.

```java
charArr[queue.poll() - 'a']++;               // 세기: 정확함

for (int i = 0; i < charArr.length; i++) {   // 찾기: 26칸(알파벳 순서)을 돌림
    if (charArr[i] == 1) {
        char c = charArr[i] + 'a';           // 컴파일 에러: int → char
        break;
    }
}
```

두 가지가 어긋났다.

1. **찾는 반복이 문자열 `s`가 아니라 `charArr`(알파벳 순서)를 돌았다.** 문제는 "문자열에서 가장 앞"을 원하는데, 알파벳 순서로 읽으면 등장 순서가 사라진다.
2. **`char c = charArr[i] + 'a'`는 `int`를 `char`에 담아서 컴파일 에러**다. 게다가 반환해야 하는 건 문자가 아니라 **인덱스**여서 그 줄이 필요 없었다.

## 최종 코드

```java
public int firstUniqChar(String s) {
    int[] charArr = new int[26];
    Queue<Character> queue = new ArrayDeque<>();
    for (int i = 0; i < s.length(); i++) {
        queue.offer(s.charAt(i));
    }

    while (!queue.isEmpty()) {
        charArr[queue.poll() - 'a']++;          // 1회차: 문자별 횟수 세기
    }

    int min = 0;
    boolean updated = false;
    for (int i = 1; i < s.length(); i++) {      // 2회차: 문자열 순서대로 확인
        if (charArr[s.charAt(min) - 'a'] != 1 && charArr[s.charAt(i) - 'a'] == 1) {
            updated = true;
            min = i;
        }
    }

    if (charArr[s.charAt(min) - 'a'] > 1) {
        min = -1;
    }

    return min;
}
```

7개 테스트가 모두 통과했고, 랜덤 30,000건에서도 정답 방식과 같은 결과였다. 다만 `min`을 0으로 시작해 "현재 후보가 유일하지 않을 때만 갱신"하는 구조라 조건이 조금 돌아간다. `updated`는 쓰이지 않는다.

## 추적표: `"loveleetcode"`

| 단계 | 설명 |
|---|---|
| 1회차(세기) | l:2, o:2, v:1, e:4, t:1, c:1, d:1 |
| 2회차(찾기) | 인덱스 0(`l`, 2회) → 건너뜀, 1(`o`, 2회) → 건너뜀, **2(`v`, 1회) → 찾음** |

## 오늘 배운 내용

**횟수는 문자 순서로 세고, 찾는 건 문자열 순서로 한다.** 세는 건 모든 문자를 한 번 보면 되지만 "가장 앞"이라는 정보는 문자열의 읽는 순서에만 들어 있다. 그래서 같은 문자열을 **두 번** 읽는다. 알파벳 26칸으로 정보를 압축하면 순서는 사라진다는 걸 기억해야 한다.

**`c - 'a'`는 소문자를 0~25 칸에 대응시키는 공식**이다. 자바에서 `char`는 숫자로 취급돼 빼기가 된다.

## 오답노트

- **틀렸던 지점 1**: "변하는 것은 알파벳의 위치"라고 생각함. **왜**: 앞의 큐 문제(LC2073, LC1700)에서 줄이 돌아가는 걸 막 풀어서 이 문제에도 이동이 있다고 착각했다. **다음에**: 이 문제에서 실제로 움직이는 게 있는지 먼저 본다.
- **틀렸던 지점 2**: 찾는 반복을 `charArr`로 돌려 알파벳 순서로 읽음. **왜**: 횟수를 모아 둔 배열을 그대로 훑는 게 자연스러워 보였다. **다음에**: "정답이 무엇의 순서로 정해지는가"를 먼저 묻는다. 여기선 문자열의 등장 순서다.
- **틀렸던 지점 3**: `char c = charArr[i] + 'a'`. **왜**: `int + char`의 결과는 `int`다. **다음에**: 반환 타입이 무엇인지(인덱스=`int`)를 먼저 확인하고, 필요 없는 변환은 만들지 않는다.

## 꿀팁

소문자만 나오는 문제는 `Map`보다 `int[26]`이 짧고 빠르다. 공식은 `count[c - 'a']`다. 대문자까지 있으면 `int[128]`(아스키 전체)로 `count[c]`를 바로 쓸 수도 있다.

## AI라면 어떻게 풀었을까

문자열을 두 번 읽지 않고 **한 번만 훑으면서 큐에 "아직 유일할 수 있는 후보의 인덱스"를 보관**한다. 새 문자를 읽을 때마다 횟수를 올리고, 큐 맨 앞의 후보가 이미 두 번 이상 나왔으면 `poll`로 버린다. 큐에는 등장 순서가 그대로 보존되므로, 끝났을 때 맨 앞이 곧 답이다. 이때 큐에는 문자가 아니라 **인덱스**를 넣는다. 반환값이 인덱스이기 때문이다.

```java
public int firstUniqCharAlt(String s) {
    int[] count = new int[26];
    Queue<Integer> queue = new ArrayDeque<>();   // 후보 인덱스 (등장 순서)
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        queue.offer(i);
        while (!queue.isEmpty() && count[s.charAt(queue.peek()) - 'a'] > 1) {
            queue.poll();                        // 이미 중복된 후보는 버림
        }
    }
    return queue.isEmpty() ? -1 : queue.peek();
}
```

트레이드오프: 문자열을 한 번만 읽는 장점이 있고, **스트림으로 문자가 들어오는 상황**(끝을 모를 때)에서도 매번 답을 알 수 있다. 대신 이 문제처럼 문자열이 이미 주어진 경우엔 두 번 읽는 방식과 시간이 비슷하고, `while`의 "맨 앞만 검사하고 버린다"는 논리가 처음엔 더 이해하기 어렵다. 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
