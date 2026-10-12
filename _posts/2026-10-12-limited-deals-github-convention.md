---
title: "[LIMITED_DEALS] GitHub 컨벤션 정하기 - 브랜치, 커밋, PR, 그리고 Create a merge commit을 고른 이유"
date: 2026-10-12 14:40:00 +0900
categories: [SIDE PROJECTS, LIMITED_DEALS]
tags: [github, convention, git, branch, commit, pull-request, merge-commit]
---

혼자 하는 학습 프로젝트지만, 면접에서 협업 흐름을 설명할 수 있도록 **실무 방식**으로 연습하기로 했다.
코드를 쓰기 전에 GitHub 컨벤션부터 정했다. 정한 내용은 리포의 `docs/github-convention.md`에도 있다.

## 1. Branch

**GitHub Flow**를 단순하게 쓴다.

- `main`은 항상 빌드가 되는 상태로 두고, 직접 커밋하지 않는다.
- 작업은 `main`에서 브랜치를 따서 하고, PR(Pull Request)로 합친다.

브랜치 이름은 `타입/설명/시작날짜`다. 설명은 영어 kebab-case, 날짜는 `YYYY-MM-DD`로 쓴다.

```
feat/product-crud/2026-10-12
fix/deal-price-calc/2026-10-15
docs/github-convention/2026-10-12
```

## 2. Commit

**Conventional Commits** 형식이다. 제목은 한글로 쓴다.

```
타입(범위): 제목 (50자 이내, 마침표 없음)

본문 (선택) - 무엇보다 왜 바꿨는지
```

| 타입 | 용도 |
|---|---|
| `feat` | 새 기능 추가 |
| `fix` | 버그 수정 |
| `hotfix` | `main`에 반영된 긴급 버그 수정 |
| `changes` | 기능 변경 (동작이 바뀌는 수정) |
| `refactor` | 동작은 그대로, 코드 구조 개선 |
| `test` | 테스트 추가 / 수정 |
| `docs` | 문서 수정 |
| `chore` | 설정, 잡일 |
| `build` | 빌드 설정, 의존성 변경 |

예: `feat(product): 상품 등록 API 추가`

기존 커밋은 `changes: gitignore에 ...`처럼 형식이 제각각이었다. 앞으로는 위 규칙을 따른다.

## 3. Pull Request

- 브랜치 하나당 PR 하나
- 제목은 커밋 형식과 같다
- 본문은 `.github/pull_request_template.md` 템플릿을 쓴다: **요약, 변경 사항, 테스트 방법, 관련 이슈**
- 머지 후 작업 브랜치는 삭제한다

### 머지 방식: Create a merge commit

PR 화면의 머지 버튼에는 세 가지 방식이 있다. 작업 브랜치에 커밋이 A, B, C 세 개 있다고 하자.

| 방식 | `main`에 남는 결과 | 특징 |
|---|---|---|
| **Create a merge commit** (기본값) | A, B, C + 머지 커밋 1개 | 이력이 그대로 남지만 그래프가 갈라졌다 합쳐져서 복잡해진다 |
| Squash and merge | A+B+C를 합친 커밋 **1개** | PR 하나가 커밋 하나로 남는다 |
| Rebase and merge | A, B, C가 `main` 끝에 일렬로 붙음 | 이력이 일직선이지만 커밋 해시가 바뀐다 |

처음에는 PR 하나가 커밋 하나로 남는 **Squash and merge**를 쓰려고 했다. 그런데 첫 PR을 기본값인 **Create a merge commit**으로 머지해 보고, 세 방식의 차이를 다시 비교한 뒤 이 방식으로 정했다.

**Create a merge commit**을 선택했다.

- 작업 브랜치에서 쌓은 커밋이 `main`에 그대로 남는다. 작업 과정과 "언제 갈라져서 언제 합쳤는지"가 이력에 보존된다.
- 머지 커밋이 PR 하나의 경계를 보여준다. 문제가 생기면 머지 커밋 하나를 되돌리면 PR 전체가 취소된다.
- GitHub 기본값이라 별도 설정이 필요 없다.

대신 이력 그래프가 갈라졌다 합쳐져서 Squash보다 복잡하다. 또 작업 중 커밋이 `main`에 그대로 남으므로, 커밋 메시지를 컨벤션대로 꼼꼼히 써야 한다.

### 리뷰 코멘트

리뷰어가 없으니 직접 코멘트를 달고, 머지 전에 Claude에게 리뷰를 맡긴다.
코드의 특정 줄에 "왜 이렇게 했는지"를 남기고, 접두사를 붙인다.

| 접두사 | 뜻 |
|---|---|
| `[Must]` | 머지 전에 반드시 고쳐야 함 |
| `[Suggestion]` | 더 나은 방법 제안 |
| `[Question]` | 궁금한 점 |
| `[Note]` | 참고용 메모 |

## 4. Issue (선택)

하나의 작은 작업 덩어리(예: 상품 CRUD 구현)를 이슈로 만들 수 있다. 필수는 아니다.
이슈가 있으면 PR 본문에 `Closes #번호`를 쓴다. 머지되면 이슈가 자동으로 닫힌다.

## 5. Claude 전용 파일

Claude가 참고하는 `CLAUDE.md` 같은 md, txt 파일은 반드시 `.gitignore`에 넣는다.

## 정리

| 항목 | 규칙 |
|---|---|
| 브랜치 | `타입/설명/시작날짜` |
| 커밋 | `타입(범위): 제목`, 제목은 한글 |
| 머지 | PR + Create a merge commit |
| 리뷰 | 직접 코멘트 + Claude 리뷰, 접두사 사용 |
| 이슈 | 선택, `Closes #번호`로 연결 |

이 컨벤션 문서 자체도 위 규칙대로 `docs/github-convention/2026-10-12` 브랜치에서 PR로 올렸다.
