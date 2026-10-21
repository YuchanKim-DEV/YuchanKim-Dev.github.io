---
title: "[BACKEND] 저장소를 한 단계 아래로 - BACKEND와 VOICE_GATEWAY를 나란히 두기"
date: 2026-10-21 14:27:55 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [gradle, project-structure, git, vscode]
---

이 서비스에는 백엔드 말고도 **음성 연계 게이트웨이**(보고서의 MOMIN Connector)가 필요하다. 기관의 기존 전화 시스템(PBX, CTI)에서 통화 음성과 이벤트를 받아 백엔드로 넘기는 역할이다. 성격이 달라서 별도 저장소로 두기로 하고, 로컬 폴더 구조부터 바꿨다 ([PR #56](https://github.com/GovTech-korea/GovTech-Korea_Backend/pull/56), 이슈 #54).

## 폴더 구조

```
GOV_TECH/
├── BACKEND/          ← 이 저장소 (GovTech-Korea_Backend)
└── VOICE_GATEWAY/    ← 예정 (GovTech-Korea_VoiceGateway)
```

저장소 폴더를 통째로 `GOV_TECH/BACKEND`로 옮겼다. `.git` 폴더가 함께 옮겨지므로 커밋 기록, 브랜치, 원격 설정, 로컬 git hook이 모두 그대로 동작한다.

## 변경 내용

Gradle 루트 프로젝트 이름도 바꿨다.

```groovy
// settings.gradle
rootProject.name = 'govtech_backend'   // 전: 'govtech'
```

결과물 jar 이름이 `govtech_backend-0.0.1-SNAPSHOT.jar`로 바뀐다. `spring.application.name`(govtech)은 애플리케이션 이름이라 그대로 뒀다. 빌드 프로젝트 이름과 애플리케이션 이름은 별개다.

GitHub 저장소 이름도 `GovTech-Korea_backend`에서 `GovTech-Korea_Backend`로 바꿨다. GitHub은 이름을 바꿔도 예전 주소를 새 주소로 자동으로 넘겨 준다. 그래도 로컬 `origin` 주소와 README는 새 이름으로 맞췄다.

## 시행착오: VS Code가 예전 프로젝트를 기억한다

폴더를 옮기자 VS Code에 이런 오류가 떴다.

```
Project 'GOV_TECH' is missing required source folder: 'src/main/java'
The project cannot be built until build path errors are resolved
```

VS Code의 Java 확장은 내부에 Eclipse 방식의 작업 공간 캐시를 둔다. 캐시를 열어 보니 프로젝트가 둘 들어 있었다.

```
GOV_TECH                  ← 옮기기 전 위치의 프로젝트 (src가 없어져서 오류)
govtech_backend-BACKEND   ← 새 위치에서 다시 가져온 프로젝트
```

코드 문제가 아니라 캐시 문제라서, Java 확장의 작업 공간 캐시를 비우고 저장소 폴더(`BACKEND`)를 기준으로 다시 열면 된다.

## 다음 할 일

- VOICE_GATEWAY 기술 스택 정하기
