---
title: "[프로그래머스-Lv2] 튜플 - 문자열을 쪼개고, 짧은 묶음부터 본다"
date: 2026-10-26 11:29:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, string, hashset, kakao]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/64065
- 파일 경로: `src/main/java/coding_test/정렬/Lv2_튜플.java`
- 난이도: Lv.2 (2019 카카오 개발자 겨울 인턴십)

{% raw %}
## 문제 설명

설명은 길지만 내용은 단순하다. 원래 숫자 줄 `(2, 1, 3, 4)`를 **앞에서부터 1개, 2개, 3개, 4개씩 자른 묶음**으로 만든다.

```
{2}  {2,1}  {2,1,3}  {2,1,3,4}
```

이 묶음들의 순서와 묶음 안 숫자의 순서를 모두 섞어 문자열로 준다. 이 문자열에서 원래 줄을 되찾는다.

```
"{{1,2,3},{2,1},{1,2,4,3},{2}}"  → [2, 1, 3, 4]
"{{20,111},{111}}"               → [111, 20]
```

**핵심:** 묶음을 크기가 작은 순서로 다시 세우면, 묶음마다 이전 묶음에 없던 숫자가 딱 하나씩 새로 나온다. 그 숫자를 차례로 모으면 원래 줄이 된다.

| 묶음 (크기순) | 새로 나온 숫자 |
|---|---|
| {2} | **2** |
| {2, 1} | **1** |
| {1, 2, 3} | **3** |
| {1, 2, 4, 3} | **4** |

## 시행착오

처음에는 문제 설명이 무슨 말인지부터 감이 오지 않았다. 문제 해석과 흐름(쪼개기 → 길이순 정렬 → 새 숫자 모으기)을 먼저 받고 시작했다.

### 1차: `set.(arr[i])` — 묶음 문자열을 통째로 넣으려 함

쪼개기와 길이순 정렬은 흐름대로 바로 짰다. 그런데 `arr[i]`에는 `"1,2,3"` 같은 **묶음 전체 문자열**이 들어 있어서 `HashSet<Integer>`에 넣을 수 없었다. 묶음을 한 번 더 쪼개야 한다.

### 2차: `charAt`으로 한 글자씩 읽기 — 여러 자리 숫자가 깨짐

```java
char c = arr[i].charAt(j);
if (c != ',') { ... c - '0' ... }
```

한 글자씩 보면 `"111"`이 숫자 111이 아니라 1, 1, 1 세 개가 된다. 원소는 최대 100,000이라 여러 자리가 흔하다.

### 3차: 통과

묶음을 `split(",")`으로 쪼개 숫자를 통째로 꺼내고 `Integer.parseInt`로 바꿨다. `set.add()`가 새로 들어갔을 때만 `true`를 돌려주는 점을 이용해, 확인과 추가를 한 줄로 처리했다.

## 최종 코드

```java
public int[] solution(String s) {
    s = s.substring(2, s.length() - 2);                    // 앞 "{{", 뒤 "}}" 떼기
    String[] arr = s.split("\\},\\{");                     // 묶음별로 쪼개기
    Arrays.sort(arr, (a, b) -> a.length() - b.length());   // 짧은 묶음부터

    int[] answer = new int[arr.length];
    HashSet<Integer> set = new HashSet<>();
    int idx = 0;
    for (int i = 0; i < arr.length; i++) {
        String[] temp = arr[i].split(",");                 // 묶음 안 숫자 쪼개기
        for (int j = 0; j < temp.length; j++) {
            if (set.add(Integer.parseInt(temp[j]))) {      // 처음 나온 숫자면 true
                answer[idx] = Integer.parseInt(temp[j]);
                idx++;
            }
        }
    }
    return answer;
}
```

묶음 개수가 곧 튜플 길이라서 `answer` 크기를 `arr.length`로 잡을 수 있다.

## 추적표: `"{{20,111},{111}}"`

| 단계 | 결과 |
|---|---|
| substring | `"20,111},{111"` |
| split `"},{"` | `["20,111", "111"]` |
| 길이순 정렬 | `["111", "20,111"]` |
| `"111"` | 111 새로 나옴 → `[111]` |
| `"20,111"` | 20 새로 나옴(111은 `add`가 false) → `[111, 20]` |

## 오늘 배운 내용

**문자열 파싱 3종 세트: `substring` → `split` → `parseInt`.** 감싸는 기호는 `substring`으로 떼고, 구분자로 `split`하고, 숫자 문자열은 `Integer.parseInt`로 바꾼다. `split`의 인자는 정규식이라 `{`, `}` 같은 특수문자 앞에는 `\\`를 붙인다.

**`HashSet.add()`는 boolean을 돌려준다.** 새로 들어가면 `true`, 이미 있으면 `false`다. `contains`로 확인하고 `add`하는 두 줄을 `if (set.add(x))` 한 줄로 줄일 수 있다.

## 오답노트

- **막혔던 지점 1**: 문제 해석. **왜**: 설명이 길고 집합 기호가 많아 "무엇을 되찾는 건지"가 안 보였다. **다음에**: 긴 문제는 입출력 예 하나를 손으로 풀어 보고, 입력과 출력 사이에서 무슨 일이 일어났는지부터 찾는다.
- **막혔던 지점 2**: `charAt`으로 숫자를 한 글자씩 읽음. **왜**: 숫자가 한 자리라고 가정했다. **다음에**: 제한사항에서 원소 최댓값을 보고, 두 자리 이상이면 `split` + `parseInt`로 통째로 꺼낸다.

## 꿀팁

**`char` → `int`는 `c - '0'`.** `'7'`(55) − `'0'`(48) = 7이다. 단, 한 자리 숫자일 때만 쓴다.

**길이순 정렬이 안전한 이유.** 다음 묶음은 이전 묶음에 숫자 하나가 더 붙은 것이라 문자열도 항상 더 길다. 그래서 숫자 개수 대신 `length()`로 정렬해도 된다.

{% endraw %}

## AI라면 어떻게 풀었을까

**많이 등장한 숫자일수록 앞이다.** 첫 원소는 모든 묶음에, 두 번째 원소는 하나 빼고 전부에 들어 있다. 묶음 구분 없이 숫자만 전부 뽑아 등장 횟수를 세고, 횟수 내림차순으로 정렬한다. [LC1636]({% post_url 2026-10-23-sort-by-frequency-lc1636 %}) 빈도 정렬과 같은 모양이다.

```java
public int[] solutionAlt(String s) {
    HashMap<Integer, Integer> count = new HashMap<>();
    for (String num : s.replaceAll("[{}]", " ").replace(",", " ").trim().split("\\s+")) {
        int x = Integer.parseInt(num);
        count.put(x, count.getOrDefault(x, 0) + 1);
    }
    Integer[] keys = count.keySet().toArray(new Integer[0]);
    Arrays.sort(keys, (a, b) -> count.get(b) - count.get(a));   // 많이 나온 숫자가 앞
    int[] answer = new int[keys.length];
    for (int i = 0; i < keys.length; i++) {
        answer[i] = keys[i];
    }
    return answer;
}
```

트레이드오프: 묶음을 나눌 필요가 없어 파싱이 단순하다. 대신 "등장 횟수 = 순서"라는 관찰을 떠올려야 한다. 본인 풀이는 문제 설명을 그대로 따라가서 떠올리기 쉽다. 무작위로 만든 튜플 20,000개에서 두 풀이 모두 정답과 같았다.
