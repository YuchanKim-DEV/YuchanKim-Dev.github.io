---
title: "[BACKEND] 모든 PR은 이슈를 닫는다 - CI로 Closes #N 강제하기"
date: 2026-10-21 11:03:07 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [github-actions, ci, github-projects, issue, pull-request, codeql]
---

GitHub Projects 보드를 정리하다가, 완료(Done)된 PR 중 이슈와 연결되지 않은 것이 보였다. 컨벤션에는 "작업 전에 이슈를 만든다"고 써 놓고, 문서·설정 같은 작은 작업은 이슈 없이 진행한 것이다. 규칙을 CI로 강제했다 ([PR #47](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/47), 이슈 #46).

## 먼저 알게 된 것: Linked pull requests 칸은 이슈 행에만 채워진다

보드의 `Linked pull requests` 칸이 비어 있는 행을 조사해 보니 두 종류였다.

| 행 | 칸 | 이유 |
|---|---|---|
| 이슈 #15~#18 | `#42`, `#44`, `#41`, `#40` | PR 본문의 `Closes #N`으로 연결됨 |
| PR #40~#44 | 비어 있음 | 이 칸은 "이 이슈를 닫은 PR"을 보여 주는 칸이라 **PR 행에는 원래 값이 없다** |
| PR #1~#3, #38, #39 | 비어 있음 | 위 이유 + **닫는 이슈 자체가 없는 PR** |

진짜 문제는 마지막 줄이었다. #38, #39는 본문에 `Refs #19`처럼 이슈를 적긴 했지만, `Refs`는 GitHub이 연결로 인식하지 않는다. `Closes`, `Fixes`, `Resolves` 계열 키워드만 이슈와 PR을 연결하고, 머지될 때 이슈를 닫는다.

## 변경 내용: issue-link 검사

```yaml
name: PR Issue Link

on:
  pull_request:
    branches: [main, develop]
    types: [opened, edited, synchronize, reopened]

permissions: {}

jobs:
  issue-link:
    name: issue-link
    runs-on: ubuntu-latest
    steps:
      - name: 이슈를 닫는 키워드 검사
        env:
          BASE: ${{ github.base_ref }}
          HEAD: ${{ github.head_ref }}
          BODY: ${{ github.event.pull_request.body }}
        run: |
          if [ "$BASE" = main ] || [[ "$HEAD" == dependabot/* ]]; then
            exit 0
          fi
          if printf '%s' "$BODY" | grep -Eiq '(close[sd]?|fix(e[sd])?|resolve[sd]?):?[[:space:]]+#[0-9]+'; then
            exit 0
          fi
          echo "::error::PR 본문에 이 PR이 닫는 이슈를 적어 주세요. 예: Closes #12"
          exit 1
```

| 설계 | 이유 |
|---|---|
| `types`에 `edited` 추가 | 기본 트리거는 본문을 고쳐도 다시 돌지 않는다. 실패한 뒤 본문만 고치면 재검사돼야 한다 |
| `pr-check.yml`과 분리 | 같은 파일에 넣으면 본문을 고칠 때마다 빌드까지 다시 돈다 |
| 본문을 `env`로 전달 | `${{ github.event.pull_request.body }}`를 `run:`에 직접 쓰면 본문 내용이 셸 명령으로 실행될 수 있다(스크립트 주입) |
| 예외 | Dependabot PR(이슈가 없음), develop → main 통합 PR |

머지 후 ruleset의 필수 검사에 `issue-link`를 추가했다. 이제 이슈를 닫지 않는 PR은 머지 버튼이 잠긴다.

## 검증: 실제 PR에서 일부러 실패시켜 보기

로컬에서 판정 로직을 9가지 경우로 돌려 본 뒤, PR #47 자체에서도 확인했다. 본문의 `Closes #46`을 잠깐 `Refs #46`으로 바꿨다가 되돌렸다.

```
00:44:56Z success   ← Closes #46
00:45:14Z failure   ← Refs #46 로 수정
00:45:40Z success   ← 원래대로
```

## 시행착오: 검사는 다 통과했는데 머지가 안 된다

자동 머지를 걸어 뒀는데 5분이 지나도 머지되지 않았다. REST API로 직접 머지를 시도하자 이유가 나왔다.

```
Repository rule violations found
A conversation must be resolved before this pull request can be merged.
```

ruleset에 "리뷰 대화를 모두 resolve해야 머지"를 걸어 뒀는데, **CodeQL 봇**이 PR에 리뷰 댓글을 남겨 둔 것이다.

> Workflow does not contain permissions — Actions job or workflow does not limit the permissions of the GITHUB_TOKEN.

워크플로에 `permissions`를 적지 않으면 그 실행의 `GITHUB_TOKEN`이 저장소 기본 권한을 그대로 갖는다. 이 검사는 이벤트 본문만 읽으니 권한이 전혀 필요 없어서 `permissions: {}`를 추가했다. 수정을 push하자 CodeQL이 다시 분석했고, 대화가 자동으로 resolve되면서 머지됐다.

같은 경고가 기존 `pr-check.yml`에도 2건 열려 있었다는 것도 이때 알았다. 그건 다음 PR에서 고쳤다.

## 다음 할 일

- `pr-check.yml`의 토큰 권한 최소화
