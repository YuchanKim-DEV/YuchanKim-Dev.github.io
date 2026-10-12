---
title: "[프로그래머스-Lv1] 같은 숫자는 싫어 - 직전에 쌓은 값과만 비교하면 된다"
date: 2026-10-12 09:44:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, programmers, stack]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/12906
- 파일 경로: `src/main/java/coding_test/스택_큐/Lv1_같은숫자는싫어.java`
- 난이도: Lv.1

## 문제 설명

0~9로 이루어진 배열 `arr`에서 **연속으로 나타나는 같은 숫자는 하나만 남기고** 제거한다. 남은 수들은 원래 순서를 유지해서 반환한다.

```
arr = [1,1,3,3,0,1,1]   → [1,3,0,1]
arr = [4,4,4,3,3]       → [4,3]
```

## 시행착오

이 문제는 **막힌 곳 없이 한 번에 통과**했다. 원래는 "큐에 넣고 `poll`한 뒤 `peek()`로 비교"하는 방식을 생각했는데, 비교 대상을 정하다가 "직전에 결과에 넣은 값과 비교"하는 쪽이 더 자연스럽다는 걸 스스로 알아챘고 스택으로 바꿨다.

확인하느라 한 가지를 질문했다. "큐에서 `peek()`는 가장 처음에 넣은 것만 나오고, `poll()`하기 전까지 그대로 맞나?" — 맞다. 다만 이건 `offer`로 뒤에 넣었을 때의 이야기이고, `push`로 앞에 넣으면 `peek()`가 방금 넣은 값이다. 이번 풀이가 스택을 쓰는 이유가 바로 이것이다. `push`한 마지막 값이 `peek()`로 바로 나오기 때문이다.

## 최종 코드

```java
public int[] solution(int[] arr) {
    Stack<Integer> queue = new Stack<>();      // 변수 이름은 queue지만 실제로는 스택
    queue.push(arr[0]);

    for (int i = 1; i < arr.length; i++) {
        if (queue.peek() != arr[i]) {          // 직전에 쌓은 값과 다르면 쌓는다
            queue.push(arr[i]);
        }
    }

    return queue.stream().mapToInt(Integer::intValue).toArray();
}
```

7개 테스트가 모두 통과했고, 랜덤 30,000건과 100만 개 입력(약 54ms)에서도 정상이었다.

## 추적표: `[1,1,3,3,0,1,1]`

| i | arr[i] | 스택 맨 위 | 비교 | 스택 |
|---|---|---|---|---|
| (시작) | 1 | - | 첫 원소 push | [1] |
| 1 | 1 | 1 | 같음 → 건너뜀 | [1] |
| 2 | 3 | 1 | 다름 → push | [1,3] |
| 3 | 3 | 3 | 같음 → 건너뜀 | [1,3] |
| 4 | 0 | 3 | 다름 → push | [1,3,0] |
| 5 | 1 | 0 | 다름 → push | [1,3,0,1] |
| 6 | 1 | 1 | 같음 → 건너뜀 | [1,3,0,1] |

## 오늘 배운 내용

**"직전에 남긴 것과 같은가?"를 묻는 문제는 스택이다.** 스택의 맨 위(`peek()`)가 곧 "직전에 남긴 값"이다. LC1544, LC2696처럼 인접 짝을 지우는 문제와 같은 계열인데, 이번엔 지우는 게 아니라 **쌓을지 말지**만 정하면 되어서 가장 단순한 형태다.

**`Stack`의 `stream()`은 쌓은 순서(아래에서 위)대로 나온다.** 그래서 `reversed()` 없이 바로 배열로 바꿔도 원래 순서가 유지된다.

## 오답노트

- **틀린 곳은 없었다.** 다만 확인할 점 하나: 변수 이름을 `queue`로 지었지만 실제로는 `Stack`이다. 이름과 자료구조가 다르면 나중에 읽을 때 헷갈린다. **다음에**는 이름을 `stack`으로 짓는다.
- `queue.peek() != arr[i]`에서 `Integer`와 `int`를 `!=`로 비교했는데, 한쪽이 `int`이면 자동으로 값 비교(언박싱)가 되어서 안전하다. 둘 다 `Integer`이면 객체 주소 비교라서 위험하니(-128~127 밖의 값에서 틀림) 그때는 `equals`를 쓴다.

## 꿀팁

`java.util.Stack`은 `Vector` 기반의 옛 클래스라서 모든 메서드에 동기화 비용이 붙는다. 스택이 필요하면 `Deque<Integer> stack = new ArrayDeque<>()`를 쓰는 것이 권장된다(`push`/`pop`/`peek` 사용법은 같다). 다만 `ArrayDeque`로 바꾸면 `push`가 앞에 넣어서 `stream()` 순서가 거꾸로 나오니 주의한다.

## AI라면 어떻게 풀었을까

처음 계획했던 "**큐 + `poll`, `peek()` 비교**"를 그대로 완성한 방식이다. 큐에서 하나를 `poll`한 뒤 **다음 값(`peek()`)과 비교**한다. 다음 값이 같으면 이 값은 연속의 중간이라 버리고, 다르거나 큐가 비었으면 연속의 마지막이라 남긴다.

```java
public int[] solutionAlt(int[] arr) {
    Queue<Integer> queue = new ArrayDeque<>();
    for (int n : arr) {
        queue.offer(n);
    }

    List<Integer> result = new ArrayList<>();
    while (!queue.isEmpty()) {
        int cur = queue.poll();
        if (queue.isEmpty() || queue.peek() != cur) {
            result.add(cur);                 // 다음 값과 다르면 연속의 마지막 → 남김
        }
    }
    return result.stream().mapToInt(Integer::intValue).toArray();
}
```

같은 값끼리 묶인 구간에서 마지막 하나만 남기므로 결과가 같다. 트레이드오프: 입력 전체를 큐에 복사하는 단계가 늘고 결과를 `List`로 따로 모아야 하지만, "앞에서부터 하나씩 꺼내며 다음 것과 비교"하는 큐의 사용법을 그대로 연습할 수 있다. 스택 방식이 코드는 더 짧다. 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
