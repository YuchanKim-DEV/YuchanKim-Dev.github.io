---
title: "[BACKEND] .editorconfig - 새 규칙이 아니라 지금 형식을 고정하기"
date: 2026-10-21 02:22:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [editorconfig, code-style, gitattributes, ide]
---

IntelliJ, VS Code처럼 에디터가 달라도 같은 형식으로 저장되도록 `.editorconfig`를 추가했다 ([PR #41](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/41), 이슈 #17).

## 먼저 지금 형식을 조사했다

규칙을 새로 정하면 기존 파일과 어긋나서 "포맷만 바뀐" 큰 diff가 생긴다. 그래서 파일마다 들여쓰기와 줄바꿈을 먼저 셌다.

| 파일 | 탭으로 시작하는 줄 | 공백으로 시작하는 줄 | CRLF |
|---|---|---|---|
| `build.gradle` | 39 | 0 | 0 |
| `GovtechApplication.java` | 3 | 0 | 0 |
| `application.yaml` | 0 | 2 | 0 |
| `pr-check.yml` | 0 | 32 | 0 |
| `gradlew.bat` | 0 | 0 | 112 |

Spring Initializr가 만든 Java·Gradle 파일은 **탭**, YAML은 문법상 **공백**, Windows 배치 파일만 **CRLF**였다. 이걸 그대로 규칙으로 옮겼다.

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{java,gradle}]
indent_style = tab
indent_size = 4

[*.{yml,yaml,json}]
indent_style = space
indent_size = 2

[*.md]
indent_style = space
indent_size = 2
trim_trailing_whitespace = false

[*.bat]
end_of_line = crlf
```

기존 파일 중 이 규칙을 어기는 파일은 없어서 재포맷 커밋도 필요 없었다.

## 새로 배운 것: .editorconfig와 .gitattributes는 하는 일이 다르다

둘 다 줄바꿈(LF/CRLF)을 다루지만 동작하는 시점이 다르다.

| | `.editorconfig` | `.gitattributes` |
|---|---|---|
| 누가 읽나 | 에디터, IDE | Git |
| 언제 | 파일을 **저장할 때** | 파일을 **커밋·체크아웃할 때** |
| 이 프로젝트 | `.bat`은 CRLF로 저장 | `*.bat text eol=crlf` |

두 파일의 규칙이 서로 다르면 저장할 때와 커밋할 때 줄바꿈이 계속 바뀐다. 그래서 `.bat`의 CRLF 규칙을 `.gitattributes`와 맞췄다.

Markdown만 줄 끝 공백 제거에서 뺀 것도 이유가 있다. Markdown에서 줄 끝 공백 두 칸은 줄바꿈(`<br>`)이라는 뜻이라, 지우면 렌더링 결과가 바뀐다.

## 다음 할 일

- Dependabot, CI 커버리지 리포트 업로드
