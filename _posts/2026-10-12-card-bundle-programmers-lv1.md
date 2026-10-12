---
title: "[프로그래머스-Lv1] 카드 뭉치 - 큐의 맨 앞만 볼 수 있다는 게 규칙이다"
date: 2026-10-12 09:33:00 +0900
categories: [알고리즘 연습, 스택_큐]
tags: [java, programmers, queue, simulation]
---

> 스택·큐 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-09-23-stack-queue-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/159994
- 파일 경로: `src/main/java/coding_test/스택_큐/Lv1_카드뭉치.java`
- 난이도: Lv.1

## 문제 설명

영어 단어가 적힌 카드 뭉치 두 개(`cards1`, `cards2`)가 있다. 각 뭉치에서는 **맨 앞 카드부터 순서대로 한 장씩만** 쓸 수 있고, 한 번 쓴 카드는 다시 못 쓰며, 건너뛸 수도 없다. 원하는 단어 배열 `goal`을 이 규칙으로 만들 수 있으면 `"Yes"`, 없으면 `"No"`를 반환한다.

```
cards1 = ["i","drink","water"], cards2 = ["want","to"]
goal   = ["i","want","to","drink","water"]   → "Yes"
```

## 시행착오

처음엔 "카드1에서 먼저 하나 뽑고 카드2를 다 뽑는 게 규칙인가?"라고 읽었다. 아니다. **어느 뭉치에서 뽑을지는 `goal`이 정한다.** `goal`을 앞에서부터 한 단어씩 보면서 "카드1 맨 앞인가, 카드2 맨 앞인가?"를 묻는 것이 전부다.

### 1차: 무한 루프

```java
while (!gQueue.isEmpty()) {
    if (c1Queue.peek().equals(gQueue.peek())) { ... gQueue.poll(); }
    else if (c2Queue.peek().equals(gQueue.peek())) { ... gQueue.poll(); }
    else if (c1Queue.isEmpty() && !c2Queue.isEmpty()) { ... }
    ...
}
```

어느 조건도 안 맞으면 `gQueue`를 줄이는 `poll()`이 실행되지 않아 같은 검사를 영원히 반복했다. 또 카드 큐는 한 번도 `poll()`하지 않았고, 비어 있는 큐에 `peek().equals(...)`를 해서 NPE 위험도 있었다.

### 2차: 인덱스 오류

```java
for (int i = 0; i < cards1.length; i++) {
    c1Queue.offer(cards1[1]);      // i가 아니라 1
}
```

카드가 한 장뿐인 테스트에서 `ArrayIndexOutOfBoundsException`이 났다. 바로 위 `goal` 반복문은 `goal[i]`로 제대로 썼는데, 복사해서 붙이면서 `i`를 `1`로 잘못 바꿨다.

### 3차: 전부 `No`

```java
if (ansSB == goalSB) { answer = "Yes"; } else { answer = "No"; }
```

`StringBuilder` 두 개를 `==`로 비교하면 내용이 아니라 **같은 객체인지**를 묻는다. 둘은 다른 객체라서 항상 `false`, 그래서 항상 `No`였다. 게다가 두 빌더에는 똑같은 단어가 똑같이 append되고 있어서 `equals`로 바꿔도 항상 `Yes`였을 것이다. "goal을 끝까지 만들었는가"는 빌더가 아니라 **반복이 끝난 뒤 `gQueue`가 비었는가**로 알 수 있었다.

## 최종 코드

```java
public String solution(String[] cards1, String[] cards2, String[] goal) {
    Queue<String> gQueue = new ArrayDeque<>();
    Queue<String> c1Queue = new ArrayDeque<>();
    Queue<String> c2Queue = new ArrayDeque<>();

    for (int i = 0; i < goal.length; i++) gQueue.offer(goal[i]);
    for (int i = 0; i < cards1.length; i++) c1Queue.offer(cards1[i]);
    for (int i = 0; i < cards2.length; i++) c2Queue.offer(cards2[i]);

    while (!gQueue.isEmpty()) {
        if (!c1Queue.isEmpty() && c1Queue.peek().equals(gQueue.peek())) {
            gQueue.poll();
            c1Queue.poll();                 // 카드를 썼으면 그 뭉치도 한 장 줄인다
        } else if (!c2Queue.isEmpty() && c2Queue.peek().equals(gQueue.peek())) {
            gQueue.poll();
            c2Queue.poll();
        } else {
            break;                          // 어느 맨 앞과도 안 맞음
        }
    }

    return gQueue.isEmpty() ? "Yes" : "No";
}
```

(제출한 코드에는 쓰이지 않는 `ansSB`가 남아 있어서 지워도 된다.)

## 추적표: 예시 1

| goal 단어 | 카드1 맨 앞 | 카드2 맨 앞 | 사용 | 남은 goal |
|---|---|---|---|---|
| i | i | want | 카드1 | want,to,drink,water |
| want | drink | want | 카드2 | to,drink,water |
| to | drink | to | 카드2 | drink,water |
| drink | drink | (빔) | 카드1 | water |
| water | water | (빔) | 카드1 | (빔) → **Yes** |

## 오늘 배운 내용

**"안 맞으면 끝낸다"를 코드로 쓰지 않으면 루프가 안 끝난다.** 반복이 `poll()`로 줄어드는 구조라면, 모든 분기에서 줄어들든가 그렇지 않은 분기에서 `break`/`return`으로 빠져나가야 한다. 그리고 판단은 **반복이 끝난 뒤의 상태**(`gQueue`가 비었나)로 한다. 중간 기록을 따로 모아 비교할 필요가 없다.

## 오답노트

- **틀렸던 지점 1**: 어느 분기도 안 맞는 경우를 처리하지 않아 무한 루프. **왜**: "맞는 경우"만 생각하고 "맞지 않는 경우"를 안 썼다. **다음에**: `while` 안의 분기를 쓴 뒤 "이 중 어느 것도 아니면?"을 꼭 묻는다.
- **틀렸던 지점 2**: `cards1[i]`를 `cards1[1]`로 씀. **왜**: 위 줄을 복사해 붙이고 바꾸다 `i`를 지웠다. **다음에**: 복사한 반복문은 변수 이름을 한 번 더 읽는다.
- **틀렸던 지점 3**: `StringBuilder`를 `==`로 비교. **왜**: 문자열처럼 `==`가 내용을 비교한다고 느꼈다. **다음에**: 객체 비교는 `equals`, 그리고 애초에 비교가 필요한지부터 의심한다.
- **틀렸던 지점 4**: 카드를 쓰고도 그 카드 큐를 `poll()`하지 않음. **왜**: `goal`을 줄이는 데만 신경 썼다. **다음에**: "무엇을 소비하면 무엇이 줄어드는가"를 한 줄로 쓴다.
- **좋았던 점**: 접근(큐 두 개 + `peek`로 맨 앞 비교)은 처음부터 스스로 정했고, 빈 큐 확인(`!isEmpty() &&`)도 힌트 직후 바로 넣었다.

## 꿀팁

비교 대상이 `null`일 수 있으면 **`null`이 아님이 확실한 쪽을 왼쪽에** 쓴다. `word.equals(q.peek())`는 큐가 비어 `peek()`이 `null`이어도 `false`만 돌려주고 NPE가 나지 않는다. 반대로 `q.peek().equals(word)`는 큐가 비면 NPE다.

## AI라면 어떻게 풀었을까

`goal`을 큐에 복사하지 않고 **그대로 앞에서부터 읽으면서** 두 카드 큐의 맨 앞과 비교한다. 단어 하나가 어느 쪽에도 안 맞으면 그 자리에서 바로 `"No"`를 반환하고, 끝까지 통과하면 `"Yes"`다. 위 꿀팁의 `word.equals(q.peek())` 덕분에 `isEmpty()` 확인도 필요 없다.

```java
public String solutionAlt(String[] cards1, String[] cards2, String[] goal) {
    Queue<String> q1 = new ArrayDeque<>(Arrays.asList(cards1));
    Queue<String> q2 = new ArrayDeque<>(Arrays.asList(cards2));

    for (String word : goal) {
        if (word.equals(q1.peek())) {
            q1.poll();
        } else if (word.equals(q2.peek())) {
            q2.poll();
        } else {
            return "No";                // 어느 맨 앞과도 안 맞음
        }
    }
    return "Yes";
}
```

트레이드오프: `goal` 큐, `StringBuilder`, `break` 뒤의 판정이 사라져서 코드가 절반으로 줄고, 무한 루프나 `==` 같은 실수가 구조적으로 생길 수 없다. 대신 "`peek()`이 `null`일 수 있다"는 점을 `equals`의 왼쪽 오른쪽 순서로 암묵적으로 처리하기 때문에, 이 요령을 모르면 읽기에 이상해 보일 수 있다.
