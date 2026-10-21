---
title: "[BACKEND] GitHub 저장소 이름 바꾸기 - 바꿔야 할 곳 찾기"
date: 2026-10-21 14:29:55 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [github, repository, rename]
---

저장소 이름을 `GovTech-Korea_backend`에서 `GovTech-Korea_Backend`로 바꿨다. 앞으로 만들 게이트웨이 저장소는 `GovTech-Korea_VoiceGateway`로 이름 규칙을 맞춘다 ([PR #59](https://github.com/GovTech-korea/GovTech-Korea_Backend/pull/59), 이슈 #58).

## 이름은 한 줄이면 바뀐다

```bash
gh repo rename GovTech-Korea_Backend -R GovTech-korea/GovTech-Korea_backend
```

GitHub은 예전 이름으로 들어오는 요청(웹 주소, git clone·push)을 새 이름으로 자동으로 넘겨 준다. 그래서 이름을 바꿔도 당장 깨지는 건 없다. 대신 예전 이름으로 **새 저장소를 만들면** 이 넘겨주기가 끊긴다. 그래서 남은 참조는 바로 새 이름으로 맞추는 편이 안전하다.

## 바꿔야 할 곳

| 위치 | 처리 |
|---|---|
| 로컬 원격 주소 | `git remote set-url origin https://github.com/GovTech-korea/GovTech-Korea_Backend.git` |
| README의 clone 명령 | 이번 PR |
| 저장소 이름을 직접 쓰는 스크립트 (Projects 동기화, 보호 규칙 설정) | 새 이름으로 수정 |
| 저장소 설정 (ruleset, CI, Projects, 이슈·PR) | 그대로 유지됨 (이름이 아니라 내부 ID로 연결) |
| 블로그의 예전 글 링크 | 리다이렉트로 동작해 그대로 둠 |

README는 고친 뒤 새 주소로 실제로 clone해서 명령이 그대로 동작하는지 확인했다.
