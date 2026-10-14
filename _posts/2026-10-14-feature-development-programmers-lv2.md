---
title: "[프로그래머스-Lv2] 기능개발 - 큐에 먼저 다 넣고, 한 묶음씩 꺼낸다"
date: 2026-10-14 15:36:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, programmers, queue, simulation]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/42586
- 파일 경로: `src/main/java/coding_test/스택_큐/Lv2_기능개발.java`
- 난이도: Lv.2

## 문제 설명

기능마다 진도(`progresses`)와 개발 속도(`speeds`)가 있고, 진도가 100%가 되면 배포할 수 있다. 뒤 기능이 먼저 끝나도 **앞 기능이 배포될 때 함께** 배포된다. 각 배포마다 몇 개의 기능이 나가는지 배열로 반환한다.

```
progresses = [93,30,55], speeds = [1,30,5]  → [2, 1]
progresses = [95,90,99,99,80,99], speeds = [1,1,1,1,1,1] → [1, 3, 2]
```

## 시행착오

어제(10/13) 시작해서 하루를 넘겨 오늘(10/14) 다시 풀었다. 어제는 시간 박스를 크게 넘겼고 미통과였다.

### 1차: "며칠 걸리는지" 계산식

```java
int num = (100 / progresses[i]) + (100 % progresses[i]);          // 속도를 안 씀
int num = ((100 - p) / s) + ((100 - p) % s);                       // 나머지 값을 그대로 더함
```

첫 식은 `speeds`를 한 번도 쓰지 않았다. 두 번째 식은 속도를 넣었지만 **나머지의 값**(예: 10)을 더했다. 나머지가 10이라는 건 "10일 더"가 아니라 **"하루가 더 필요하다"**는 뜻이다. 예시 1의 두 번째 기능은 `70 / 30 = 2`일에 10이 모자라서 **3일**인데 식은 12가 나왔다. 올림 나눗셈 `(a + b - 1) / b` 한 줄로 해결됐다. 그런데 처음엔 이 한 줄 위에 `if`로 한 번 더 `+1`을 해서 **이중 올림**이 됐다.

### 2차: `Stack`으로 묶기

```java
} else if (stack.peek() >= num) { ... pop 전부 ... }
else if (stack.peek() < num) { stack.push(num); }   // 새 배포가 시작돼도 앞 묶음을 닫지 않음
```

더 큰 일수가 나오면 **새 배포가 시작**되어 앞 묶음을 끝내야 하는데, 쌓기만 했다. 그리고 이 문제는 앞에서부터 순서대로 처리하는 **큐** 문제인데 스택을 썼다.

핵심 규칙은 대화로 정리됐다. "`7, 3, 9`에서 새 배포는 **9에서 시작**한다. 앞 기능(7일)보다 오래 걸리는 기능이 나오는 때다. 첫 기능은 무조건 새 배포의 시작이다."

### 3차: 큐로 바꾼 뒤의 두 가지

```java
while (deque.peek() >= progress) {          // NullPointerException — 큐가 비면 peek()가 null
```

큐가 비었는데 `peek()`를 비교해서 NPE가 났다. `!deque.isEmpty() &&`를 **앞에** 붙여 해결했다. 또 큐를 `for` 안에서 하나씩 채우면서 동시에 꺼내던 구조도, **일수를 먼저 다 넣고 별도 `while`에서 처리**하는 구조로 바꾸자 정리가 됐다.

### 4차: 부등호 방향

```java
while (!deque.isEmpty() && pollNum <= deque.peek()) {   // 7 <= 3 → 거짓
```

기준일 `7`, 다음 `3`일 때 "같이 나간다"가 참이어야 하는데 `7 <= 3`은 거짓이다. **다음 값이 기준 이하**여야 같이 나가므로 `pollNum >= deque.peek()`로 고쳤다.

## 최종 코드

```java
public int[] solution(int[] progresses, int[] speeds) {
    Deque<Integer> deque = new ArrayDeque<>();
    List<Integer> list = new ArrayList<>();

    for (int i = 0; i < progresses.length; i++) {
        int days = (100 - progresses[i] + speeds[i] - 1) / speeds[i];   // 올림 나눗셈
        deque.offer(days);                                              // 일수를 먼저 다 넣는다
    }

    while (!deque.isEmpty()) {
        int counter = 1;
        int pollNum = deque.poll();                                     // 기준일
        while (!deque.isEmpty() && pollNum >= deque.peek()) {           // 다음이 기준 이하면 함께
            deque.poll();
            counter++;
        }
        list.add(counter);                                              // 한 묶음 끝
    }

    return list.stream().mapToInt(Integer::intValue).toArray();
}
```

8개 테스트가 모두 통과했고, 랜덤 30,000건에서도 정답 계산과 일치했다.

## 추적표: 일수 `[7, 3, 9]`

| 단계 | 큐(앞→뒤) | 기준일 | 하는 일 | 개수 | 결과 |
|---|---|---|---|---|---|
| 1 | 7,3,9 | - | 7을 poll해서 기준일로 | 1 | [] |
| 2 | 3,9 | 7 | peek 3 ≤ 7 → 함께 poll | 2 | [] |
| 3 | 9 | 7 | peek 9 > 7 → 묶음 종료 | - | [2] |
| 4 | 9 | - | 9를 poll해서 기준일로 | 1 | [2] |
| 5 | (빔) | 9 | 큐가 비어 종료 | - | [2, 1] |

## 오늘 배운 내용

**"앞에서부터 순서대로 처리하되, 뒤에 이어지는 것들을 한 묶음으로 처리"하는 문제는 큐 + `peek`다.** 맨 앞을 `poll`해서 기준으로 삼고, `peek`로 다음 것이 조건에 맞는 동안 계속 `poll`하면 한 묶음이다. 큐를 **먼저 다 채운 뒤** 별도의 `while`에서 처리하는 구조가 안정적이다.

**올림 나눗셈 `(a + b - 1) / b`는 "나머지가 있으면 하나 더"를 한 줄로 해결한다.** 나머지의 **값이 아니라 있는지 없는지**가 의미다.

## 오답노트

- **틀렸던 지점 1**: 나머지의 값을 그대로 더함. **왜**: "나머지가 있으면 하루 더"를 "나머지를 더한다"로 옮겼다. **다음에**: 나눗셈에 나머지가 나오면 "며칠/몇 개/몇 번"처럼 **단위를 올림하는 것인지** 확인한다.
- **틀렸던 지점 2**: 올림 공식 위에 `if`로 한 번 더 `+1` (이중 올림). **왜**: 두 가지 방법을 둘 다 적용했다. **다음에**: 같은 목적의 코드가 두 번 들어갔는지 한 번 읽는다.
- **틀렸던 지점 3**: 새 묶음이 시작될 때 앞 묶음을 닫지 않음. **왜**: "묶는 규칙"을 한 줄로 말하지 않은 채 코드부터 썼다. **다음에**: 묶는 규칙을 `7, 3, 9` 같은 작은 예시로 먼저 한 줄로 쓴다.
- **틀렸던 지점 4**: 빈 큐에 `peek()`. **왜**: 안쪽 `while`이 큐를 다 비울 수 있다는 걸 놓쳤다. **다음에**: `peek`를 쓰는 조건 앞에는 `!isEmpty() &&`를 먼저 쓴다. (카드 뭉치에서도 같은 실수가 있었다.)
- **틀렸던 지점 5**: 부등호 방향. **왜**: "기준일 이하" 문장을 코드로 옮길 때 어느 변수가 기준인지 헷갈렸다. **다음에**: 작은 예시(`7`, `3`)를 조건에 직접 대입해서 참/거짓을 확인한다.
- **좋았던 점**: 큐로 바꾸자마자 구조(먼저 다 넣고 별도 `while`)를 스스로 잡았다. 부등호 하나만 남았을 때 예시를 대입하자 바로 찾았다.

## 꿀팁

`peek()`는 큐가 비면 `null`이라, `int`와 비교하는 순간 NPE가 난다. **"비어 있지 않다"를 항상 앞에** 쓴다. `while (!q.isEmpty() && q.peek() ...)`가 한 덩어리다.

## AI라면 어떻게 풀었을까

며칠 걸리는지 **미리 계산하지 않고, 문제 설명 그대로 시간을 하루씩 흘려보낸다.** 큐에는 기능의 번호를 순서대로 넣고, 하루가 지날 때마다 모든 기능의 진도를 속도만큼 올린 뒤 **큐 맨 앞(`peek`)의 진도가 100 이상이면 연달아 `poll`**해서 그날의 배포 개수로 센다. 앞 기능이 안 끝났으면 `peek`에서 막혀서 뒤 기능은 자동으로 못 나간다.

```java
public int[] solutionAlt(int[] progresses, int[] speeds) {
    int n = progresses.length;
    int[] now = progresses.clone();                  // 입력 보존
    Queue<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < n; i++) {
        queue.offer(i);
    }

    List<Integer> result = new ArrayList<>();
    while (!queue.isEmpty()) {
        for (int i = 0; i < n; i++) {
            now[i] += speeds[i];                     // 하루가 지남
        }
        int count = 0;
        while (!queue.isEmpty() && now[queue.peek()] >= 100) {
            queue.poll();                            // 맨 앞이 끝났으면 함께 배포
            count++;
        }
        if (count > 0) {
            result.add(count);                       // 그날 배포가 있었음
        }
    }
    return result.stream().mapToInt(Integer::intValue).toArray();
}
```

트레이드오프: 올림 나눗셈과 기준일 비교가 필요 없고 문제 설명을 그대로 옮긴 거라 직관적이다. 대신 날마다 모든 기능의 진도를 올리느라 일수만큼 반복이 늘어난다(이 문제는 최대 100개 기능, 최대 100일이라 충분히 빠르다). 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
