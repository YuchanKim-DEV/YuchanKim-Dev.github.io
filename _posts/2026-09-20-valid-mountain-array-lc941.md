---
title: "[LeetCode-941] 한 번 true가 된 플래그는 다시 false가 되지 않는다"
date: 2026-09-20 21:00:00 +0900
categories: [알고리즘 연습, 배열]
tags: [java, leetcode, array]
---

- 문제 링크: https://leetcode.com/problems/valid-mountain-array/
- 파일 경로: `src/main/java/coding_test/배열_리스트/LC0941_ValidMountainArray.java`
- 난이도: Easy

## 문제 설명

정수 배열 `arr`이 주어진다. 이 배열이 "산 모양(mountain array)"이면 `true`, 아니면 `false`를 반환한다. 산 모양이란 길이가 3 이상이고, 어떤 정상 지점(맨 앞/맨 뒤는 될 수 없음)까지는 엄격히 증가하고 그 뒤로는 끝까지 엄격히 감소하는 배열을 말한다.

```
Input: arr = [0,3,2,1]
Output: true

Input: arr = [1,2,3,4]
Output: false  // 계속 오르기만 해서 정상이 맨 끝이 됨
```

## 시행착오

이전까지 풀었던 485/1550/674/896은 전부 "카운터 하나 또는 플래그 두 개"로 되는 문제였는데, 이 문제는 "오르막 상태", "내리막 상태", "이미 정상을 지났는가"를 동시에 추적해야 해서 `up`, `down`, `transition` 세 개의 불리언 플래그로 접근했다.

몇 단계를 거쳐서야 통과했다.

1. **처음 시작이 내리막인 경우** (`{2,0,2}`처럼 산이 아니라 골짜기 모양) — 처음엔 이 케이스를 놓쳤는데, "정상은 맨 앞이 될 수 없다"는 조건을 다시 읽고 나서, 첫 비교부터 내리막이면 곧바로 `false`를 반환하는 조건을 추가했다.
2. **루프를 다 통과했을 때의 기본 반환값** — `return false;`로 잘못 써놔서 정상적인 산 모양도 전부 `false`가 나온 적이 있었다. 루프를 무사히 다 돌았다는 건 실패 조건에 안 걸렸다는 뜻이므로, 최종 판정은 `transition && up && down`(오르막도 있었고 내리막도 있었는지)으로 걸어야 했다.
3. **오르막-내리막-오르막이 반복되는 경우** (`{0,1,2,1,2}` 같은, 제출 후 LeetCode가 알려준 반례) — `true`가 나와야 정상인데 여기선 `true`가 잘못 나왔다. 원인은 "이미 내리막이 시작된 뒤 다시 오르막이 나오면 무효 처리"하려던 조건이 `!up && down`이었는데, `up`은 처음 오르막에서 한 번 `true`가 된 뒤로 다시 `false`로 바뀌는 지점이 코드 어디에도 없었다. 그래서 `!up`이 이 시점엔 항상 거짓이라 그 `return false` 조건 자체가 한 번도 실행되지 않는 죽은 코드였다. `transition`이 이미 `true`인 상태(=이미 내리막이 시작된 상태)에서는 `up`/`down` 플래그를 더 따질 필요 없이, `prev < arr[i]`라는 사실 하나만으로 바로 무효라고 판단하면 된다는 걸 깨닫고 조건을 단순화했다.

## 최종 코드

```java
public boolean validMountainArray(int[] arr) {
    if (arr.length <= 2) {
        return false;
    }

    boolean up = false;
    boolean down = false;
    boolean transition = false;
    for (int i = 1; i < arr.length; i++) {
        int prev = arr[i - 1];
        if (!transition) {
            if (prev > arr[i] && up && !down) {
                down = true;
                transition = true;
            } else if (prev < arr[i] && !up && !down) {
                up = true;
            } else if (prev > arr[i] && !up && !down) {
                return false;
            }
        } else {
            if (prev > arr[i] && up && !down) {
                down = true;
            } else if (prev < arr[i]) {
                return false;
            }
        }

        if (prev == arr[i]) {
            return false;
        }
    }

    return transition && up && down;
}
```

## 오늘 배운 내용

**한 번 `true`로 바뀐 플래그가 그 뒤로 다시 `false`가 되지 않는다면, "이 플래그가 `false`일 때"라는 조건은 그 시점 이후로 영원히 실행되지 않는 죽은 코드가 될 수 있다.** `up`이 처음 오르막에서 한 번 켜진 뒤로는 끝까지 계속 `true`였는데도, "재차 오르막이 나오면 무효" 판정 조건에 `!up`을 그대로 남겨둬서 그 조건이 절대 참이 될 수 없었다. 조건문에 변수를 쓸 때는 "이 변수가 지금 실행 흐름상 실제로 그 값을 가질 수 있는가"까지 확인해야 한다.

## 오답노트

- **틀렸던 패턴 1**: 첫 비교부터 내리막인 경우(`{2,0,2}`)를 무효로 막는 조건이 없었음.
- **왜 틀렸나**: "정상은 맨 앞이 될 수 없다"는 제약을 오르막 쪽에서만 생각하고, 내리막이 배열 맨 앞부터 시작하는 경우는 따로 안 막아뒀다.
- **틀렸던 패턴 2**: 루프 종료 후 기본 반환값을 `false`로 잘못 씀.
- **왜 틀렸나**: 실패 조건에 걸리지 않고 루프를 끝까지 통과했다는 것 자체가 이미 유효할 가능성이 높다는 걸 놓치고, 최종 판정 없이 무조건 `false`를 반환해버렸다.
- **틀렸던 패턴 3**: 내리막 이후 재차 오르막을 막는 조건이 `!up && down`이었는데, `up`이 이미 영구히 `true`라 이 조건이 죽은 코드였음.
- **왜 틀렸나**: 플래그가 그 시점에 실제로 가질 수 있는 값을 확인하지 않고, "이 상태를 막고 싶다"는 의도만으로 조건을 만들었다.
- **고친 패턴**: `transition`이 `true`인 상태에서는 `prev < arr[i]`만으로 바로 무효 처리.
- **다음에 떠올릴 시점**: 여러 불리언 플래그를 쓰는 코드에서 버그가 안 잡히면, 각 조건문이 "지금 이 시점에 그 변수가 실제로 그 값일 수 있는가"부터 손으로 확인한다. 절대 안 바뀌는 값에 기대는 조건은 죽은 코드일 가능성이 높다.
