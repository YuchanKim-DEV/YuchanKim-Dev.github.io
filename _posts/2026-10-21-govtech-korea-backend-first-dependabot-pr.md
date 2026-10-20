---
title: "[BACKEND] Dependabot의 첫 PR - GitHub Actions 메이저 버전 4개 올리기"
date: 2026-10-21 02:42:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [dependabot, github-actions, ci]
---

Dependabot을 설정하고 몇 분 뒤 첫 PR이 열렸다. 최종적으로 머지한 것은 [PR #45](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/45)다.
그 사이에 PR #43이 열렸다가 닫혔는데, 그 과정에서 Dependabot이 동작하는 방식을 알게 됐다.

## 변경 내용

| 액션 | 전 | 후 |
|---|---|---|
| `actions/checkout` | v4 | v7 |
| `actions/setup-java` | v4 | v6 |
| `gradle/actions/setup-gradle` | v4 | v6 |
| `actions/upload-artifact` | v4 | v7 |

설정해 둔 대로 PR 제목은 `ci: bump the actions group across 1 directory with 4 updates`였고, `ci` 라벨이 붙었고, 대상은 `develop`이었다. 브랜치 이름(`dependabot/github_actions/...`)도 미리 허용해 둔 덕분에 `branch-flow` 검사를 통과했다.

Gradle 쪽 검사도 돌았지만 PR은 열리지 않았다. Spring Boot 4.1.1이 관리하는 의존성들이 이미 최신이었다는 뜻이다.

## PR #43은 왜 닫혔나

1. 처음 열린 PR #43은 3개 액션(checkout, setup-java, setup-gradle)만 올렸다.
2. 그 직후에 `upload-artifact@v4`를 쓰는 PR #44를 develop에 머지했다.
3. #43이 develop보다 뒤처져서, `@dependabot rebase` 댓글로 최신 develop에 맞춰 달라고 요청했다.
4. Dependabot의 답은 이랬다.

   > Looks like these dependencies are updatable in another way, so this is no longer needed.

5. Dependabot은 #43을 닫고, 새로 생긴 `upload-artifact`까지 포함한 **4개짜리 그룹 PR #45**를 새로 열었다.

그룹 업데이트(`groups`)를 쓰면 Dependabot은 "지금 올릴 수 있는 것 전체"를 PR 하나로 만든다. 그룹 구성이 바뀌면 기존 PR을 rebase하지 않고 새 PR로 갈아 끼운다.

## 메이저 버전을 올려도 되는지 확인한 방법

메이저 버전이 한 번에 2~3단계씩 올라가서 그대로 머지하기 전에 확인했다.

- 워크플로 파일을 바꾸는 PR은 **바뀐 워크플로로 CI가 돈다.** #45의 CI가 새 버전으로 Java 25 설치, Gradle 빌드, 테스트를 모두 통과했다 (`build` 58초).
- 새 `upload-artifact@v7`로도 `test-reports`(116KB)가 실제로 올라간 것을 API로 확인했다.

```
$ gh api repos/.../actions/runs/<run id>/artifacts --jq '.artifacts[] | "\(.name) \(.size_in_bytes)B"'
test-reports 116182B
```

확인을 마친 뒤 merge commit 방식으로 머지했다.

## 다음 할 일

- 개발 착수 전 결정 사항(기술 스택, 아키텍처, WBS) 논의
