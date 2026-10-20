---
title: "[프로그래머스-Lv2] 의상 - 종류마다 '안 입는다'를 하나 더해서 곱한다"
date: 2026-10-20 09:57:00 +0900
categories: [알고리즘 연습, 해시]
tags: [java, programmers, hash, hashmap, combination]
---

> 해시 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-20-hash-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/42578
- 파일 경로: `src/main/java/coding_test/해시/Lv2_의상.java`
- 난이도: Lv.2

## 문제 설명

옷마다 `[이름, 종류]`가 주어진다. **종류별로 최대 하나**만 입을 수 있고, **최소 한 개는** 입는다. 서로 다른 조합의 수를 구한다.

```
[["yellow_hat","headgear"],["blue_sunglasses","eyewear"],["green_turban","headgear"]]  → 5
[["crow_mask","face"],["blue_sunglasses","face"],["smoky_makeup","face"]]              → 3
```

## 시행착오

해시 부분(종류 → 개수)은 바로 짰다. 완주하지 못한 선수에서 쓴 `getOrDefault + 1` 그대로다. 막힌 건 **개수로 조합 수를 계산하는 식**이었다.

### 1차: `(개수끼리 곱) + 종류 수`

```java
for (...) answer *= e.getValue();
answer += map.size();
```

예시 1(headgear 2개, eyewear 1개)이 `2 × 1 + 2 = 4`로 나왔다(정답 5). 이 식은 "두 종류를 다 입는 경우(2×1)"와 "한 종류만 입는 경우"를 섞어서 세는데, 한 종류만 입는 경우를 **종류 수(2)**로 셌다. 실제로는 그 종류의 **옷 개수만큼**(2 + 1 = 3) 있다. 그래서 종류가 세 개 이상이면 더 크게 틀렸다(Edge 3: 7이어야 하는데 4).

맞는 식은 이렇다.

> 종류마다 "**안 입는다**"는 선택지를 하나 더한다 → `(개수 + 1)`
> 모두 곱하고, **전부 안 입는 경우 1가지**를 뺀다 → `곱 - 1`

- headgear: yellow_hat, green_turban, **안 씀** → 3가지
- eyewear: blue_sunglasses, **안 씀** → 2가지
- `3 × 2 - 1 = 5`

### 2차: `-1`을 반복 안에 넣으려 함

"그러면 `answer *= (e.getValue() + 1) - 1`이겠네"라고 했는데, 그러면 `(개수 + 1) - 1 = 개수`라서 1차와 같아진다. **`+1`은 종류마다(반복 안), `-1`은 전체에서 한 번(반복 밖)**이다.

## 최종 코드

```java
public int solution(String[][] clothes) {
    int answer = 1;
    Map<String, Integer> map = new HashMap<>();
    for (int i = 0; i < clothes.length; i++) {
        map.put(clothes[i][1], map.getOrDefault(clothes[i][1], 0) + 1);   // 종류 → 개수
    }

    if (map.size() == 1) {
        Map.Entry<String, Integer> e = map.entrySet().iterator().next();
        answer = e.getValue();
    } else {
        for (Map.Entry<String, Integer> e : map.entrySet()) {
            answer *= e.getValue() + 1;          // 종류마다 "안 입는다" 포함
        }
        answer -= 1;                             // 전부 안 입는 경우 제외
    }
    return answer;
}
```

7개 테스트(의상 30개·30종류 → 1,073,741,823 포함)가 모두 통과했고, 랜덤 30,000건에서도 정답과 일치했다.

## 추적표: 예시 1

| 종류 | 개수 | 선택지(+1) |
|---|---|---|
| headgear | 2 | 3 |
| eyewear | 1 | 2 |
| 곱 | | 6 |
| -1 (전부 안 입음) | | **5** |

## 오늘 배운 내용

**"각 그룹에서 최대 하나씩 고르는 경우의 수"는 `(그룹 크기 + 1)`을 모두 곱하고 1을 뺀다.** `+1`은 "그 그룹에서 아무것도 안 고른다", `-1`은 "모든 그룹에서 아무것도 안 고른다"(문제에서 금지)다. 해시는 그룹 크기를 세는 도구이고, 답은 이 곱셈 공식에서 나온다.

## 오답노트

- **틀렸던 지점 1**: `곱 + 종류 수`. **왜**: 경우를 "둘 다 입음 / 하나만 입음"으로 나눠 세려다 하나만 입는 경우를 잘못 셌다. **다음에**: 경우를 나눠 세기보다 "각 그룹의 선택지 수"를 곱한다.
- **틀렸던 지점 2**: `-1`을 반복 안에 넣으려 함. **왜**: `+1`과 `-1`이 둘 다 같은 줄에 있다고 생각했다. **다음에**: 무엇이 "그룹마다"이고 무엇이 "전체에서 한 번"인지 구분한다.
- **개선할 점**: `map.size() == 1` 분기는 없어도 된다. 종류가 하나면 `(개수 + 1) - 1 = 개수`로 같은 식에서 나온다.

## AI라면 어떻게 풀었을까

같은 공식을 스트림으로 한 번에 쓴다. `groupingBy(종류, counting())`가 내부적으로 `HashMap<종류, 개수>`를 만들고, `reduce`로 `(개수 + 1)`을 곱해 나간다. 종류가 하나뿐이어도 같은 식이라 특수 분기가 없다.

```java
public int solutionAlt(String[][] clothes) {
    return Arrays.stream(clothes)
            .collect(Collectors.groupingBy(c -> c[1], Collectors.counting()))   // 종류 → 개수
            .values().stream()
            .reduce(1L, (acc, cnt) -> acc * (cnt + 1))                         // (개수+1) 곱
            .intValue() - 1;                                                   // 전부 안 입는 경우 제외
}
```

트레이드오프: 한 줄 흐름으로 "그룹 → 개수 → 곱 → -1"이 그대로 읽히고, `long`으로 곱해서 중간 넘침 걱정이 적다. 대신 `groupingBy`, `counting`, `reduce`를 알아야 읽히고, 디버깅할 때 중간값을 보기 어렵다. 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
