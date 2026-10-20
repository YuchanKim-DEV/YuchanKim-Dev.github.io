---
title: "[BACKEND] README는 실행 가이드로 - 적은 명령은 전부 직접 돌려 보기"
date: 2026-10-21 02:16:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [readme, documentation, gradle, toolchain, lombok]
---

README를 다시 썼다 ([PR #39](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/39)).
처음 README에는 브랜치·커밋·PR 규칙 요약이 들어 있었는데, 저장소를 처음 내려받은 사람에게 먼저 필요한 건 규칙이 아니라 **실행 방법**이다. 규칙은 별도 문서 저장소(DEV_DOCUMENTS)로 옮길 예정이라 README에서는 뺐다.

## 구성

| 절 | 내용 |
|---|---|
| 필요한 것 | JDK 25, Git. Gradle은 Wrapper를 쓰므로 설치 불필요 |
| 내려받기 | clone 후 `develop` 브랜치로 전환 |
| JDK 25 설정 | 설치 → `./gradlew -q javaToolchains`로 확인 → 안 보이면 `~/.gradle/gradle.properties`에 경로 지정 |
| 실행 | `./gradlew bootRun` |
| 빌드와 테스트 | 명령어, 테스트·커버리지 리포트와 jar 위치 |
| 설정 | `application.yaml` 위치, 비밀값은 환경변수로 |
| IDE | IntelliJ·VS Code의 Lombok, JDK 설정 |

## README에 적은 명령은 전부 직접 돌려 봤다

문서에 적힌 명령이 실제로 안 되면 처음 온 사람이 바로 막힌다. 그래서 적은 내용을 하나씩 확인했다.

```bash
$ ls -d /opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk/Contents/Home   # 예시 경로 존재
$ ./gradlew bootRun
Tomcat started on port 8080
Started GovtechApplication in 0.784 seconds
$ curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/
404
```

아직 API가 하나도 없어서 루트 경로는 404다. 이것도 README에 그대로 적었다. 처음 실행한 사람이 404를 보고 "실행이 잘못됐나" 헷갈리지 않게 하기 위해서다.

빌드 결과물 위치도 확인했다.

```
build/reports/tests/test/index.html          # 테스트 리포트
build/reports/jacoco/test/html/index.html    # 커버리지 리포트
build/libs/govtech-0.0.1-SNAPSHOT.jar        # 실행 jar
build/libs/govtech-0.0.1-SNAPSHOT-plain.jar  # 의존성 없는 jar
```

`-plain.jar`는 Spring Boot Gradle 플러그인이 일반 `jar` 태스크로 만드는 파일이다. 의존성이 들어 있지 않아 `java -jar`로 실행할 수 없다. 실행용은 `bootJar`가 만든 파일 쪽이다.

## 다음 할 일

- 컴파일할 때 나오는 Lombok 경고 정리
- Dependabot, CI 커버리지 리포트, `.editorconfig`
