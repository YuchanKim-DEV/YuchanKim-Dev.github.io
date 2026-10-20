---
title: "[BACKEND] CI 테스트·커버리지 리포트를 artifact로 - 실패해도 올리기"
date: 2026-10-21 02:29:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [github-actions, ci, artifact, jacoco, test-report]
---

PR마다 도는 CI의 빌드 결과에서 테스트 리포트와 커버리지 리포트를 내려받을 수 있게 했다 ([PR #44](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/44), 이슈 #16).

## 왜 필요한가

CI에서 테스트가 실패하면 지금은 Actions 로그만 볼 수 있다. 테스트 실패 원인은 콘솔에 찍도록 해 뒀지만([테스트 환경 구성 글]({% post_url 2026-10-21-govtech-korea-backend-test-environment %})), 테스트가 많아지면 HTML 리포트로 보는 편이 훨씬 낫다. 커버리지 리포트는 로그로는 아예 볼 수 없다.

## 변경 내용

```yaml
- name: 빌드 및 테스트
  run: ./gradlew build
- name: 테스트·커버리지 리포트 업로드
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: test-reports
    path: |
      build/reports/tests/
      build/reports/jacoco/
    if-no-files-found: ignore
    retention-days: 14
```

| 설정 | 이유 |
|---|---|
| `if: always()` | 기본값은 앞 단계가 성공해야 실행된다. 리포트가 가장 필요한 순간은 테스트가 **실패했을 때**라서 항상 실행 |
| `if-no-files-found: ignore` | 컴파일 단계에서 실패하면 리포트 자체가 없다. 이때 업로드 단계까지 경고를 내지 않게 |
| `retention-days: 14` | PR 확인 용도라 2주면 충분하다. 저장 공간도 아낀다 |

## 확인

이 PR의 CI 실행에서 artifact가 실제로 만들어졌는지 API로 확인했다.

```
$ gh api repos/.../actions/runs/<run id>/artifacts --jq '.artifacts[] | "\(.name) \(.size_in_bytes)B expires \(.expires_at)"'
test-reports 116156B expires 2026-11-03T17:26:18Z
```

PR의 Checks 탭 → 해당 실행 → 아래쪽 Artifacts에서 zip으로 내려받을 수 있다.

## 다음 할 일

- Dependabot이 연 첫 PR(Actions 메이저 버전 업데이트) 검증
