---
title: "[프로그래머스-Lv1] 완주하지 못한 선수 - 이름별 개수를 세고, 남은 한 명을 찾는다"
date: 2026-10-20 09:20:00 +0900
categories: [알고리즘 연습, 해시]
tags: [java, programmers, hash, hashmap]
---

> 해시 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-20-hash-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/42576
- 파일 경로: `src/main/java/coding_test/해시/Lv1_완주하지못한선수.java`
- 난이도: Lv.1

## 문제 설명

마라톤 참가자 명단(`participant`)과 완주자 명단(`completion`)이 주어진다. 완주하지 못한 **단 한 명**의 이름을 반환한다. 참가자는 최대 10만 명이고, **동명이인**이 있을 수 있다.

```
participant = ["mislav", "stanko", "mislav", "ana"]
completion  = ["stanko", "ana", "mislav"]
→ "mislav"   (참가자에 두 명, 완주자에 한 명)
```

## 시행착오

해시 카테고리의 첫 문제였다. 전날 저녁에 세팅받고 못 풀었다가 다음 날 아침에 다시 풀었다.

`HashMap`의 메서드 세 개(개수 늘리기, 줄이기, 훑기)를 먼저 받고 시작했고, 중간에 **"map을 순회하는 방법"**만 물었다. 순회 방법은 세 가지였다.

```java
for (String name : map.keySet()) { ... }                       // 키만
for (Map.Entry<String, Integer> e : map.entrySet()) { ... }    // 키와 값
for (int count : map.values()) { ... }                         // 값만 (이름을 모름)
```

"개수가 0이 아닌 **이름**"을 찾아야 하니 키를 알 수 있는 `entrySet()`을 골랐다. 로직 자체는 힌트 없이 처음 짠 대로 7개 테스트를 모두 통과했다.

## 최종 코드

```java
public String solution(String[] participant, String[] completion) {
    String answer = "";

    Map<String, Integer> map = new HashMap<>();
    for (int i = 0; i < participant.length; i++) {
        map.put(participant[i], map.getOrDefault(participant[i], 0) + 1);   // 이름별 +1
    }

    for (int i = 0; i < completion.length; i++) {
        map.put(completion[i], map.getOrDefault(completion[i], 0) - 1);     // 완주하면 -1
    }

    for (Map.Entry<String, Integer> e : map.entrySet()) {
        if (e.getValue() != 0) {
            answer = e.getKey();                                            // 남은 한 명
        }
    }
    return answer;
}
```

7개 테스트가 모두 통과했고, 랜덤 30,000건(동명이인 포함)과 10만 명 입력(24ms)도 통과했다.

## 추적표: 예시 3

| 단계 | 처리 | 맵 상태 |
|---|---|---|
| 참가자 | mislav, stanko, mislav, ana | {mislav=2, stanko=1, ana=1} |
| 완주자 | stanko, ana, mislav | {mislav=1, stanko=0, ana=0} |
| 훑기 | 0이 아닌 것 | **mislav** |

## 오늘 배운 내용

**"몇 번 나왔나"를 세는 문제는 `HashMap<이름, 개수>`다.** 동명이인처럼 같은 이름이 여러 번 나와도 개수로 관리하면 자연스럽게 처리된다. 정렬해서 하나씩 비교하는 방법도 있지만(`O(n log n)`), 해시는 넣고 빼는 데 `O(1)`이라 전체가 `O(n)`이다.

## 오답노트

- **틀린 곳은 없었다.**
- **확인할 점**: 마지막 반복에서 답을 찾아도 끝까지 돈다. 답은 한 명뿐이니 찾자마자 `return e.getKey();`로 끝내도 된다.
- **좋았던 점**: 개수 늘리기·줄이기를 `getOrDefault`로 처리해서, 완주자 이름이 맵에 없을 때의 `null` 문제(NPE)를 처음부터 피했다.

## AI라면 어떻게 풀었을까

개수를 다 센 뒤 전체를 훑어 0이 아닌 이름을 찾는 대신, **개수가 0이 되는 순간 그 이름을 맵에서 지운다.** 그러면 마지막에 맵에 남는 이름은 완주하지 못한 한 명뿐이라, 전체를 훑을 필요 없이 그 하나를 꺼내면 된다. `merge`는 `getOrDefault + put` 두 줄을 한 줄로 줄이고, 합친 결과값을 돌려준다.

```java
public String solutionAlt(String[] participant, String[] completion) {
    Map<String, Integer> map = new HashMap<>();
    for (String p : participant) {
        map.merge(p, 1, Integer::sum);          // 이름별 개수 +1
    }
    for (String c : completion) {
        if (map.merge(c, -1, Integer::sum) == 0) {
            map.remove(c);                      // 다 완주했으면 지운다
        }
    }
    return map.keySet().iterator().next();      // 남은 한 명
}
```

트레이드오프: 마지막에 맵을 훑지 않고, `merge`로 코드가 짧아진다. 대신 `merge`의 동작(없으면 넣고, 있으면 합치고, 결과를 돌려준다)과 `Integer::sum` 같은 메서드 참조를 알아야 읽힌다. 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
