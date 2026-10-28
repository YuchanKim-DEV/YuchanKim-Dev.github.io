---
title: "[프로그래머스-Lv1] 예산 - 싼 것부터 담는다"
date: 2026-10-28 09:21:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, greedy]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/12982
- 파일 경로: `src/main/java/coding_test/정렬/Lv1_예산.java`
- 난이도: Lv.1

## 문제 설명

부서마다 신청한 금액 `d`가 있고, 전체 예산은 `budget`이다. 신청 금액을 깎아 줄 수 없고 **전액을 줘야** 한다. 이 조건에서 **최대 몇 개 부서**를 지원할 수 있는지 구한다.

```
d = [1,3,2,5,4], budget = 9   → 3
d = [2,2,3,3],   budget = 10  → 4
```

## 시행착오

**힌트 없이 한 번에 통과했다.** 지난주에 "언제쯤 힌트 없이 풀 수 있을까"를 고민했는데, 이번에는 "첫 15분은 힌트 없이" 규칙으로 시작해서 끝까지 혼자 풀었다.

접근은 바로 떠올랐다. 부서 수를 최대로 하려면 **신청 금액이 작은 부서부터** 지원해야 한다. 그래서 오름차순으로 정렬하고 앞에서부터 더해 나가다가, 예산을 넘는 순간 멈춘다.

## 최종 코드

```java
public int solution(int[] d, int budget) {
    int answer = 0;

    Arrays.sort(d);                          // 싼 부서부터
    int budgetCnt = 0;                       // 지금까지 쓴 돈
    int counter = 0;                         // 지원한 부서 수
    for (int i = 0; i < d.length; i++) {
        budgetCnt += d[i];
        if (budgetCnt > budget) {            // 넘으면 이 부서는 못 줌
            answer = counter;
            break;
        } else if (budgetCnt == budget) {    // 딱 맞으면 이 부서까지 주고 끝
            counter++;
            answer = counter;
            break;
        } else {
            counter++;
        }
    }

    answer = counter;
    return answer;
}
```

## 추적표: `[1,3,2,5,4], budget = 9`

정렬 후 `[1, 2, 3, 4, 5]`

| i | d[i] | budgetCnt | 비교 | counter |
|---|---|---|---|---|
| 0 | 1 | 1 | < 9 | 1 |
| 1 | 2 | 3 | < 9 | 2 |
| 2 | 3 | 6 | < 9 | 3 |
| 3 | 4 | 10 | **> 9 → 멈춤** | 3 |

## 오늘 배운 내용

**"최대 몇 개"를 묻는 문제에서 비용이 있으면 작은 것부터 담는다.** 큰 것을 하나 담을 돈이면 작은 것을 여러 개 담을 수 있다. 정렬 한 번으로 "어떤 것부터 볼지"가 정해진다. 어제 최솟값 만들기에 이어 정렬로 선택 순서를 정하는(그리디) 문제였다.

## 오답노트

- **틀린 곳은 없었다.** 다듬을 점이 하나 있다. 마지막 줄 `answer = counter`가 위에서 대입한 `answer`를 모두 덮어쓴다. 그래서 `if`와 `else if` 안의 `answer = counter`는 하는 일이 없다. **다음에**: 반환 직전에 같은 값을 다시 대입하고 있다면, 위쪽 대입이 정말 필요한지 본다.

## AI라면 어떻게 풀었을까

흐름은 같고 분기를 하나로 줄인다. "이 부서를 지원하면 예산을 넘는가?"만 보고, 넘으면 멈춘다. 넘지 않으면 예산에서 빼고 센다.

```java
public int solutionAlt(int[] d, int budget) {
    Arrays.sort(d);                          // 싼 부서부터
    int count = 0;
    for (int cost : d) {
        if (cost > budget) {
            break;                           // 이 부서부터는 못 줌
        }
        budget -= cost;
        count++;
    }
    return count;
}
```

트레이드오프는 없다. "딱 맞을 때"를 따로 다루지 않아도 되고 변수도 하나 줄어든다. 랜덤 20,000건에서 두 풀이의 결과가 같았다.
