---
title: "[BACKEND] README와 PR·이슈 템플릿 - 그리고 JDK 25는 어디서 찾았나"
date: 2026-10-21 01:50:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [github, readme, pull-request-template, issue-template, gradle, toolchain]
---

저장소가 public이 되었으니 첫 화면을 정리했다. README, PR 템플릿, 이슈 템플릿을 추가했다 ([PR #2](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/2)).
README를 쓰다가 "JDK 25가 없어도 빌드된다"는 문장이 사실인지 확인했는데, 사실이 아니었다.

## 변경 내용

| 파일 | 내용 |
|---|---|
| `README.md` | 프로젝트 소개, 기술 스택, 실행 방법, 브랜치·커밋·PR 규칙 요약 |
| `.github/pull_request_template.md` | 변경 내용 / 테스트 결과 / 관련 이슈 |
| `.github/ISSUE_TEMPLATE/feature.md` | 제목 `[feat] `, 라벨 `feat` 자동 지정 |
| `.github/ISSUE_TEMPLATE/bug.md` | 제목 `[fix] `, 라벨 `fix` 자동 지정 |

커밋은 성격 규칙대로 셋으로 나눴다. 셋 다 `docs` 타입이지만 목적이 달라서다.

```
docs: README 추가
docs: PR 템플릿 추가
docs: 이슈 템플릿 추가
```

PR 템플릿의 테스트 항목은 컨벤션의 "push 전에 테스트하고 결과를 PR에 남긴다" 규칙을 그대로 옮긴 것이다.

```
## 테스트
- 실행: ./gradlew build
- 결과: 통과 (전체 N개, 실패 0개)
- 추가·수정한 테스트:
```

이슈 라벨도 기본 라벨(`bug`, `enhancement` 등)을 지우고, 브랜치 접두사와 같은 14개(`feat`, `fix`, `refactor`, `ci` …)로 바꿨다. 이슈, 브랜치, 커밋, 라벨이 모두 같은 단어를 쓴다.

## 새로 배운 것: Gradle toolchain은 JDK를 어떻게 찾나

`build.gradle`은 Java 25를 요구한다.

```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}
```

그런데 `/usr/libexec/java_home -V`로 보면 맥에 등록된 JDK는 21 하나뿐이다. 그래도 빌드가 됐다. 그래서 README에 "JDK 25가 없어도 toolchain이 알아서 찾는다"고 쓰려다가, 실제로 무엇을 찾고 있는지 확인했다.

```
$ ./gradlew -q javaToolchains

 + Homebrew JDK 21 (21.0.11)
     | Detected by:        Current JVM
 + Homebrew JDK 25 (25.0.4.1)
     | Detected by:        Gradle property 'org.gradle.java.installations.paths'
```

JDK 25는 Homebrew로 설치돼 있었다. 다만 맥의 JDK 목록에는 없어서 Gradle이 스스로 찾지 못한다. 대신 `~/.gradle/gradle.properties`의 `org.gradle.java.installations.paths`에 경로를 적어 둔 덕분에 찾고 있었다.

`Auto-download: Enabled`라고 나오지만, 실제로 내려받으려면 foojay 같은 다운로드 저장소 플러그인이 `settings.gradle`에 있어야 한다. 이 프로젝트에는 없다. 그래서 README에는 이렇게 고쳐 적었다.

> JDK 25가 설치되어 있어야 합니다. 기본 JDK가 다른 버전이어도 Gradle toolchain이 설치된 JDK 25를 찾아 빌드합니다.
> 찾지 못하면 `~/.gradle/gradle.properties`에 `org.gradle.java.installations.paths=<JDK 25 경로>`를 지정합니다.

CI 서버에는 이런 설정이 없으니, 워크플로에서 `actions/setup-java`로 25를 직접 설치한다.

## 함께 켠 GitHub 설정

PR과 별개로 저장소 설정도 정리했다.

- **secret scanning, push protection**: 토큰이나 키가 담긴 커밋은 push 단계에서 막힌다
- **Dependabot 보안 알림**: 의존성에 취약점이 생기면 알림
- **CodeQL**: 코드 정적 분석. 이번 PR부터 `CodeQL` 검사가 함께 돌았다
- 위키·프로젝트 탭 끄기, "Update branch" 버튼 켜기

## 다음 할 일

- 테스트 환경 구성 (JUnit, Mockito)
