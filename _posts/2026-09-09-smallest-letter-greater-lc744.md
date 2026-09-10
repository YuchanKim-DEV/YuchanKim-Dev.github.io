---
title: "답이 없는 경우를 배열 밖 인덱스로 표현하는 법 - 오늘 가장 오래 걸린 문제"
date: 2026-09-09 21:00:00 +0900
categories: [코딩테스트, 이진탐색]
tags: [java, leetcode, binary-search]
---

- 문제 링크: https://leetcode.com/problems/find-smallest-letter-greater-than-target/
- 파일 경로: `src/main/java/coding_test/이진탐색/LC0744_FindSmallestLetterGreaterThanTarget.java`
- 난이도: Easy

## 문제 설명

오름차순 정렬된 문자 배열 `letters`와 문자 `target`이 주어진다. `target`보다 **사전순으로 더 큰 문자 중 가장 작은 것**을 반환한다. 그런 문자가 없으면(= target이 배열의 모든 문자보다 크거나 같으면) 순환해서 배열의 첫 문자를 반환한다.

```
letters = [c,f,j], target = 'a'  -> 'c'
letters = [c,f,j], target = 'c'  -> 'f'
letters = [x,x,y,y], target = 'z' -> 'x' (순환)
```

## 시행착오 — 오늘 가장 오래 걸린 문제

### 1차 시도 — `==`일 때 mid+1 리턴

```java
if (letters[mid] == target) {
    return letters[mid + 1];
}
```

이 문제는 "일치하면 그 자리가 답"이 아니라 "더 큰 값 중 최소"를 찾는 문제라, `==`이어도 아직 답이 아니다. `mid+1`이 항상 맞는 답이라는 보장도 없어서 틀린 접근이었다.

### 2차 시도 — 빈 분기로 인한 무한루프 위험

`else if (letters[mid] > target)` 분기 안을 비워둔 채로 두어서 `left`/`right` 갱신이 안 되는 상태였다. 278에서 배운 패턴("즉시 리턴 조건이 없을 때는 `right = mid`로 후보를 남기고 `left = mid+1`로 확실히 제외")을 그대로 응용해서 구조를 다시 짰다:

```java
if (letters[mid] > target) {
    right = mid;
} else {
    left = mid + 1;
}
```

이 부분은 278의 패턴을 스스로 응용해서 도출했다.

### 3차 시도 — 리턴문 하드코딩 버그

```java
return letters[0];   // 항상 0번째만 반환하는 실수
```

루프가 끝난 뒤 `left`가 답이어야 하는데, 무조건 `letters[0]`을 반환하도록 짜놔서 `target='c'`인 경우처럼 `left`가 다른 값이 된 케이스가 전부 틀렸다. `return letters[0]`을 `return letters[left]`로 고쳐서 대부분의 케이스는 해결.

### 4차 시도 — "답이 없는 경우"(순환)를 표현 못 함 → 결국 힌트로 해결

`right = letters.length - 1`로 잡으면 `left`가 절대 배열 마지막 인덱스를 넘을 수 없다. 그런데 "target보다 큰 문자가 배열에 하나도 없다"는 상황은 마지막 인덱스보다 한 칸 더 밖을 가리켜야 표현할 수 있는 상태였다. 이 부분은 스스로 도출하지 못하고 힌트를 받았다:

- `right`를 `letters.length - 1`이 아니라 **`letters.length`**로 잡아서, `left`가 배열 밖(=길이 자체)까지 갈 수 있게 함
- 리턴할 때 `letters[left % letters.length]`로 나머지 연산을 써서, `left`가 배열 밖으로 나갔으면 자동으로 `0`번째로 순환되게 함

## 최종 코드

```java
public char nextGreatestLetter(char[] letters, char target) {
    int left = 0;
    int right = letters.length;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (letters[mid] > target) {
            right = mid;
        } else if (letters[mid] <= target) {
            left = mid + 1;
        }
    }

    return letters[left % letters.length];
}
```

## 변수 추적표 (letters = [x,x,y,y], target = 'z' — 답이 없어서 순환하는 케이스)

| 반복 | left | right | mid | letters[mid] | 비교 | 갱신 |
|---|---|---|---|---|---|---|
| 1 | 0 | 4 | 0+(4-0)/2=**2** | 'y' | 'y' <= 'z' | left = 3 |
| 2 | 3 | 4 | 3+(4-3)/2=**3** | 'y' | 'y' <= 'z' | left = 4 |
| — | 4 | 4 | `left(4) < right(4)`? 거짓 → 종료 | | | |

루프 종료 시 `left = 4` (배열 길이와 같음, 즉 "밖"). `letters[4 % 4]` = `letters[0]` = `'x'` (정답)

`right`를 `letters.length - 1`(=3)로 잡았다면 `left`는 절대 4까지 못 가고 최대 3에서 멈췄을 것이고, 그러면 이 "답 없음" 상태 자체를 표현할 수 없었다.

## 한 줄 오답노트

> "찾는 게 아예 없을 수도 있다"는 상태는 배열 인덱스 범위 밖(`length`)으로 표현하고, 최종 리턴에서 `% length`로 안전하게 순환시킨다. `right`를 `length-1`로 좁게 잡으면 이 상태 자체를 나타낼 방법이 없어진다.
