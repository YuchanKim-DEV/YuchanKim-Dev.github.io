---
title: "[LeetCode-744] \"답이 없는 경우\"를 배열 밖 인덱스로 표현하는 법 - right=length와 % 순환 처리"
date: 2026-09-09 21:00:00 +0900
categories: [알고리즘 연습, 이진탐색]
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

## 오늘 배운 내용

이 문제에서 가장 중요한 내용은, **"찾는 게 아예 없을 수도 있다"는 상태를 배열 인덱스 범위 밖(`length`)으로 표현**한다는 것이다. `right`를 배열 길이 자체로 넉넉히 잡아두면 `left`가 배열 밖까지 갈 수 있고, 최종 리턴에서 `% length`로 안전하게 순환시키면 "답 없음"과 "배열 첫 원소로 순환"이라는 문제의 요구사항을 동시에 만족시킬 수 있다. `right`를 `length - 1`처럼 좁게 잡으면 이 상태 자체를 나타낼 방법이 아예 없어진다.

## 오답노트

- **1차 시도**: "일치하면 그 자리가 답"이라는 즉시 리턴 조건을 이 문제에 잘못 적용했다. 이 문제는 "더 큰 값 중 최소"를 찾는 문제라 `==`도 아직 답이 아니라는 걸 놓쳤다.
- **2차 시도**: 조건 분기를 다 채우지 않아 `left`/`right`가 갱신되지 않는 경로가 생겼다. 이진 탐색을 짤 때는 모든 분기에서 `left` 또는 `right`가 반드시 변경되는지 매번 확인해야 한다.
- **3차 시도**: 로직은 맞았는데 최종 리턴문을 하드코딩(`letters[0]`)해버린 실수. 로직을 다 맞춰놓고도 마지막 한 줄에서 틀리는 경우가 생각보다 많다는 걸 배웠다.
- **4차 시도(가장 중요)**: "답이 없는 경우"를 배열 인덱스로 표현하지 못했다. `right`를 배열 길이 자체로 잡아야 `left`가 배열 밖까지 갈 수 있다는 발상은 스스로 떠올리지 못하고 힌트를 받았다 — 앞으로 "찾는 대상이 아예 없을 수도 있는" 문제를 만나면 "배열 밖 상태를 인덱스로 어떻게 표현할까"부터 먼저 고민한다.
- **언제 다시 떠올릴까**: 순환/래핑(wrap-around)이 필요한 문제를 만나면 `right`를 넉넉하게 잡고 최종적으로 `% length` 연산을 쓰는 패턴을 가장 먼저 떠올린다.

## 꿀팁

- 이진 탐색 코드를 짤 때 모든 `if`/`else` 분기에서 "이 분기는 `left`나 `right` 중 어떤 걸 바꾸는가"를 표로 미리 그려보면, 2차 시도처럼 갱신이 빠진 분기를 코드 작성 전에 미리 잡아낼 수 있다.
- "답이 없을 수도 있는" 문제 유형은 이 문제(744) 외에도 자주 나온다. `right`를 배열 길이(범위 밖 한 칸)로 넉넉히 잡고 `%`로 순환시키는 이 템플릿을 기억해두면 비슷한 문제에 바로 적용할 수 있다.

## AI라면 어떻게 풀었을까

이 문제도 35번처럼 `Arrays.binarySearch`의 음수 반환 규약을 활용할 수 있다. target과 정확히 같은 값이 있어도 "그보다 큰 값"을 찾아야 하므로 약간의 변형이 필요하다.

```java
int idx = Arrays.binarySearch(letters, (char) (target + 1));
if (idx < 0) {
    idx = -(idx) - 1;
}
return letters[idx % letters.length];
```

`target + 1`을 검색해서 "그 값 이상인 첫 위치"를 찾는 트릭이다 — 직접 짠 이진 탐색 로직과 원리는 같지만, 라이브러리 호출 한 줄로 줄어든다. `% letters.length`로 순환 처리하는 부분은 직접 짠 코드와 동일하게 필요하다.
