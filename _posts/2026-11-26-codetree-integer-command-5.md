---
title: "[코드트리] 정수 명령 처리 5 - next()는 한 단어만 읽고, 번호는 size-1과 k-1"
date: 2026-11-26 17:00:00 +0900
categories: [알고리즘 연습, 배열_리스트]
tags: [java, codetree, arraylist, dynamic-array, scanner]
---

- 문제 링크: 코드트리 정수 명령 처리 5
- 파일 경로: `src/main/java/coding_test/배열_리스트/CT_정수명령처리5.java`
- 난이도: Easy, 20XP (정답률 82%, 평균 21분)
- 제한: 1 ≤ N ≤ 10,000, 정수 1~100,000, Time Limit 3000ms

## 문제 설명

동적 배열(ArrayList)에 대해 명령 N개를 처리한다.

| 명령 | 의미 |
|---|---|
| `push_back A` | 맨 뒤에 A를 넣는다 |
| `pop_back` | 맨 뒤 원소를 삭제한다 |
| `size` | 원소 개수를 출력한다 |
| `get k` | **k번째** 숫자를 출력한다 (1부터 센다) |

```
9
push_back 10 / push_back 20 / get 1 / get 2 / size / pop_back / size / get 1 / size
→ 10 / 20 / 2 / 1 / 10 / 1
```

## 시행착오

구조(명령 이름으로 분기 → ArrayList 사용)는 처음부터 맞았는데, 세 곳에서 틀렸다.

1. **`sc.next()`는 공백 전까지 한 단어만 읽는다.** `push_back 10`을 읽으면 `str`에는 `"push_back"`만 들어 있고 `10`은 입력에 그대로 남아 있다. 그런데 `str.substring(10)`으로 자르려 해서 에러가 났다(`"push_back"`은 9글자). `get 1`도 같다. → 숫자가 따라오는 명령(`push_back`, `get`)에서만 `sc.nextInt()`를 한 번 더 읽는다.
2. **`list.remove(list.size())`** → 마지막 칸의 번호는 `size - 1`이다. 원소가 2개면 번호는 0, 1이라 `remove(2)`는 범위 밖이다.
3. **`list.get(num)`** → 문제의 k번째는 **1부터 센다.** `get 1`의 출력이 첫 원소 10이므로 `list.get(num - 1)`이다.

그 외에 `str.contains("push_back")`처럼 `contains`로 분기하던 것은 `equals`로 바꿨다. 이름이 부분적으로 겹치는 명령이 생기면 오작동하기 때문이다.

## 최종 코드

```java
ArrayList<Integer> list = new ArrayList<>();
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
for (int i = 0; i < n; i++) {
    String str = sc.next();
    if (str.equals("push_back")) {
        int num = sc.nextInt();
        list.add(num);
    } else if (str.equals("pop_back")) {
        list.remove(list.size() - 1);
    } else if (str.equals("size")) {
        System.out.println(list.size());
    } else if (str.equals("get")) {
        int num = sc.nextInt();
        System.out.println(list.get(num - 1));
    }
}
```

## 추적표: 예제 1

| 명령 | list (번호 0, 1, ...) | 출력 |
|---|---|---|
| push_back 10 | [10] | |
| push_back 20 | [10, 20] | |
| get 1 | [10, 20] | `list.get(0)` = **10** |
| get 2 | [10, 20] | `list.get(1)` = **20** |
| size | [10, 20] | **2** |
| pop_back | [10] | `list.remove(2 - 1)` |
| size | [10] | **1** |
| get 1 | [10] | `list.get(0)` = **10** |
| size | [10] | **1** |

## 오늘 배운 내용

- **`Scanner.next()`는 한 토큰(공백 전까지)만 읽는다.** 한 줄에 `이름 숫자`가 같이 있으면 `next()`로 이름, 필요할 때 `nextInt()`로 숫자를 따로 읽는다. `substring`으로 자르는 방식은 한 줄 전체를 읽는 `nextLine()`일 때나 쓴다.
- **마지막 칸의 번호는 `size - 1`.** 번호는 0부터, 개수는 1부터 세서 항상 1 차이가 난다.
- **문제의 "k번째"가 1부터인지 0부터인지 예제로 확인한다.** 예제 `get 1 → 10`에서 1부터 센다는 것을 알 수 있다.
- 숫자가 따라오는 명령은 일부(`push_back`, `get`)뿐이다. 명령마다 입력이 몇 개 더 오는지 구분한다.

## 오답노트

- `substring(10)`, `substring(5)`로 숫자를 자름 → 같은 줄을 한 번에 읽는 것이 아니라서 에러. `nextInt()`로 따로 읽는다.
- `remove(list.size())` → `remove(list.size() - 1)`.
- `get(num)` → `get(num - 1)`.
- `contains` → `equals`.

## AI라면 어떻게 풀었을까

`if-else` 대신 `switch`로 명령 이름을 분기하고, 출력은 `StringBuilder`에 모아서 한 번에 낸다.

```java
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
List<Integer> list = new ArrayList<>();
StringBuilder sb = new StringBuilder();
while (n-- > 0) {
    switch (sc.next()) {
        case "push_back": list.add(sc.nextInt()); break;
        case "pop_back":  list.remove(list.size() - 1); break;
        case "size":      sb.append(list.size()).append('\n'); break;
        case "get":       sb.append(list.get(sc.nextInt() - 1)).append('\n'); break;
    }
}
System.out.print(sb);
```

핵심은 같다. `next()`로 명령 이름만 읽고, 숫자가 따라오는 명령에서만 `nextInt()`를 한 번 더 부른다. N ≤ 10,000이라 줄마다 `println`해도 되지만, `StringBuilder`에 모으면 입출력이 큰 문제에서도 안전하다. `ArrayList`의 맨 뒤 `add`·`remove`와 번호 `get`은 모두 O(1)이다.
