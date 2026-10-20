---
title: "[프로그래머스-Lv1] 폰켓몬 - 값이 필요 없으면 HashMap이 아니라 HashSet"
date: 2026-10-20 09:40:00 +0900
categories: [알고리즘 연습, 해시]
tags: [java, programmers, hash, hashset]
---

> 해시 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-20-hash-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/1845
- 파일 경로: `src/main/java/coding_test/해시/Lv1_폰켓몬.java`
- 난이도: Lv.1

## 문제 설명

폰켓몬 N마리(종류 번호 배열 `nums`, N은 짝수) 중 **N/2마리**를 가져갈 수 있다. 가장 많은 **종류**를 고를 때 그 종류 수를 반환한다.

```
nums = [3,1,2,3]       → 2   (2마리를 고를 수 있고, 종류는 3가지 → 최대 2종)
nums = [3,3,3,2,2,4]   → 3   (3마리, 종류 3가지 → 3종)
nums = [3,3,3,2,2,2]   → 2   (3마리, 종류 2가지 → 2종)
```

## 시행착오

**틀린 곳 없이 한 번에 통과했다**(약 12분). 다만 직전 문제(완주하지 못한 선수)에서 익힌 `HashMap`을 그대로 썼다.

```java
Map<Integer, Integer> map = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    map.put(nums[i], 1);          // 값은 항상 1
}
for (Map.Entry<Integer, Integer> e : map.entrySet()) {
    if (answer >= nums.length / 2) break;
    else answer++;
}
```

풀고 나서 스스로 물었다. **"여기는 HashSet을 쓰는 게 더 나았을까? HashMap만 익숙해서."** 맞는 의문이었다. 맵의 값은 항상 1이라 쓰이지 않는다. 키만 필요하면 `HashSet`이다.

## 최종 코드

```java
public int solution(int[] nums) {
    int answer = 0;
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        map.put(nums[i], 1);                         // 같은 종류는 한 칸만 남는다
    }

    for (Map.Entry<Integer, Integer> e : map.entrySet()) {
        if (answer >= nums.length / 2) {
            break;                                   // N/2마리까지만 고를 수 있다
        } else {
            answer++;
        }
    }
    return answer;
}
```

8개 테스트가 모두 통과했고, 랜덤 30,000건에서도 정답과 일치했다.

## 추적표: `[3,3,3,2,2,4]`

| 단계 | 내용 |
|---|---|
| 종류 모으기 | {3, 2, 4} → 3종 |
| 고를 수 있는 수 | N/2 = 3 |
| 답 | min(3종, 3마리) = **3** |

## 오늘 배운 내용

**키마다 값이 필요하면 `HashMap`, 키(있다/없다)만 필요하면 `HashSet`이다.** 이 문제는 "어떤 종류가 있나"만 알면 되므로 `HashSet`이 맞다. 중복은 넣는 순간 자동으로 빠져서 `set.size()`가 곧 종류 수다.

**답이 "두 제한 중 작은 쪽"이면 `Math.min`이다.** 종류가 아무리 많아도 N/2마리까지만 가져갈 수 있고, 마리 수가 남아도 종류가 부족하면 그 이상은 안 된다.

## 오답노트

- **틀린 곳은 없었다.**
- **개선할 점**: 값이 항상 1인 `HashMap`은 `HashSet`으로 바꾼다. 마지막 반복은 `Math.min(종류 수, N/2)` 한 줄로 바뀐다.
- **좋았던 점**: 맞힌 뒤에도 "더 맞는 도구가 있었나?"를 스스로 물었다. 이 질문이 다음 문제에서 도구를 고르는 속도를 올린다.

## AI라면 어떻게 풀었을까

키(종류 번호)만 저장하는 `HashSet`을 쓴다. 중복은 자동으로 빠지므로 `set.size()`가 종류 수이고, 답은 종류 수와 N/2 중 작은 값이다.

```java
public int solutionAlt(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int n : nums) {
        set.add(n);                                  // 중복 종류는 자동으로 무시
    }
    return Math.min(set.size(), nums.length / 2);    // 종류 수와 N/2 중 작은 값
}
```

트레이드오프: 쓰지 않는 값을 저장하지 않고, 마지막 반복이 `Math.min` 한 줄로 바뀌어 의도("종류 수와 N/2 중 작은 쪽")가 바로 읽힌다. 기존 풀이와 시간 차이는 거의 없다(둘 다 `O(N)`). 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
