---
title: "[프로그래머스-Lv1] K번째수 - 문제의 '번째'는 1부터, 배열은 0부터"
date: 2026-10-21 10:15:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/42748
- 파일 경로: `src/main/java/coding_test/정렬/Lv1_K번째수.java`
- 난이도: Lv.1

## 문제 설명

배열 `array`를 **i번째부터 j번째까지 자르고 정렬했을 때 k번째 수**를 구한다. `[i, j, k]`가 여러 개(`commands`) 주어지면 각각의 결과를 배열로 반환한다. 여기서 "번째"는 모두 **1부터** 센다.

```
array = [1,5,2,6,3,7,4], commands = [[2,5,3],[4,4,1],[1,7,3]] → [5, 6, 3]
```

## 시행착오

정렬 카테고리의 첫 문제였다. 정렬 자체는 `Arrays.sort` 한 줄이라 쉬웠고, 틀린 곳은 거의 다 **"1부터 세는 번째"와 "0부터 세는 인덱스"**의 어긋남이었다.

### 1차: `ArrayIndexOutOfBoundsException: Index 3 out of bounds for length 3`

```java
Arrays.sort(array);                                          // ① 원본 전체를 정렬
int[] temp = new int[commands[i][1] - commands[i][0]];       // ② 길이가 1 모자람
for (int j = commands[i][0]; j < commands[i][1]; j++) {      // ③ 1부터 세는 걸 그대로
    temp[idx] = commands[i][j];                              // ④ array가 아니라 commands에서 꺼냄
    idx++;
}
answer[i] = temp[commands[i][2]];                            // ⑤ k번째를 그대로 인덱스로
```

에러는 ④에서 났다. `commands[i]`는 `[i, j, k]` 세 칸짜리라 `j = 3`에서 터졌다. 나머지도 하나씩 틀렸다.

1. 원본 전체를 먼저 정렬하면 자를 구간 자체가 바뀐다. **자른 뒤에** 정렬해야 한다.
2. 2번째부터 5번째까지는 `5 - 2 = 3`개가 아니라 **4개**(`+ 1`)다.
3. "2번째"는 인덱스 1이다. 시작은 `commands[i][0] - 1`이다.
4. 값을 꺼낼 곳은 `array[j]`다.
5. "3번째"는 인덱스 2다.

### 2차: `- 1`이 괄호 밖에

```java
answer[i] = temp[commands[i][2]] - 1;    // 꺼낸 "값"에서 1을 뺌
```

1을 빼야 하는 건 **인덱스**인데 괄호 밖에 써서, 꺼낸 **값**에서 1을 뺐다. k가 마지막 칸이면 범위를 넘어 터졌다. `temp[commands[i][2] - 1]`로 고쳐 통과했다.

## 최종 코드

```java
public int[] solution(int[] array, int[][] commands) {
    int[] answer = new int[commands.length];
    for (int i = 0; i < commands.length; i++) {
        int[] temp = new int[commands[i][1] - commands[i][0] + 1];   // j - i + 1개
        int idx = 0;
        for (int j = commands[i][0] - 1; j < commands[i][1]; j++) {  // i번째(인덱스 i-1)부터
            temp[idx] = array[j];
            idx++;
        }
        Arrays.sort(temp);                                           // 자른 뒤에 정렬
        answer[i] = temp[commands[i][2] - 1];                        // k번째(인덱스 k-1)
    }
    return answer;
}
```

5개 테스트가 통과했고, 랜덤 30,000건에서도 정답과 일치했다.

## 추적표: `[2, 5, 3]`

| 단계 | 값 |
|---|---|
| 2번째 ~ 5번째 (인덱스 1 ~ 4) | `[5, 2, 6, 3]` |
| 정렬 | `[2, 3, 5, 6]` |
| 3번째 (인덱스 2) | **5** |

## 오늘 배운 내용

**문제의 "k번째"는 1부터, 배열 인덱스는 0부터다.** "i번째부터 j번째"를 인덱스로 바꾸면 `i - 1`부터 `j - 1`까지이고, 개수는 `j - i + 1`이다. `Arrays.copyOfRange(arr, i - 1, j)`는 이 변환을 한 번에 해 준다(끝 미포함이라 `j`를 그대로 쓴다).

**`- 1`은 어디에 붙는지가 중요하다.** `temp[k - 1]`(인덱스에서 뺌)과 `temp[k] - 1`(값에서 뺌)은 완전히 다르다.

## 오답노트

- **틀렸던 지점 1**: 원본 전체를 정렬. **왜**: "정렬한다"를 먼저 떠올렸다. **다음에**: 문제의 순서(자르기 → 정렬 → 꺼내기)를 그대로 따른다.
- **틀렸던 지점 2**: 길이 `j - i`. **왜**: 양 끝을 모두 포함한다는 걸 놓쳤다. **다음에**: 작은 예(2~5)를 손으로 세어 본다.
- **틀렸던 지점 3**: `commands[i][j]`에서 값을 꺼냄. **왜**: 반복 변수 이름 `j`와 `commands[i][1]`(j값)이 섞였다. **다음에**: 값을 꺼낼 배열 이름을 한 번 더 읽는다.
- **틀렸던 지점 4**: `- 1`을 괄호 밖에. **왜**: "1을 빼야 한다"까지만 생각하고 어디서 빼는지 안 봤다. **다음에**: 인덱스 보정은 `[ ]` 안에서 한다.

## AI라면 어떻게 풀었을까

새 배열을 만들고 `for`문으로 한 칸씩 옮기는 대신 **`Arrays.copyOfRange`로 한 번에 자른다.** `copyOfRange(배열, 시작, 끝)`은 시작 포함·끝 미포함(0부터)이라 "i번째부터 j번째"는 `(i - 1, j)`로 그대로 옮겨진다. 길이 계산과 `idx` 변수가 필요 없다.

```java
public int[] solutionAlt(int[] array, int[][] commands) {
    int[] answer = new int[commands.length];
    for (int c = 0; c < commands.length; c++) {
        int[] cmd = commands[c];
        int[] part = Arrays.copyOfRange(array, cmd[0] - 1, cmd[1]);   // i번째 ~ j번째
        Arrays.sort(part);
        answer[c] = part[cmd[2] - 1];                                // k번째
    }
    return answer;
}
```

트레이드오프: 이번에 틀린 부분(길이 `+1`, 시작 `-1`, 꺼낼 배열)이 대부분 사라진다. 대신 `copyOfRange`의 "끝 미포함" 규칙을 알아야 한다. 랜덤 30,000건에서 본인 풀이와 출력이 일치했다.
