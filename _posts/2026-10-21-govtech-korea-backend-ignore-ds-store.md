---
title: "[BACKEND] .DS_Store를 gitignore에 - 폴더를 옮기자 보이기 시작한 파일"
date: 2026-10-21 14:28:55 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [gitignore, macos, git]
---

저장소 폴더를 옮긴 뒤 `git status`에 처음 보는 파일들이 잡혔다 ([PR #57](https://github.com/GovTech-korea/GovTech-Korea_Backend/pull/57), 이슈 #55).

```
?? .DS_Store
?? .github/.DS_Store
?? src/.DS_Store
?? src/main/.DS_Store
```

## .DS_Store가 뭔가

macOS Finder가 폴더를 열 때 만드는 숨김 파일이다. 아이콘 위치, 보기 방식 같은 Finder 설정이 들어 있다. 프로젝트와는 아무 관계가 없는데, Finder로 폴더를 열 때마다 생기기 때문에 그대로 두면 언젠가 `git add .`에 섞여 커밋된다. 이번에 Finder로 폴더를 옮기면서 여러 폴더에 한꺼번에 생겼다.

## 변경 내용

```gitignore
### macOS ###
.DS_Store
```

Spring Initializr가 만들어 준 `.gitignore`에는 IDE 관련 항목(IntelliJ, VS Code, STS 등)만 있고 OS 파일은 없었다. 추가한 뒤 `git status`가 다시 깨끗해졌다.
