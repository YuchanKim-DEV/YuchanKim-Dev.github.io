---
title: "[LeetCode-1598] 폴더 이동도 결국 스택에 쌓인 개수다"
date: 2026-10-05 16:40:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/crawler-log-folder/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC1598_CrawlerLogFolder.java`
- 난이도: Easy

## 문제 설명

폴더 이동 기록 `logs`가 주어진다. 명령은 세 종류다.

- `"../"`: 상위 폴더로 이동한다. 이미 메인 폴더면 그대로 있는다.
- `"./"`: 현재 폴더에 그대로 있는다.
- `"x/"`: 하위 폴더 `x`로 들어간다.

모든 이동이 끝난 뒤, 메인 폴더로 돌아가려면 최소 몇 번 이동해야 하는지 반환한다.

```
Input: logs = ["d1/","d2/","../","d21/","./"]
Output: 2
```

## 시행착오

힌트 없이 한 번에 통과했다. 직전에 푼 [LC1614]({% post_url 2026-10-05-maximum-nesting-depth-lc1614 %})에서 "스택에 쌓인 개수가 곧 깊이"라는 걸 배웠는데, 이 문제도 똑같은 구조라 바로 연결됐다.

- 하위 폴더로 들어가면 `push`, 상위로 가면 `pop`, 제자리면 아무것도 안 한다.
- 다 끝났을 때 스택에 남은 개수 = 메인까지 올라가야 하는 횟수.

유일한 함정은 **메인 폴더에서 `"../"`가 나올 때**다. 빈 스택에서 `pop()`을 하면 `NoSuchElementException`이 터지는데, 처음부터 `isEmpty()`로 막아뒀다.

## 최종 코드

```java
public int minOperations(String[] logs) {
    Deque<String> stack = new ArrayDeque<>();
    for (int i = 0; i < logs.length; i++) {
        String str = logs[i];
        if (str.equals("../")) {
            if (!stack.isEmpty()) {
                stack.pop();
            }
        } else if (str.equals("./")) {
            continue;
        } else {
            stack.push(str);
        }
    }
    return stack.size();
}
```

## 오늘 배운 내용

**같은 아이디어가 다른 옷을 입고 나온다.** 괄호 깊이(1614)와 폴더 깊이(1598)는 겉모습은 다르지만 "들어가면 쌓고, 나오면 빼고, 개수가 곧 깊이"라는 같은 문제다. 새 문제를 보면 "전에 본 것 중 비슷한 구조가 있나?"부터 떠올리면 된다.

그리고 1614와 달리 이 문제는 **입력이 올바르다는 보장이 없다**(메인에서 또 올라가려 할 수 있음). 1614는 "괄호가 항상 짝이 맞는다"는 조건이 있어서 빈 스택 검사가 필요 없었다. 문제 조건에 따라 빈 스택 방어가 필요한지가 달라진다.

## 오답노트

- 이번엔 틀린 곳 없음.
- **다음에 떠올릴 시점**: `pop()`을 쓸 때마다 "여기서 스택이 비어 있을 수 있나?"를 한 번 묻는다. 문제 조건에 "항상 올바르다"는 보장이 없으면 방어한다.

## AI라면 어떻게 풀었을까

`pop()` 대신 `poll()`을 쓰면 빈 스택에서 예외를 던지지 않고 `null`을 돌려주기 때문에 `isEmpty()` 검사를 따로 쓰지 않아도 된다. `"./"`는 아무것도 하지 않으면 되니 분기에서 아예 뺐다.

```java
public int minOperations(String[] logs) {
    Deque<String> stack = new ArrayDeque<>();
    for (String log : logs) {
        if (log.equals("../")) {
            stack.poll();          // 비어 있으면 null만 반환, 예외 없음
        } else if (!log.equals("./")) {
            stack.push(log);
        }
    }
    return stack.size();
}
```

`ArrayDeque`에서 `pop()`은 비어 있으면 `NoSuchElementException`을 던지고, `poll()`은 `null`을 반환한다 ([Java 21 ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html)). 다만 "비어 있을 때 빼려 한다"는 상황이 코드에 드러나지 않으니, 처음에는 지금처럼 `isEmpty()`로 명시하는 쪽이 읽기 더 쉽다.
