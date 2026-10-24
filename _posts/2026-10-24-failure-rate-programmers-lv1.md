---
title: "[프로그래머스-Lv1] 실패율 - double 정렬은 Double.compare, 부호만 맞으면 된다"
date: 2026-10-24 17:06:00 +0900
categories: [알고리즘 연습, 정렬]
tags: [java, programmers, sort, comparator, kakao]
---

> 정렬 개념과 자주 쓰는 메서드는 [카테고리 총정리 글]({% post_url 2026-10-21-sort-overview %})에서 다룬다.

- 문제 링크: https://school.programmers.co.kr/learn/courses/30/lessons/42889
- 파일 경로: `src/main/java/coding_test/정렬/Lv1_실패율.java`
- 난이도: Lv.1 (2019 KAKAO BLIND RECRUITMENT)

## 문제 설명

`stages`에는 사용자마다 **지금 멈춰 있는 스테이지 번호**가 들어 있다(`N + 1`은 전부 클리어한 사람). 스테이지별 실패율을 구해 **실패율이 높은 순서로**, 같으면 **번호가 작은 순서로** 스테이지 번호를 반환한다.

- 실패율 = 그 스테이지에 멈춘 사람 수 / 그 스테이지에 도달한 사람 수
- 도달한 사람이 없으면 실패율은 0

```
N = 5, stages = [2,1,2,6,2,4,3,3] → [3,4,2,1,5]
N = 4, stages = [4,4,4,4,4]       → [4,1,2,3]
```

## 시행착오

어제 LC1636에 이어 2기준 정렬이었다. 정렬 번호 배열 만들기, `Integer[]` ↔ `int[]` 변환은 바로 됐고, 막힌 곳은 **`double` 비교**와 **사람 수 세기**, 그리고 또 **`? :` 한쪽에 값 하나만 반환**이었다.

### 1차: 컴파일 에러 — 타입 두 개

```java
int[] answer = new int[N];
...
Arrays.sort(answer, (a, b) -> map.get(a) != map.get(b) ? map.get(b) - map.get(a) : a - b);
```

1. `answer`가 `int[]`라 Comparator를 쓸 수 없다(어제와 같은 에러). `Integer[]`로 정렬하고 마지막에 `int[]`로 옮긴다.
2. 실패율은 `double`이라 `map.get(b) - map.get(a)`도 `double`이다. Comparator는 `int`를 돌려줘야 한다. `(int)`로 바꾸면 0.2 같은 차이가 0이 돼서 "같다"로 처리되므로 안 된다. **`double`끼리는 `Double.compare(x, y)`를 쓴다.**
3. `Double`끼리 `!=`는 값이 같아도 거의 항상 "다르다"로 나온다(객체 비교). `Double.compare(...) == 0`으로 같은지 본다.

### 2차: 결과가 전부 `[1, 2, 3, ...]`

```java
for (int j = 0; j < stages.length; j++) {
    if (stages[j] >= i) {
        numer++;                  // 도달한 사람
    } else if (stages[j] > i) {   // ① 절대 들어올 수 없음
        demon++;
    }
    demon = numer - demon;        // ② 반복할 때마다 덮어씀
}
...
Arrays.sort(answer, (a, b) -> Double.compare(b, a) == 0 ? b - a : a - b);   // ③
```

1. `stages[j] > i`인 사람은 이미 `>= i`에 걸린다. `else if`에는 아무도 안 들어온다.
2. 멈춘 사람은 **`stages[j] == i`**인 사람이다. 따로 세면 되고, 뺄셈은 필요 없다.
3. `Double.compare(b, a)`는 실패율이 아니라 **스테이지 번호**를 비교했다. `map.get(...)`이 빠졌다. 또 `== 0`(같을 때)이면 번호가 **작은** 게 앞이어야 하는데 `b - a`로 반대였다.

### 3차: 3개 틀림

사람 수 세기를 `== i`로 고치고, 도달한 사람이 0이면 실패율을 0으로 두는 처리(`0 / 0` 방지)도 넣었다.

```java
Double.compare(map.get(b), map.get(a)) == 0 ? a - b : b - a
//                                       ✓ 같으면 번호 작은 게 앞
//                                               ✗ 다르면 번호 큰 게 앞 (실패율 안 봄)
```

`:` 뒤는 **실패율이 다를 때**인데 번호를 비교했다.

### 4차: 다시 `[1, 2, 3, ...]`

```java
: Double.compare(map.get(b), map.get(a)) != 0 ? b : a
```

`b`나 `a`(번호 하나)를 돌려줬다. 번호는 항상 양수라 "b가 앞"으로만 읽혀서 정렬이 깨진다. 어제 `? map.get(a)`와 같은 실수다. 게다가 `:` 뒤로 왔다는 것 자체가 이미 "다르다"는 뜻이라 조건을 또 쓸 필요가 없다.

### 5차: "0이나 -1만 나오는데 어떻게?"

`Double.compare(map.get(b), map.get(a))` 하나만 넣으라는 말에 "이건 0이나 -1만 나오는데 정렬이 되나?"가 궁금했다. **Comparator는 크기가 아니라 부호만 본다.** -1이든 -100이든 "a가 앞"이다. 그리고 `Double.compare`는 b 쪽이 작으면 1도 돌려준다. 넣고 돌리자 9개 모두 통과했다.

## 최종 코드

```java
public int[] solution(int N, int[] stages) {
    HashMap<Integer, Double> map = new HashMap<>();
    Integer[] answer = new Integer[N];

    int stageNum = 1;
    for (int i = 0; i < answer.length; i++) {
        answer[i] = stageNum;                    // [1, 2, ..., N]
        stageNum++;
    }

    for (int i = 1; i <= N; i++) {
        double demon = 0;                        // i번에 멈춘 사람
        double numer = 0;                        // i번에 도달한 사람
        double failRate = 0;
        for (int j = 0; j < stages.length; j++) {
            if (stages[j] >= i) {
                numer++;
            }
            if (stages[j] == i) {
                demon++;
            }
        }

        if (demon == 0 && numer == 0) {          // 도달한 사람이 없으면 0
            failRate = 0;
        } else {
            failRate = demon / numer;
        }
        map.put(i, failRate);
    }

    Arrays.sort(answer, (a, b) -> Double.compare(map.get(b), map.get(a)) == 0 ? a - b   // 같으면 번호 작은 게 앞
            : Double.compare(map.get(b), map.get(a)));                                 // 다르면 실패율 큰 게 앞

    int[] ans = new int[N];
    for (int i = 0; i < answer.length; i++) {
        ans[i] = answer[i];
    }
    return ans;
}
```

랜덤 20,000건에서 아래 AI 풀이와 결과가 같았다. 최악 입력(N = 500, 사용자 200,000명)은 291ms였다.

## 추적표: `N = 5, [2,1,2,6,2,4,3,3]`

| 스테이지 | 멈춘 사람 (`== i`) | 도달한 사람 (`>= i`) | 실패율 |
|---|---|---|---|
| 1 | 1 | 8 | 0.125 |
| 2 | 3 | 7 | 0.428… |
| 3 | 2 | 4 | 0.5 |
| 4 | 1 | 2 | 0.5 |
| 5 | 0 | 1 | 0 |

| a | b | `Double.compare(b의 실패율, a의 실패율)` | 결과 |
|---|---|---|---|
| 1 | 3 | `compare(0.5, 0.125)` = 1 | 3이 앞 |
| 3 | 4 | `compare(0.5, 0.5)` = 0 → `a - b` = -1 | 3이 앞 |
| 2 | 1 | `compare(0.125, 0.428)` = -1 | 2가 앞 |

→ `[3, 4, 2, 1, 5]`

## 오늘 배운 내용

**`double` 기준 정렬은 `Double.compare`를 쓰고, Comparator는 부호만 맞으면 된다.** `double` 뺄셈은 Comparator가 받는 `int`가 아니고, `(int)`로 자르면 작은 차이가 0이 된다. `Double.compare(x, y)`는 -1, 0, 1만 돌려주지만 정렬에는 그걸로 충분하다. 음수면 a가 앞, 양수면 b가 앞이라는 **방향**만 쓰기 때문이다.

## 오답노트

- **틀렸던 지점 1**: `int[]`에 Comparator. **왜**: 어제 배운 걸 새 문제에서 다시 놓쳤다. **다음에**: `Arrays.sort(배열, 람다)`를 쓰기 전에 배열 타입이 객체(`Integer[]`)인지 본다.
- **틀렸던 지점 2**: `double` 뺄셈을 Comparator에서 반환. **왜**: `a - b` 습관. **다음에**: 기준 값이 `double`이면 무조건 `Double.compare`.
- **틀렸던 지점 3**: `else if (stages[j] > i)`, `demon = numer - demon`. **왜**: "멈춘 사람 = 도달 - 통과"로 빼서 구하려 했는데, 조건이 겹쳐서 아무도 안 세졌다. **다음에**: 세려는 사람을 조건 하나로 바로 표현할 수 있는지 본다(멈춘 사람 = `== i`).
- **틀렸던 지점 4**: `Double.compare(b, a)`에 `map.get` 누락. **왜**: `a`, `b`가 번호라는 걸 잊었다. **다음에**: Comparator 안의 `a`, `b`는 배열 칸의 값(여기선 번호)이다. 기준 값은 `map.get(a)`로 꺼낸다.
- **틀렸던 지점 5**: `:` 뒤에 `b - a`, 그다음엔 `? b : a`. **왜**: `?`/`:` 한쪽에 "한쪽 값"이나 "다른 기준"을 넣는 실수가 3문제 연속으로 나왔다. **다음에**: `조건 ? X : Y`에서 X와 Y가 각각 **a와 b를 같은 기준으로 비교한 식**인지 확인한다. 헷갈리면 `comparing().thenComparing()`으로 쓴다.

## 꿀팁

**변수 이름을 분모/분자와 맞춘다.** 이번 코드는 멈춘 사람(분자)을 `demon`, 도달한 사람(분모)을 `numer`로 이름 붙였다. 결과는 맞았지만, 영어로 분모가 denominator, 분자가 numerator라서 이름과 의미가 반대다. `stop`, `reach`처럼 뜻이 바로 보이는 이름을 쓰면 `stop / reach`처럼 식도 그대로 읽힌다.

## AI라면 어떻게 풀었을까

**멈춘 사람 수를 한 번만 세고, 도달 인원은 빼 나간다.** 1번 스테이지에는 전원이 도달하고, 다음 스테이지로 갈 때마다 "방금 스테이지에 멈춘 사람"만큼 줄어든다(예시의 8 → 7 → 4 → 2 → 1). 그래서 스테이지마다 `stages` 전체를 다시 돌 필요가 없다. 정렬은 두 기준을 `comparing().reversed().thenComparing()`으로 이어 쓴다.

```java
public int[] solutionAlt(int N, int[] stages) {
    int[] stop = new int[N + 2];                 // stop[i] = i번 스테이지에 멈춘 사람 수
    for (int s : stages) {
        stop[s]++;
    }
    double[] rate = new double[N + 1];
    int reach = stages.length;                   // 1번 스테이지에는 전원 도달
    for (int i = 1; i <= N; i++) {
        rate[i] = reach == 0 ? 0 : (double) stop[i] / reach;
        reach -= stop[i];                        // i번에 멈춘 사람은 다음 스테이지에 못 감
    }
    Integer[] order = new Integer[N];
    for (int i = 0; i < N; i++) {
        order[i] = i + 1;
    }
    Arrays.sort(order, Comparator.comparing((Integer s) -> rate[s]).reversed()   // 실패율 내림차순
            .thenComparing(Comparator.naturalOrder()));                         // 같으면 번호 오름차순
    int[] ans = new int[N];
    for (int i = 0; i < N; i++) {
        ans[i] = order[i];
    }
    return ans;
}
```

트레이드오프: 반복 횟수가 `N × 사용자 수`(최대 1억 번)에서 `N + 사용자 수`로 줄어 최악 입력이 291ms → 1ms가 된다. 정렬 부분은 `?`/`:` 자리 고민이 없다. 대신 `.reversed()`는 **그 앞까지의 기준 전체**를 뒤집는다. 그래서 `thenComparing`보다 **먼저** 붙여야 번호 기준까지 뒤집히지 않는다. 실패율은 `HashMap` 대신 번호를 칸 번호로 쓰는 `double[]`에 담았다.
