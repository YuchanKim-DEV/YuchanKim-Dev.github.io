---
title: "[BACKEND] GitHub Actions의 GITHUB_TOKEN 권한을 읽기 전용으로 - CodeQL 경고 처리"
date: 2026-10-21 11:04:07 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [github-actions, security, codeql, github-token, permissions]
---

PR 검사 워크플로(`pr-check.yml`)의 토큰 권한을 읽기 전용으로 제한했다 ([PR #49](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/49), 이슈 #48).
CodeQL이 저장소 Security 탭에 경고 2건을 띄워 두고 있었는데, [이전 PR]({% post_url 2026-10-21-govtech-korea-backend-require-issue-link %})에서 같은 지적이 머지를 막고 나서야 알아챘다.

## 경고 내용

| 항목 | 값 |
|---|---|
| 규칙 | `actions/missing-workflow-permissions` |
| 심각도 | medium |
| 위치 | `.github/workflows/pr-check.yml` (잡 2개) |

## 새로 배운 것: GITHUB_TOKEN과 permissions

GitHub Actions는 워크플로를 실행할 때마다 그 실행 전용 토큰 `GITHUB_TOKEN`을 발급한다. `actions/checkout`이 저장소를 받을 때도 이 토큰을 쓴다.

이 토큰이 무엇을 할 수 있는지는 워크플로의 `permissions` 블록이 정한다. **블록이 없으면 저장소·조직의 기본 권한을 그대로 받는다.** 기본 권한이 "읽기·쓰기"로 설정된 곳이라면, 워크플로가 쓰는 서드파티 액션 하나가 오염됐을 때 그 토큰으로 코드를 push하거나 릴리스를 만들 수도 있다.

그래서 필요한 권한만 적는 것이 원칙이다(최소 권한).

```yaml
permissions:
  contents: read
```

| 워크플로 | 필요한 것 | 설정 |
|---|---|---|
| `pr-check.yml` | checkout(저장소 읽기) | `contents: read` |
| `pr-issue-link.yml` | 이벤트 본문만 읽음 | `{}` (권한 없음) |

## 확인: artifact 업로드는?

리포트를 올리는 `actions/upload-artifact`는 쓰기 작업이라 권한이 더 필요할까 걱정했다. 하지만 artifact 업로드는 `GITHUB_TOKEN`이 아니라 Actions 런타임이 따로 주는 토큰을 쓴다. 이 PR의 CI에서 읽기 권한만으로 리포트가 올라가는 것을 확인했다.

```
build        success
branch-flow  success
issue-link   success
test-reports 116154B
```

## 다음 할 일

- 개발 착수 전 결정 사항(기술 스택, 아키텍처, WBS) 논의
