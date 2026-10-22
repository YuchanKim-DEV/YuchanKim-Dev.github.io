---
title: "[정렬] 시작하기 전에 정리하는 Arrays.sort·Comparator와 자주 쓰는 메서드"
date: 2026-10-21 10:00:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, sort, comparator]
---

카테고리: **정렬**
이 글은 문제 하나를 다루는 글이 아니라, 이 카테고리를 시작하기 전에 알아야 할 개념 전체를 한 번에 정리한 글이다. 앞으로 이 카테고리의 다른 글들은 이 글에서 정리한 내용을 전제로, 그 문제에서만 있었던 시행착오를 다룬다.

## 코테에서 정렬이란?

퀵 정렬, 병합 정렬을 **직접 구현할 일은 거의 없다.** 라이브러리(`Arrays.sort`, `Collections.sort`)를 쓰고, 필요하면 **`Comparator`로 정렬 기준만 바꾼다.** 정렬 문제의 핵심은 "무엇을 기준으로 줄 세우나"를 코드로 쓰는 것이다.

## 기본 정렬

| 대상 | 코드 | 비고 |
|---|---|---|
| `int[]` 오름차순 | `Arrays.sort(arr)` | 기본형 배열 |
| 배열 일부만 | `Arrays.sort(arr, from, to)` | `from` 포함, `to` 미포함 |
| `String[]` 사전순 | `Arrays.sort(strs)` | |
| `List` 오름차순 | `Collections.sort(list)` 또는 `list.sort(null)` | |
| 내림차순 (객체 배열) | `Arrays.sort(strs, Collections.reverseOrder())` | `Integer[]`, `String[]`만 가능 |
| 내림차순 (`int[]`) | 오름차순 정렬 후 뒤에서부터 읽기 | `int[]`에는 `reverseOrder()`를 못 쓴다 |

## 배열 자르기·복사

| 하고 싶은 것 | 코드 |
|---|---|
| 일부 복사 | `Arrays.copyOfRange(arr, from, to)` (`from` 포함, `to` 미포함, 0부터) |
| 앞에서 n개 복사 | `Arrays.copyOf(arr, n)` |
| 배열 출력 | `Arrays.toString(arr)` |
| `int[]` → `Integer[]` | `Arrays.stream(arr).boxed().toArray(Integer[]::new)` |

문제에서 "i번째부터 j번째"(1부터 셈)는 `copyOfRange(arr, i - 1, j)`다.

## Comparator: 정렬 기준 바꾸기

`Comparator`는 "**두 개를 받아서 누가 앞에 올지**" 정하는 규칙이다. 결과가 **음수면 앞의 것이 먼저, 양수면 뒤의 것이 먼저, 0이면 같음**이다.

```java
// 람다로 직접 쓰기: n번째 글자 기준
Arrays.sort(strs, (a, b) -> a.charAt(n) - b.charAt(n));

// Comparator 메서드로 쓰기 (읽기 쉽다)
Arrays.sort(strs, Comparator.comparing(s -> s.charAt(n)));

// 기준이 같으면 다음 기준으로 (thenComparing)
Arrays.sort(strs, Comparator.comparing((String s) -> s.charAt(n))
                            .thenComparing(Comparator.naturalOrder()));

// 2차원 배열: 첫 번째 값 기준 오름차순, 같으면 두 번째 값 내림차순
Arrays.sort(arr, (a, b) -> a[0] != b[0] ? a[0] - b[0] : b[1] - a[1]);
```

| 메서드 | 뜻 |
|---|---|
| `Comparator.comparing(키)` | 키를 기준으로 오름차순 |
| `.reversed()` | 거꾸로 |
| `.thenComparing(키)` | 앞 기준이 같으면 다음 기준 |
| `Comparator.naturalOrder()` | 원래 순서(숫자 오름차순, 문자열 사전순) |
| `Comparator.reverseOrder()` | 원래 순서의 반대 |
| `Integer.compare(a, b)` | 두 `int` 비교 (넘침 걱정 없음) |
| `a.compareTo(b)` | 두 문자열 사전순 비교 |

## 정렬로 푸는 대표 패턴

1. **정렬 후 k번째** — 잘라서 정렬하고 인덱스로 꺼내기
2. **기준 바꾸기** — 특정 글자, 길이, 여러 기준 순서대로 (`Comparator`)
3. **이어 붙인 결과로 비교** — "가장 큰 수"처럼 `(a + b)`와 `(b + a)`를 비교
4. **정렬하면 보이는 성질** — H-Index처럼 정렬한 뒤 앞에서부터 조건 확인
   - 오름차순 정렬 후 인덱스 `i`부터 끝까지는 `n - i`개이고 모두 `arr[i]` 이상이다. "x 이상인 개수"를 반복문으로 세지 않고 `n - i`로 바로 얻는다.
5. **정렬 + 그리디** — 작은 것부터(또는 큰 것부터) 처리하면 최적이 되는 문제

## 자주 하는 실수

- **원본을 먼저 정렬**: 일부만 잘라 정렬해야 하는데 원본 전체를 정렬하면 구간이 바뀐다.
- **1번째 / 0번째 혼동**: 문제의 "k번째"는 1부터, 배열 인덱스는 0부터다.
- **`int[]`에 `Collections.reverseOrder()`**: 컴파일 에러다. 기본형 배열에는 `Comparator`를 쓸 수 없다.
- **`a - b` 넘침**: 값이 매우 크거나 음수와 섞이면 `a - b`가 `int` 범위를 넘을 수 있다. `Integer.compare(a, b)`가 안전하다.
- **`Arrays.sort`는 반환값이 없다**: `arr = Arrays.sort(arr)`가 아니라 `Arrays.sort(arr);` 한 줄이다(배열 자체가 바뀐다).

## 시간복잡도

| 연산 | 시간 |
|---|---|
| `Arrays.sort` (기본형) | 평균 `O(n log n)` (듀얼 피벗 퀵 정렬) |
| `Arrays.sort` (객체, `Comparator`) | `O(n log n)`, **안정 정렬**(같은 값은 원래 순서 유지) |
| `Arrays.copyOfRange` | `O(복사 길이)` |

10만 개 정렬은 수십 밀리초면 끝난다. 정렬을 매번 반복(예: 쿼리마다 전체 정렬)하는지만 조심한다.
