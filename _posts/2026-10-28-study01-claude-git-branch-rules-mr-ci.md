---
title: "[STUDY_01][CLAUDE] 설정 - Git 브랜치 규칙을 MR과 CI로 강제하기"
date: 2026-10-28 14:38:26 +0900
categories: [STUDY, CLAUDE]
tags: [claude-code, git, gitlab, gitlab-ci, merge-request, workflow]
---

## 목적

브랜치 규칙을 문서로만 정해 두면, 바쁠 때 한 번쯤은 어기게 된다. 사람도 그렇고 Claude도 그렇다. 그래서 이번에는 규칙을 **서버가 막게** 만들고, 그 흐름을 Claude가 처음부터 끝까지 따라가게 했다.

정한 규칙은 이렇다.

- 작업 브랜치(`feat/`, `fix/` 등)는 **`dev`로만** merge한다.
- `production`에는 **`dev`에서만** merge한다.
- 긴급 수정(`hotfix/`)도 예외 없이 `dev`에서 테스트한 뒤 `production`으로 간다.

## 접근 방식

규칙을 세 겹으로 걸었다.

```
feat/xxx 에서 작업, push
        │
        ▼
MR 생성: feat/xxx → dev
        │
        ├─ ① 보호 브랜치    : dev 에 직접 push 불가 → MR 로만 바뀐다
        ├─ ② CI 검사        : 출처 브랜치가 규칙에 맞는지 자동 확인
        └─ ③ 파이프라인 필수 : ② 가 실패하면 Merge 버튼이 막힌다
        │
        ▼
Merge → dev
```

| 규칙 | 막는 것 |
|---|---|
| ① 보호 브랜치 | `production`, `dev`에 직접 push, 강제 push |
| ② CI 검사 | 정해진 경로가 아닌 MR |
| ③ 파이프라인 성공 시에만 merge | 검사에 실패한 MR의 merge |

MR(Merge Request)은 규칙 자체가 아니라 "이 브랜치를 저 브랜치에 합쳐 달라"는 **요청서**다. ①이 MR을 유일한 통로로 만들고, ②와 ③이 그 통로를 검사한다.

## 처음 구상

GitLab 무료 버전에는 "이 브랜치에는 저 브랜치만 merge할 수 있다" 같은 설정이 없다. 보호 브랜치로 직접 push는 막을 수 있지만, **어디서 오는 MR인지**는 따지지 않는다.

그래서 MR이 만들어질 때 도는 CI job으로 출처와 대상을 직접 검사하기로 했다. GitLab은 MR 파이프라인에 출처와 대상 브랜치 이름을 변수로 넣어 준다.

```yaml
check-merge-path:
  stage: .pre
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  script:
    - |
      src="$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME"
      dst="$CI_MERGE_REQUEST_TARGET_BRANCH_NAME"
      case "$dst" in
        production)
          [ "$src" = "dev" ] || { echo "production 은 dev 에서만"; exit 1; } ;;
        dev)
          case "$src" in
            feat/*|fix/*|refactor/*|test/*|docs/*|chore/*|hotfix/*) ;;
            *) echo "dev 에는 작업 브랜치만"; exit 1 ;;
          esac ;;
      esac
```

검사는 빌드보다 먼저 끝나는 게 좋겠다고 생각해서 `.pre` 단계에 넣었다.

## 구현

보호 브랜치는 GitLab API로 걸었다. 직접 push는 아무도 못 하게(`No one`), merge는 Maintainer만, 강제 push는 금지.

CI 파일은 규칙대로 작업 브랜치에서 올렸다. Claude가 한 순서는 이렇다.

1. `dev`에서 `chore/branch-rules` 브랜치 생성, 커밋, push
2. API로 MR 생성 (`chore/branch-rules` → `dev`)
3. 파이프라인 결과를 기다렸다가 job 로그 확인
4. 통과하면 merge (작업 브랜치 자동 삭제)
5. `dev` → `production` MR 생성, 같은 검사 통과 확인 후 merge
6. 마지막에 "파이프라인 성공 시에만 merge" 설정 켜기

6번을 마지막에 한 이유가 있다. CI 파일이 들어가기 전에 이 설정부터 켜면, 파이프라인이 없는 MR은 **영원히 merge할 수 없다.** 규칙을 들여오는 MR 자체가 막혀 버린다.

## 다시 구상

MR을 만들었는데 파이프라인이 **아예 생기지 않았다.** 실패도 아니고, 그냥 없었다.

원인은 `.pre` 단계였다. GitLab은 `.pre`와 `.post` 단계에만 job이 있는 파이프라인은 **만들지 않는다.** 이 두 단계는 다른 단계의 앞뒤에 붙는 보조 단계라서, 본 단계 job이 하나도 없으면 파이프라인을 만들 이유가 없다고 본다.

```diff
 check-merge-path:
-  stage: .pre
+  stage: test
```

`test`는 따로 정의하지 않아도 있는 기본 단계다. 바꾸고 push하자 MR에 파이프라인이 붙었고, 로그에 `MR: chore/branch-rules -> dev`가 찍히며 통과했다. `dev` → `production` MR도 같은 검사를 통과한 뒤 merge했다.

## 겪은 문제

**1. API로 MR을 만들 때 `Bad Request`.** MR 제목을 한글로 넣었더니 요청 자체가 거부됐다. 명령줄 인자로 넘기는 과정에서 한글 인코딩이 깨진 것이다. 요청 본문을 **UTF-8 JSON 파일**로 만들어 `--data-binary @파일`로 보내니 해결됐다.

**2. 파이프라인이 안 생김.** 위의 `.pre` 단계 문제. 오류 메시지가 없어서 찾기 어려웠다. "실패"가 아니라 "없음"이면 파이프라인 생성 조건부터 의심해야 한다.

**3. merge 커밋 메시지도 규칙에 맞추기.** GitLab 기본 merge 메시지는 `Merge branch 'x' into 'y'`다. 커밋 규칙에 `merge` 타입을 정해 두었기 때문에, merge할 때 메시지를 `merge: chore/branch-rules → dev (...)` 형식으로 지정했다.

## 새로 배운 것

- GitLab 보호 브랜치는 "누가 push/merge 할 수 있는가"만 정한다. "어디서 온 MR인가"는 CI로 검사해야 한다.
- `.pre`, `.post` 단계만 있으면 파이프라인이 만들어지지 않는다.
- "파이프라인 성공 시에만 merge"는 **CI 파일이 들어간 뒤에** 켜야 한다.
- MR은 규칙이 아니라 통로다. 규칙은 통로에 거는 것이다.
