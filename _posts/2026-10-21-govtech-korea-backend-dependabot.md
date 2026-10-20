---
title: "[BACKEND] Dependabot 설정 - 브랜치 규칙과 커밋 규칙에 맞추기"
date: 2026-10-21 02:25:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [dependabot, github-actions, ci, dependency-management]
---

의존성과 GitHub Actions 버전을 Dependabot이 매주 확인해서 PR을 열도록 설정했다 ([PR #42](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/42), 이슈 #15).
설정 파일만 넣으면 끝날 줄 알았는데, 이 저장소는 브랜치·커밋 규칙이 까다로워서 Dependabot을 규칙에 맞추는 작업이 먼저 필요했다.

## Dependabot의 두 가지 기능

| 기능 | 하는 일 | 설정 |
|---|---|---|
| 보안 알림 (alerts) | 쓰고 있는 의존성에 알려진 취약점이 생기면 알림 | 저장소 설정에서 켬 (이미 켜 둠) |
| 버전 업데이트 (version updates) | 새 버전이 나오면 업데이트 PR 생성 | `.github/dependabot.yml` |

이번에 추가한 건 두 번째다.

## 문제: Dependabot PR은 규칙을 통과하지 못한다

이 저장소의 CI(`branch-flow`)는 develop으로 들어오는 PR의 브랜치 이름이 `feat/`, `fix/` 같은 정해진 접두사로 시작해야 통과시킨다. 그런데 Dependabot은 이런 이름으로 브랜치를 만든다.

```
dependabot/gradle/<패키지>-<버전>
dependabot/github_actions/<그룹 또는 액션>-<해시>
```

그대로 두면 Dependabot PR이 전부 CI에서 실패하고, ruleset 때문에 머지도 못 한다. 그래서 커밋을 둘로 나눴다.

```
ci: Dependabot PR 브랜치를 브랜치 흐름 검사에서 허용
ci: Dependabot 버전 업데이트 설정
```

```bash
# pr-check.yml의 branch-flow
elif ! [[ "$HEAD" =~ ^(feat|update|fix|...|chore|dependabot)/ ]]; then
```

## 설정

```yaml
version: 2
updates:
  - package-ecosystem: gradle
    directory: /
    target-branch: develop
    schedule:
      interval: weekly
      day: monday
      time: "09:00"
      timezone: Asia/Seoul
    labels: [deps]
    commit-message:
      prefix: deps
    groups:
      spring-boot:
        patterns:
          - org.springframework.boot*
          - io.spring.dependency-management*

  - package-ecosystem: github-actions
    directory: /
    target-branch: develop
    labels: [ci]
    commit-message:
      prefix: ci
    groups:
      actions:
        patterns: ["*"]
```

| 설정 | 이유 |
|---|---|
| `target-branch: develop` | 기본값은 기본 브랜치다. 지정하지 않았다가 main으로 PR이 열리면 브랜치 흐름 규칙 위반 |
| `commit-message.prefix` | Dependabot 커밋 제목이 `deps: bump ...`, `ci: bump ...`가 되어 커밋 컨벤션과 맞는다 |
| `labels` | 이슈·PR 라벨을 커밋 타입과 같은 이름으로 쓰고 있어서 맞췄다 |
| `groups` | Spring Boot 관련 의존성은 같이 올라가야 의미가 있고, Actions 업데이트는 하나로 묶어 PR 수를 줄인다 |

## 새로 배운 것: 설정을 넣자마자 첫 검사가 돈다

develop(기본 브랜치)에 머지되자마자 Actions 탭에 이런 실행이 생겼다.

```
gradle in /. - Update #1640129560          dynamic  in_progress
github_actions in /. - Update #1640129550  dynamic  in_progress
```

Dependabot의 버전 검사도 GitHub Actions 실행으로 보인다. 일정(매주 월요일)을 기다리지 않고 설정이 들어온 시점에 한 번 바로 검사한다.

## 다음 할 일

- Dependabot이 연 PR을 확인하고 처리
- CI에서 테스트·커버리지 리포트를 내려받을 수 있게 업로드
