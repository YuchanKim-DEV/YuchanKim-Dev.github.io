---
title: "[LeetCode-1909] 위반이 1번이어도, 지울 원소에 따라 결과가 갈린다"
date: 2026-09-22 10:40:00 +0900
categories: [알고리즘 연습, 배열]
tags: [java, leetcode, array]
---

- 문제 링크: https://leetcode.com/problems/remove-one-element-to-make-the-array-strictly-increasing/
- 파일 경로: `src/main/java/coding_test/배열_리스트/LC1909_RemoveOneElementToMakeTheArrayStrictlyIncreasing.java`
- 난이도: Easy

## 문제 설명

정수 배열 `nums`가 주어진다. 원소를 정확히 하나만 제거해서 엄격히 증가하는 배열로 만들 수 있으면 `true`, 없으면 `false`를 반환한다. 이미 엄격히 증가하는 배열이면 그것도 `true`다.

```
Input: nums = [2,3,1,2]
Output: false  // 어느 원소 하나를 제거해도 안 됨
```

## 시행착오

1752(Check if Array Is Sorted and Rotated)를 풀 때처럼 "엄격히 증가"가 깨지는 지점(`nums[i-1] >= nums[i]`)이 몇 번 나오는지 세는 것으로 시작했다. 처음엔 1752와 똑같이 "위반이 1번이면 무조건 `true`"라고 생각했는데, `[2,3,1,2]`(위반이 인덱스 2에서 딱 1번, `3 >= 1`)에서 걸렸다 — 실제로는 `false`가 나와야 하는데 위반 횟수만 보면 `true`가 나와버렸다.

이유를 다시 생각해보니, 1752는 "회전된 배열인가"만 물어서 위반 지점 자체를 신경 쓸 필요가 없었지만, 이 문제는 "그 지점에서 **어느 원소를 지우느냐**"에 따라 결과가 갈린다. `[2,3,1,2]`에서 위반 지점(인덱스 2, 값 1) 앞의 3을 지우면 `[2,1,2]`(여전히 `2>=1`이라 깨짐), 뒤의 1을 지우면 `[2,3,2]`(여전히 `3>=2`라 깨짐) — 둘 다 안 되니까 `false`다.

그래서 위반이 정확히 1번일 때는, 그 위반이 일어난 인덱스를 저장해뒀다가, "앞 원소를 지웠을 때"와 "뒤 원소를 지웠을 때" 각각 그 경계가 이어지는지 직접 확인하는 방식으로 바꿨다. 처음엔 이걸 반복문으로 전체 배열을 다시 훑어서 확인하려고 했는데, 위반이 그 지점 하나뿐이라는 게 이미 보장돼 있으니 반복문 없이 경계 두 지점(`nums[i-2]`와 `nums[i]`, 또는 `nums[i-1]`과 `nums[i+1]`)만 비교하면 충분하다는 걸 깨달았다.

## 최종 코드

```java
public boolean canBeIncreasing(int[] nums) {
    int counter = 0;
    int invalidIdx = 0;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i - 1] >= nums[i]) {
            counter++;
            invalidIdx = i;
        }
    }

    if (counter == 1) {
        if (invalidIdx - 1 == 0 || invalidIdx == nums.length - 1) {
            return true;
        } else if (nums[invalidIdx - 2] < nums[invalidIdx] || nums[invalidIdx - 1] < nums[invalidIdx + 1]) {
            return true;
        } else {
            return false;
        }
    }

    return !(counter >= 2);
}
```

## 오늘 배운 내용

**"위반이 몇 번 나오는가"를 세는 것만으로는 부족한 문제가 있다 — 그 위반을 "어떻게 해소하느냐"까지 확인해야 하는 경우가 있다.** 1752는 위반 횟수 자체가 답이었지만, 이 문제는 위반 횟수(1번 이하)는 필요조건일 뿐이고, 그 지점에서 실제로 원소 하나를 지워서 경계가 이어지는지가 충분조건이다. 비슷해 보이는 두 문제(1752와 1909)를 같은 틀로 풀려다가 한 번 걸려봐야, 그 틀이 어디까지 통하고 어디서부터 안 통하는지가 명확해진다.

## 오답노트

- **틀렸던 패턴**: 1752와 똑같이 "위반이 1번이면 무조건 true"로 판단.
- **왜 틀렸나**: 1752는 "회전된 배열인가"만 확인하면 됐지만, 이 문제는 "그 지점에서 원소 하나를 지웠을 때 실제로 이어지는가"까지 확인해야 하는 추가 조건이 있었다.
- **고친 패턴**: 위반 인덱스를 저장해두고, 앞/뒤 중 하나를 지웠을 때 경계(`nums[i-2]`↔`nums[i]` 또는 `nums[i-1]`↔`nums[i+1]`)가 이어지는지 직접 확인.
- **다음에 떠올릴 시점**: 이전에 풀었던 문제와 겉모습이 비슷해 보여도(둘 다 "위반 횟수 세기"), "위반 횟수만으로 충분한가, 아니면 위반을 해소하는 방법까지 확인해야 하는가"를 먼저 구분한다.

## AI라면 어떻게 풀었을까

지금 코드는 위반 지점을 하나만 저장해서(`invalidIdx`) 판단하는데, 아예 "두 번째 위반이 나오는 순간 바로 false"로 조기 종료하면서 동시에 앞/뒤 제거를 그 자리에서 바로 시도해보는 한 번의 순회로도 풀 수 있다.

```java
public boolean canBeIncreasing(int[] nums) {
    boolean removed = false;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i - 1] < nums[i]) {
            continue;
        }
        if (removed) {
            return false;
        }
        removed = true;
        // 앞 원소를 지워도 되고(i==1이거나 nums[i-2] < nums[i]),
        // 뒤 원소를 지워도 된다(nums[i-1] < nums[i+1]이거나 i==마지막)면 계속 진행
        if (i > 1 && nums[i - 2] >= nums[i] && (i == nums.length - 1 || nums[i - 1] >= nums[i + 1])) {
            return false;
        }
    }
    return true;
}
```

지금 코드가 "일단 위반을 다 센 다음, 끝나고 나서 한 번 더 검사"하는 2단계 구조라면, 이 버전은 "위반을 만나는 그 순간 바로 판단"하는 1단계 구조다. 로직 자체는 똑같은데, 배열을 한 번만 훑고 끝낸다는 점이 다르다. 다만 조건식이 한 줄에 몰려 있어서 가독성은 지금 코드(위반 인덱스를 저장해두고 나중에 따로 검사)가 더 낫다 — 항상 "짧은 코드가 좋은 코드"는 아니라는 것도 함께 기억해두면 좋다.
