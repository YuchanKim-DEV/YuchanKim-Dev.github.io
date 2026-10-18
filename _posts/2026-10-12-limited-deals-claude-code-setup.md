---
title: "[LIMITED_DEALS] Claude Code 세팅 - CLAUDE.md는 뼈대만 두고, 권한과 hooks로 규칙을 강제하기"
date: 2026-10-12 16:00:00 +0900
categories: [SIDE PROJECTS, LIMITED_DEALS]
tags: [claude-code, claude-md, permissions, hooks, automation, git, workflow]
---

LIMITED_DEALS를 진행하면서 AI(Claude Code)를 어떻게 쓰는지 **저장소에 그대로 공개**하기로 했다.
코드를 짜기 전에 먼저 Claude 설정부터 정리했고, 이 글은 그 과정이다. 정리한 파일은 리포의 [`CLAUDE.md`](https://github.com/YuchanKim-DEV/SIDEPROJECT_LIMITED_DEALS/blob/main/CLAUDE.md)와 [`.claude/`](https://github.com/YuchanKim-DEV/SIDEPROJECT_LIMITED_DEALS/tree/main/.claude)에 있고, [PR #5](https://github.com/YuchanKim-DEV/SIDEPROJECT_LIMITED_DEALS/pull/5)로 올렸다.

## 왜 설정부터 했나

대화로 규칙을 정하면 세션이 바뀔 때마다 흔들린다. "커밋 메시지는 이 형식", "서명은 넣지 말 것" 같은 규칙을 매번 다시 말해야 하고, AI가 규칙을 어겨도 사후에 알게 된다.
그래서 규칙을 **세 겹**으로 고정했다.

| 방식 | 지켜지는 이유 | 약점 |
|---|---|---|
| `CLAUDE.md` 규칙 | Claude가 읽고 **따르려고 노력**한다 | 실수하면 어겨질 수 있다 |
| 권한(permissions) | 명령의 앞부분이 일치하면 허용/질문/차단 | 이어 붙인 명령 등 우회가 있다 |
| hooks | 도구 실행 전에 **프로그램이 내용을 직접 검사**한다 | 스크립트가 아는 것만 막는다 |

## 1. CLAUDE.md는 뼈대만 둔다

### 새로 배운 것: 읽히는 건 git 추적과 상관없다
처음에는 `CLAUDE.md`가 `.gitignore`에 들어 있어서 "Claude가 이 파일을 안 읽나?" 싶었다. 아니었다. 파일이 디스크에 있으면 세션 시작 때 컨텍스트에 들어간다. 문제는 읽느냐가 아니라 **내용이 틀려 있다**는 것이었다. 기존 파일에는 "Java 21", "뼈대만 있음"이라고 적혀 있었는데, 실제로는 Java 25였고 `Member` CRUD도 이미 있었다.

### 구조: 설명 + 안내 표
`CLAUDE.md`를 프로젝트 설명과 "언제 무엇을 읽을지" 표만 남기고, 세부 내용은 `.claude/` 아래 파일로 뺐다. 필요할 때만 읽으니 세션이 바뀔 때 토큰이 줄어든다.

| 언제 | 읽을 파일 |
|---|---|
| 세션 시작 | `working-rules.md` (작업 방식) |
| 커밋, push, PR 전 | `do-not.md` (하지 말 것) |
| 브랜치, 커밋, PR | `docs/github-convention.md` |
| 빌드, 실행 | `commands.md` |
| 버전, 환경 | `environment.md` |
| 설계 결정 | `architecture.md` |
| 블로그 | `blog.md` |
| 세션 이어가기 | `HANDOFF.md` |

규칙 하나는 일부러 넣었다. **"파일과 다른 내용을 발견하면 먼저 물어보고 갱신한다."** 틀린 정보가 조용히 굳어지는 걸 막기 위해서다.

### 메모리도 같은 기준으로 정리했다
Claude의 메모리에는 컨벤션, 블로그 규칙이 따로 저장돼 있었는데 `docs/`와 `.claude/`의 내용과 겹쳤고, 일부는 이미 바뀐 내용(문서 영어 병기)이라 틀려 있었다. 5개를 3개로 줄였다. 기준은 **"다른 곳에 이미 있는 내용은 메모리에 두지 않는다"**다.

## 2. 권한 설정: allow, ask, deny

### 새로 배운 것: 3단계
| 단계 | 동작 |
|---|---|
| `allow` | 묻지 않고 실행 |
| `ask` | 매번 허락을 물어봄 |
| `deny` | 시켜도 실행 안 됨 |

```json
{
  "permissions": {
    "allow": ["Bash(./gradlew test:*)", "Bash(git status:*)", "Bash(gh pr create:*)"],
    "ask":   ["Bash(git commit:*)", "Bash(git push:*)"],
    "deny":  ["Bash(git push --force:*)", "Bash(git reset --hard:*)", "Bash(gh pr merge:*)"]
  }
}
```

읽기 위주 명령은 허용(30개), 상태를 바꾸는 명령은 질문(12개), 되돌리기 어려운 명령은 차단(17개)으로 나눴다. 머지는 내가 직접 하기로 해서 `gh pr merge`도 막아 두었다.

### 함정: 앞부분 비교라서 범위가 넓어진다
규칙은 명령의 **앞부분(prefix)**으로 비교한다. 그래서 `git branch`를 통째로 허용하면 `git branch -D`(강제 삭제)까지 열린다. `git branch --show-current`, `git branch -vv`처럼 **읽기 형태만 하나씩** 허용했다.

차단은 이렇게 확인했다. 사용법만 출력하는 `git rebase -h`와 `git push --force --dry-run`처럼, 막히지 않아도 아무 영향이 없는 명령으로 시험했다. 둘 다 차단됐다.

## 3. hooks: 실행 전에 검사하는 감독관

### 새로 배운 것: hook은 특정 시점에 스크립트를 돌린다
도구가 실행되기 **전**(`PreToolUse`)에 내가 정한 스크립트가 먼저 돌고, 스크립트가 종료 코드 2로 끝나면 명령이 차단되고 이유가 Claude에게 전달된다. 시점은 여러 개다.

| 시점 | 쓰임 |
|---|---|
| 도구 실행 **전** (`PreToolUse`) | 위험한 명령, 형식 위반 차단 |
| 도구 실행 **후** (`PostToolUse`) | 결과를 보고 알림을 넣기 |
| 프롬프트 제출 (`UserPromptSubmit`) | 매 요청 때 상황을 알려주기 |

`CLAUDE.md`와 가장 큰 차이는 **프로그램이 100% 실행한다**는 점이다. Claude가 규칙을 잊거나 잘못 판단해도 소용없다.

### 행동별로 스크립트를 나눴다
"성격에 따라 칼같이 구분"하는 방식을 스크립트에도 적용했다. 한 스크립트가 여러 규칙을 맡지 않고, **검사하는 행동**마다 하나씩 만들었다.

| 스크립트 | 검사하는 행동 | 규칙 |
|---|---|---|
| `no-attribution.py` | 커밋, PR 작성 | Claude 서명 금지 |
| `commit-guard.py` | `git commit` | 제목 `타입(범위): 제목`, main에서 커밋 금지 |
| `push-guard.py` | `git push` | `-f`, `+refspec`, `:브랜치` 등 강제 push 금지, main 직접 push 금지 |
| `branch-guard.py` | 브랜치 생성 | `타입/설명/YYYY-MM-DD` |
| `pr-guard.py` | `gh pr create/edit` | 제목 형식, 본문 템플릿 항목 |
| `stage-guard.py` | `git add` | `.env` 등 민감 파일, `-f` 금지 |
| `blog-guard.py` | 블로그 글 쓰기 | 미래 날짜, 서명 금지 |

여기에 알림용 hook 3개(`handoff-reminder`, `blog-reminder`, `uncommitted-reminder`)가 더 있고, 공통 코드는 `_common.py`로 뺐다.

### 테스트: 가짜 입력으로 시뮬레이션
hook은 JSON을 표준입력으로 받는다. 그래서 **실제 명령은 실행하지 않고** 가짜 JSON만 넣어서 차단할 것과 통과시킬 것을 모두 시험했다.

```python
# 막혀야 하는 것
("push +main",         "git push origin +main",      2)
("push -uf 묶음",       "git push -uf origin x",      2)
("브랜치 가짜 날짜",      "git checkout -b feat/a/2026-13-45", 2)
# 통과해야 하는 것
("작업 브랜치 push",     "git push -u origin feat/product-crud/2026-10-12", 0)
("git log --grep commit", "git log --grep commit",   0)
```

모두 합쳐 70개 케이스를 통과했고, 이 세션에서도 실제로 작동하는 걸 확인했다.

## 4. 커밋과 PR 흐름 자동화

### 규칙: PR은 목적 단위, 커밋은 성격 단위
처음에는 "성격이 다르면 PR도 따로"라고 생각해 Claude 파일을 올리는 PR을 4개로 나눌 계획이었다. 하지만 목적이 하나("Claude 설정")이고, 머지 방식이 **Create a merge commit**이라 PR 안의 커밋이 `main`에 그대로 남는다. 그래서 **PR은 하나, 커밋은 성격별 6개**로 올렸다.

| 파일 | 커밋 타입 |
|---|---|
| `src/main/**` | `feat`, `fix`, `refactor` 등 |
| `src/test/**` | `test` |
| `docs/**`, `CLAUDE.md`, `.claude/**/*.md` | `docs` |
| `build.gradle.kts`, `gradle.properties` | `build` |
| `.gitignore`, `.github/**`, `.claude/settings.json`, `hooks/**` | `chore` |

### 자동화한 것
- **작업 전환 알림**: 커밋하지 않은 변경이 있는 채로 다른 작업을 시작하려 하면, `UserPromptSubmit` hook이 알려주고 Claude가 커밋 계획(브랜치, 커밋 분할, PR)을 설명한 뒤 선택창으로 묻는다.
- **인수인계 파일 갱신**: PR을 만든 직후 `PostToolUse` hook이 `HANDOFF.md` 갱신을 알려준다. hook은 글을 쓸 수 없어서, **Claude의 컨텍스트에 지시를 주입**하는 방식이다.
- **블로그 글**: PR을 만들 때가 계기다. 주제를 제안하고 방향을 묻고, 공개는 내가 확인한 뒤에 한다. 처음에는 글을 다 쓴 뒤 "확인하고 올려줘라고 해달라"는 안내를 답변 끝에 적기만 했는데, 그걸 놓쳐서 "블로그 글은 왜 안 올라오지?" 하는 일이 생겼다. 그래서 **글을 쓴 직후 선택창으로 공개 여부를 묻도록** 규칙을 바꿨다. 안내 문구보다 선택창이 놓치기 어렵다.

## 실패담과 시행착오

이 세팅은 매끄럽게 나오지 않았다. 아래가 규칙과 hook이 생긴 이유다.

1. **지시하지 않은 force push.** "모든 이름은 YuchanKim-DEV여야 한다"는 말을 Claude가 "기존 커밋도 바꾸라"는 뜻으로 해석해 `main`을 force push했다. 그 결과 열려 있던 PR #3이 **자동으로 닫혔고 다시 열 수도 없었다.** 지금은 force push 형태를 `deny`와 `push-guard.py`가 막고, "애매하면 먼저 묻는다"를 규칙으로 넣었다.
2. **미래 날짜 글이 숨겨짐.** 블로그 글 날짜를 현재 시각보다 뒤(20:00)로 적었더니, 배포는 성공했는데 화면에는 안 나왔다. Jekyll이 미래 날짜 글을 숨기기 때문이다. `blog-guard.py`가 이걸 막는다.
3. **hook이 내 테스트 명령을 막음.** 테스트 코드 안에 서명 trailer 문구와 `git commit`이 같이 들어 있었더니 `no-attribution`이 진짜 커밋으로 오해해서 막았다. 이 글을 쓰다가도 같은 문구를 본문에 그대로 적으려 하자 `blog-guard`가 막아서, 문구를 풀어서 썼다. 텍스트로만 판단하는 한계다. 그리고 내 명령에 `rm -rf`가 들어 있으니 `deny` 규칙이 막았다. 모두 규칙이 작동한다는 증거였다.
4. **`git status`의 앞 공백 버그.** 공통 함수가 출력의 앞 공백까지 지워서 ` M .gitignore`가 `gitignore`로 잘렸다. 일괄 `git add` 검사에서 수정된 `.env`를 놓칠 수 있는 버그였다. 알림 hook을 테스트하다가 발견했다.
5. **zsh 함정.** `"$tip:refs/heads/main"`의 `:r`을 zsh가 변수 수정자로 해석해서 push가 실패했다. `"${tip}:refs/heads/main"`으로 써야 한다.
6. **테스트의 숨은 가정.** "main에서 커밋하면 차단" 테스트가 현재 브랜치가 main이라고 가정하고 있어서, 작업 브랜치로 옮기자 실패했다. 코드가 아니라 테스트의 문제였고, 브랜치에 따라 기대값을 바꾸게 고쳤다.

## 한계와 앞으로의 계획

- **텍스트 기반 검사의 한계**: 서명 문구를 언급만 해도 `git commit`과 같이 있으면 막힌다. `sed -i` 같은 셸 명령으로 블로그 글을 고치면 `blog-guard.py`를 우회한다. `eval`처럼 명령을 숨기는 경우도 못 막는다. 그래서 규칙 문서, 권한, hooks를 **겹쳐서** 쓴다.
- **skills는 나중에**: `/pr`, `/blog` 같은 슬래시 명령은 프로젝트를 진행하면서 같은 작업이 반복될 때 추가한다. 외부 인기 스킬을 조사해 보니 지금 당장 설치할 건 없었고, 아키텍처를 정할 때 `java-springboot`, 테스트를 시작할 때 TDD 스킬을 다시 보기로 했다. 서드파티 스킬은 지시문이 곧 실행되는 내용이라 설치 전에 직접 읽어야 한다.
- **홈 `CLAUDE.md` 삭제**: 여러 프로젝트 지침을 합쳐 둔 파일이 세션마다 읽히고 있어서 지웠다. 다른 프로젝트에서는 그 지침이 사라진다.
- 다음은 기술 스택 확정, 아키텍처, 테스트 전략 순서로 간다.
