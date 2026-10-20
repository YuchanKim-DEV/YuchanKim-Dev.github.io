---
title: "[BACKEND] 프로젝트 세팅 - Spring Boot 4.1.1, Git Flow, 그리고 main·develop을 서버에서 잠그기"
date: 2026-10-21 01:40:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [spring-boot, git-flow, github, ruleset, github-actions, ci, convention]
---

GovTech Korea 공모전에 낼 백엔드 프로젝트를 시작했다.
첫 단계는 코드가 아니라 **규칙**이다. 버전을 정하고, 브랜치·커밋 규칙을 정하고, 그 규칙을 사람이 아니라 도구가 지키게 만들었다.
마지막으로 첫 PR([#1](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/1))로 CI를 넣었다.

## 환경

| 항목 | 값 |
|---|---|
| Spring Boot | 4.1.1 |
| Java | 25 (Gradle toolchain) |
| 빌드 | Gradle 9, Groovy DSL |
| 의존성 | Web MVC, Validation, Lombok, DevTools |
| 저장소 | `GovTech-korea/GovTech-Korea_backend` (public) |

### Spring Initializr에서 만난 500 오류

M1이나 SNAPSHOT이 붙지 않은 최신 안정 버전은 4.1.1이었다. Initializr 메타데이터에는 `4.1.1.RELEASE`로 나오는데, 이 값을 그대로 `bootVersion`에 넣어 요청하면 서버가 500을 돌려준다.

```
Bom 'org.springframework.boot:spring-boot-dependencies:4.1.1.RELEASE' could not be resolved
```

실제 Maven 좌표에는 `.RELEASE`가 붙지 않기 때문이다. `bootVersion=4.1.1`로 요청하니 정상적으로 받아졌다.

## 브랜치 전략: 간소화한 Git Flow

```
작업 브랜치 ──PR──▶ develop ──PR (통합 테스트 후)──▶ main
```

- `main`: 제출용. 항상 동작하는 상태
- `develop`: 개발 통합
- 작업 브랜치: `feat/12-sign-up-api`처럼 `<타입>/<이슈번호>-<설명>`

규칙은 두 가지다. **작업 브랜치는 develop으로만, main은 develop에서만** PR을 받는다. 그리고 `main`, `develop`에는 절대 직접 push하지 않는다.

## 커밋은 성격을 까다롭게 나눈다

원칙은 **브랜치 하나 = 목적 하나, 커밋 하나 = 성격 하나**다. 타입을 20개로 세분화했다.

| 분류 | 타입 |
|---|---|
| 기능 | `feat`, `update` |
| 수정 | `fix`, `hotfix`, `security` |
| 코드 개선 | `refactor`, `perf`, `style`, `rename`, `remove` |
| 테스트·문서 | `test`, `docs`, `comment` |
| 설정 | `build`, `deps`, `config`, `ci`, `chore` |
| 기타 | `init`, `revert` |

타입이 많으면 고르는 기준이 흔들린다. 그래서 변경 덩어리마다 순서대로 대입하는 **판정 순서**를 정했다. "되돌림인가 → CI 파일인가 → 의존성인가 → … → 동작이 바뀌는가 → 새 기능인가, 기존 기능 변경인가"처럼 처음 "예"가 나오는 타입을 쓴다. 두 타입이 나오면 커밋을 나눈다.

브랜치마다 허용하는 커밋 타입도 정했다. 예를 들어 `feat/` 브랜치에서는 `refactor` 커밋을 할 수 없다. 기능 작업 중 리팩터링이 필요하면 `refactor/` 브랜치를 따로 판다.

## 규칙을 도구가 지키게 하기

규칙을 문서로만 두면 결국 안 지킨다. 그래서 세 겹으로 막았다.

| 위치 | 도구 | 막는 것 |
|---|---|---|
| 내 컴퓨터 | git `commit-msg` hook | 형식이 틀린 메시지, 브랜치에 허용되지 않은 타입, main·develop 직접 커밋 |
| 내 컴퓨터 | git `pre-push` hook | main·develop push, 이름 규칙을 어긴 브랜치 push |
| GitHub | ruleset | 모든 사람의 main·develop 직접 push, force push, 삭제, CI 미통과 머지 |

로컬 hook은 내 컴퓨터에서만 동작한다. 다른 컴퓨터에서 push하거나 GitHub 웹에서 머지하면 피할 수 있다. 그래서 서버 쪽 잠금이 필요했다.

## 새로 배운 것 1: ruleset과 public 저장소

**ruleset**은 GitHub 서버가 특정 브랜치에 거는 규칙 묶음이다. `main`, `develop`에 아래 규칙을 걸었다.

- 직접 push·force push·삭제 금지 (PR로만 반영)
- 필수 status check: `branch-flow`, `build` 통과
- 머지 방식은 merge commit만
- 관리자도 우회 불가

처음에는 저장소를 private으로 만들 생각이었다. 그런데 **GitHub Free 조직은 private 저장소에 ruleset을 걸 수 없다**. 그래서 public으로 만들었다. 대신 비밀번호나 API 키가 커밋에 들어가면 바로 노출되므로, secret scanning과 push protection을 같이 켰다.

## 새로 배운 것 2: CI

**CI(Continuous Integration)**는 코드를 올릴 때마다 서버가 빌드와 테스트를 자동으로 돌려 확인하는 것이다. "내 컴퓨터에선 됐는데"를 막는다.

GitHub Actions로 PR마다 두 가지를 검사한다.

```yaml
jobs:
  branch-flow:   # develop으론 작업 브랜치만, main으론 develop만
  build:         # Java 25 설치 후 ./gradlew build
```

ruleset이 이 두 검사를 필수로 요구한다. 그래서 **CI 워크플로가 develop에 들어가기 전에는 어떤 PR도 머지할 수 없다.** 첫 PR이 이 워크플로를 추가하는 PR이어야 했던 이유다. PR #1에서 워크플로가 처음 실행됐고, `branch-flow` 3초, `build` 1분 5초로 통과했다.

## 새로 배운 것 3: Squash를 안 쓰는 이유, 그리고 rebase

머지 방식은 세 가지다.

```
merge commit:  A ─ B ─ C ─────── M     ← 합쳤다는 커밋 M, 갈라진 기록이 남음
                    └─ X ─ Y ─┘
squash:        A ─ B ─ C ─ (X+Y)       ← 작업 커밋을 하나로 합침
rebase:        A ─ B ─ C ─ X' ─ Y'     ← X, Y를 C 뒤로 옮겨 다시 만듦 (해시가 바뀜)
```

squash는 기록이 깔끔하지만 작업 중 커밋이 사라진다. rebase는 일직선이지만 커밋 해시가 바뀌고, 이미 push한 커밋에 쓰면 force push가 필요해진다. 커밋을 성격별로 까다롭게 나누기로 했으니, 그 커밋들이 그대로 남는 **merge commit**만 쓰기로 했다. 저장소 설정에서 squash와 rebase 버튼은 꺼 두었다.

## 다음 할 일

- README와 PR·이슈 템플릿
- 테스트 환경 구성 (JUnit, Mockito)
