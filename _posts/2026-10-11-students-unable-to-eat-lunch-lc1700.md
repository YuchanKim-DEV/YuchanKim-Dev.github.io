---
title: "[LeetCode-1700] 스택의 '맨 위'가 어느 쪽인지는 문제가 정한다"
date: 2026-10-11 14:20:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, leetcode, stack, queue, simulation]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://leetcode.com/problems/number-of-students-unable-to-eat-lunch/
- 파일 경로: `src/main/java/coding_test/스택_큐/LC1700_NumberOfStudentsUnableToEatLunch.java`
- 난이도: Easy

## 문제 설명

학생들이 한 줄(큐)로 서 있고, 샌드위치(0=원형, 1=사각)는 스택에 쌓여 있다. 맨 앞 학생이 맨 위 샌드위치를 좋아하면 가져가고 줄에서 나가며, 아니면 샌드위치를 두고 줄 맨 뒤로 간다. 줄의 모든 학생이 맨 위 샌드위치를 원하지 않으면 끝난다. 못 먹은 학생 수를 구한다. **`sandwiches[0]`이 스택의 맨 위다.**

```
Input: students = [1,1,1,0,0,1], sandwiches = [1,0,0,0,1,1]
Output: 3
```

## 시행착오

토요일 [LC2073]({% post_url 2026-10-10-time-needed-to-buy-tickets-lc2073 %})의 "큐에 식별자를 넣는다"를 그대로 가져와서 학생 번호를 큐에, 샌드위치 번호를 스택에 넣는 구조로 시작했다. 방향은 맞았는데 세 가지가 어긋났다.

### 1차: `else if` 조건이 `if`와 똑같음 → 무한 루프

```java
if (students[qPeek] == sandwiches[sPeek]) { ... }
else if (students[qPeek] == sandwiches[sPeek]) { ... }   // 위와 같은 조건
```

"아니면 뒤로 간다"는 쪽이 `!=`여야 하는데 `==`를 또 썼다. 그래서 맞지 않을 때 `count++`가 한 번도 실행되지 않았고, `count`가 0에서 안 움직이니 `break`가 걸릴 수 없어 `while`이 끝나지 않았다. 예시 1조차 출력이 안 나왔다.

### 2차: 조건은 고쳤지만 `peekLast`와 `push` 방향

```java
int qPeek = queue.peekLast();                 // 줄의 "맨 앞"이어야 하는데 맨 뒤
for (...) stack.push(i);                       // 0..n-1을 push → 맨 위가 n-1번
```

7개 중 3개가 틀렸다(예시 1, 예시 2, Edge 5).

1. **`peekLast()`는 줄의 맨 뒤다.** 줄은 `offer`로 뒤에 넣고 앞에서 꺼내는데, 맨 앞 학생을 `peek()`가 아니라 `peekLast()`로 봤다.
2. **`push(0)`부터 `push(n-1)`까지 하면 맨 위가 `sandwiches[n-1]`이다.** 문제는 `sandwiches[0]`이 맨 위라고 한다.

"예시 2는 2 아닌가요?"라고 물었는데, 손으로 따라간 계산에서 샌드위치 `100011`에서 **오른쪽 끝의 `1`**을 빼고 있었다. 오른쪽을 맨 위로 보는 감각이 `push` 방향 실수와 같은 뿌리였다. 문제 원문(`i = 0 is the top of the stack`)을 다시 읽고서야 보였다.

## 최종 코드

```java
public int countStudents(int[] students, int[] sandwiches) {
    Deque<Integer> queue = new ArrayDeque<>();
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < students.length; i++) {
        queue.offer(i);                         // 학생 번호
    }
    for (int i = 0; i < sandwiches.length; i++) {
        stack.offer(i);                         // 샌드위치 번호 (앞이 맨 위)
    }

    int count = 0;                              // 연속으로 못 가져간 횟수

    while (!queue.isEmpty() && !stack.isEmpty()) {
        int qPeek = queue.peek();
        int sPeek = stack.peek();

        if (students[qPeek] == sandwiches[sPeek]) {
            stack.poll();
            queue.poll();
            count = 0;
        } else if (students[qPeek] != sandwiches[sPeek]) {
            int back = queue.poll();
            queue.offer(back);
            count++;
        }

        if (count == queue.size()) {
            break;
        }
    }
    return queue.size();
}
```

## 추적표: `students=[1,1,1,0,0,1]`, `sandwiches=[1,0,0,0,1,1]`

| 단계 | 줄(앞→뒤) | 맨 위 샌드위치 | 처리 | count | 줄 길이 |
|---|---|---|---|---|---|
| 1 | 1,1,1,0,0,1 | `sandwiches[0]=1` | 가져감 | 0 | 5 |
| 2 | 1,1,0,0,1 | `sandwiches[1]=0` | 1 → 뒤로 | 1 | 5 |
| 3 | 1,0,0,1,1 | 0 | 1 → 뒤로 | 2 | 5 |
| 4 | 0,0,1,1,1 | 0 | 가져감 | 0 | 4 |
| 5 | 0,1,1,1 | `sandwiches[2]=0` | 가져감 | 0 | 3 |
| 6 | 1,1,1 | `sandwiches[3]=0` | 1 → 뒤로 ×3 | 3 | 3 |
| 끝 | 1,1,1 | | `count == 줄 길이` → break | 3 | **3** |

## 오늘 배운 내용

**종료 조건은 "연속으로 못 가져간 횟수가 줄 길이와 같아지는 것"이다.** 줄이 비지 않아도 끝나는 문제라서 `while (!queue.isEmpty())`만으로는 영원히 돈다. "한 바퀴를 돌았는데 아무도 못 가져갔다"를 `count == queue.size()` 한 줄로 쓰는 게 핵심이다. 가져가는 순간 `count = 0`으로 되돌려야 "연속"이 된다.

## 오답노트

- **틀렸던 지점 1**: `else if`에 같은 조건을 복사해 `==`를 두 번 씀. **왜**: 위 줄을 복사해 붙이고 연산자만 바꾸는 걸 잊었다. **다음에**: `else`가 필요한 곳에 같은 조건이 반복되면 "정말 다른 경우인가?"를 먼저 본다.
- **틀렸던 지점 2**: `peekLast()`를 줄의 맨 앞으로 착각. **왜**: `peek`을 "마지막 값"으로 알고 있었다. `push`를 쓸 때만 그렇게 보인다. **다음에**: `Deque`의 `peek`은 항상 head(앞)이고, 맨 뒤는 `peekLast`다.
- **틀렸던 지점 3**: 스택의 맨 위를 배열의 오른쪽 끝으로 봄. **왜**: 자바에서 `push`한 마지막 값이 맨 위라서 "오른쪽이 맨 위"라는 감각이 굳어 있다. **다음에**: 문제에서 스택의 방향을 어디에 적었는지(`i = 0 is the top`) 먼저 확인한다.
- **좋았던 점**: 어제 수도코드로 배운 "변하는 것과 변하지 않는 것" 질문이 있어서, 이번엔 구조(번호를 큐에, 배열을 상태로)를 **수도코드 없이** 혼자 잡았다.

## AI라면 어떻게 풀었을까

학생의 선호와 샌드위치 종류는 풀이 중에 변하지 않는다. LC2073은 남은 표 수가 변해서 "번호"를 넣어야 했지만, 여기서는 **번호 대신 값 자체**를 큐와 스택에 넣어도 된다. 그러면 `students[]`, `sandwiches[]`를 다시 찾아볼 필요 없이 `peek()`끼리 바로 비교한다. 샌드위치는 뒤에서부터 `push`해서 `sandwiches[0]`이 맨 위가 되게 한다.

```java
public int countStudentsAlt(int[] students, int[] sandwiches) {
    Queue<Integer> queue = new ArrayDeque<>();
    for (int s : students) {
        queue.offer(s);
    }
    Deque<Integer> stack = new ArrayDeque<>();
    for (int i = sandwiches.length - 1; i >= 0; i--) {
        stack.push(sandwiches[i]);              // 마지막에 넣은 sandwiches[0]이 맨 위
    }

    int rotate = 0;                             // 연속으로 못 가져간 횟수
    while (!queue.isEmpty() && rotate < queue.size()) {
        int want = queue.peek();
        int top = stack.peek();
        if (want == top) {
            queue.poll();
            stack.pop();
            rotate = 0;
        } else {
            queue.offer(queue.poll());          // 뒤로 보냄
            rotate++;
        }
    }
    return queue.size();
}
```

트레이드오프: 인덱스 배열을 거치지 않아 코드가 짧고, 샌드위치 스택이 진짜 `push`/`pop` 스택이라 문제의 그림과 일대일로 맞는다. 대신 **값이 중간에 바뀌는 문제(LC2073)에는 못 쓴다.** 값만 넣으면 누구인지 알 수 없기 때문이다. 이 문제는 값이 안 변해서 가능하다.
