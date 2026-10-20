---
title: "[해시] 시작하기 전에 정리하는 HashMap·HashSet과 자주 쓰는 메서드"
date: 2026-10-20 09:10:00 +0900
categories: [알고리즘 연습, 해시]
tags: [java, hash, hashmap, hashset]
---

카테고리: **해시**
이 글은 문제 하나를 다루는 글이 아니라, 이 카테고리를 시작하기 전에 알아야 할 개념 전체를 한 번에 정리한 글이다. 앞으로 이 카테고리의 다른 글들은 이 글에서 정리한 내용을 전제로, 그 문제에서만 있었던 시행착오를 다룬다.

## 해시(Hash)란?

**키(key)로 값(value)을 바로 찾는 표**다. 배열은 "몇 번째 칸인가"(인덱스)로 값을 찾지만, 해시는 **이름·문자열·숫자 같은 키 자체**로 값을 찾는다. 찾기·넣기·지우기가 평균 `O(1)`이라, "이게 있었나?", "이게 몇 번 나왔나?"를 반복해서 묻는 문제에서 이중 `for`문을 없애 준다.

Java에서는 두 가지를 주로 쓴다.

| 자료구조 | 저장하는 것 | 언제 쓰나 |
|---|---|---|
| `HashMap<K, V>` | 키 → 값 쌍 | 개수 세기, 이름별 정보 저장, 그룹 묶기 |
| `HashSet<E>` | 키만 (중복 없음) | 존재 확인, 중복 제거, 종류 수 세기 |

```java
Map<String, Integer> map = new HashMap<>();
Set<Integer> set = new HashSet<>();
```

## HashMap 자주 쓰는 메서드

| 하고 싶은 것 | 코드 | 비고 |
|---|---|---|
| 넣기 / 덮어쓰기 | `map.put(k, v)` | 같은 키가 있으면 값이 바뀐다 |
| 꺼내기 | `map.get(k)` | 없으면 **`null`** |
| 없을 때 기본값 | `map.getOrDefault(k, 0)` | 개수 세기의 기본 |
| 키가 있나 | `map.containsKey(k)` | |
| 값이 있나 | `map.containsValue(v)` | `O(n)`이라 느림 |
| 지우기 | `map.remove(k)` | |
| 개수 | `map.size()`, `map.isEmpty()` | |
| 없을 때만 넣기 | `map.putIfAbsent(k, v)` | |
| 합치며 넣기 | `map.merge(k, 1, Integer::sum)` | 없으면 1, 있으면 +1. 결과값을 돌려준다 |
| 없으면 만들어서 꺼내기 | `map.computeIfAbsent(k, x -> new ArrayList<>()).add(v)` | 그룹 묶기 |
| 키만 훑기 | `for (K k : map.keySet())` | |
| 값만 훑기 | `for (V v : map.values())` | |
| 키·값 같이 훑기 | `for (Map.Entry<K, V> e : map.entrySet())` | `e.getKey()`, `e.getValue()` |

### 개수 세기 (가장 많이 쓴다)

```java
for (String name : names) {
    map.put(name, map.getOrDefault(name, 0) + 1);   // 또는 map.merge(name, 1, Integer::sum);
}
```

## HashSet 자주 쓰는 메서드

| 하고 싶은 것 | 코드 | 비고 |
|---|---|---|
| 넣기 | `set.add(x)` | 이미 있으면 `false`를 돌려주고 무시 |
| 있나 | `set.contains(x)` | 해시의 핵심. `O(1)` |
| 지우기 | `set.remove(x)` | |
| 종류 수 | `set.size()` | 중복이 빠진 개수 |
| 훑기 | `for (E x : set)` | 순서 보장 없음 |
| 리스트로 만들기 | `new HashSet<>(Arrays.asList(arr))` | `String[]` 같은 객체 배열만 |
| `int[]`로 만들기 | `Arrays.stream(arr).boxed().collect(Collectors.toSet())` | 기본형 배열은 박싱 필요 |

```java
Set<Integer> set = new HashSet<>();
for (int n : nums) {
    set.add(n);              // 중복은 자동으로 빠진다
}
int kinds = set.size();      // 종류 수
```

## 해시로 푸는 대표 패턴

1. **개수 세기** — 이름·숫자별 등장 횟수 (`getOrDefault + 1`, `merge`)
2. **존재 확인 / 중복 판별** — "본 적 있나?"를 `contains`로 `O(1)`에 (`HashSet`)
3. **종류 수 세기** — `set.size()`
4. **짝 찾기** — "target - x가 이미 나왔나?" (LeetCode 1 Two Sum처럼 값 → 인덱스 저장)
5. **그룹 묶기** — 같은 기준(정렬한 문자열, 첫 글자 등)끼리 리스트로 (`computeIfAbsent`)
6. **이름 ↔ 번호 바꾸기** — 문자열을 번호로 바꿔 배열처럼 쓰기

## 자주 하는 실수

- **없는 키를 `get`해서 NPE**: `int c = map.get(k);`에서 `k`가 없으면 `null`을 `int`로 바꾸다 터진다. `getOrDefault`를 쓴다.
- **`Integer`끼리 `==` 비교**: `map.get(a) == map.get(b)`는 -128~127 밖의 값에서 `false`가 될 수 있다. `.equals()`나 `int`로 꺼내서 비교한다.
- **순서를 기대함**: `HashMap`, `HashSet`은 넣은 순서도, 정렬 순서도 보장하지 않는다. 넣은 순서가 필요하면 `LinkedHashMap`, 정렬 순서가 필요하면 `TreeMap`.
- **순회 중에 지우기**: `for (K k : map.keySet()) map.remove(k);`는 `ConcurrentModificationException`. 지울 키를 따로 모았다가 지우거나 `Iterator.remove()`를 쓴다.
- **배열을 키로 씀**: `int[]`는 내용이 같아도 다른 키로 취급된다. `Arrays.toString(arr)`이나 `List<Integer>`로 바꿔서 키로 쓴다.

## 시간복잡도

| 연산 | HashMap / HashSet | 비교: 배열에서 찾기 |
|---|---|---|
| 넣기 | 평균 `O(1)` | - |
| 찾기 (`get`, `contains`) | 평균 `O(1)` | `O(n)` |
| 지우기 | 평균 `O(1)` | `O(n)` |
| 전체 훑기 | `O(n)` | `O(n)` |

"찾기를 n번 반복"하면 배열은 `O(n²)`, 해시는 `O(n)`이다. 입력이 10만 개 이상이고 이중 `for`문이 보이면 해시를 먼저 떠올린다.
