---
title: "Spring Initializr의 Project 섹션 - 빌드 도구가 하는 일과 Maven vs Gradle"
date: 2026-09-22 20:00:00 +0900
categories: [SpringBoot, 기초]
tags: [spring-boot, gradle, maven, build-tool, java]
---

start.spring.io에서 프로젝트를 만들 때 가장 먼저 나오는 **Project** 섹션을 정리한다.
"Gradle - Groovy / Gradle - Kotlin / Maven" 중에 뭘 골라야 하는지 헷갈려서 짚어본다.

## Project 섹션이 정하는 것

이 섹션은 두 가지를 고르는 자리다.

```
Gradle - Groovy   ┐
Gradle - Kotlin   ┼─ 빌드 도구는 둘 다 Gradle, 설정 파일 문법만 다름
Maven             ┘─ 완전히 다른 빌드 도구
```

**빌드 도구**를 고르는 거지, 프레임워크나 언어를 고르는 게 아니다. Language(Java/Kotlin/Groovy)는 완전히 별개 항목이다.

## 빌드 도구가 하는 일

라이브러리 버전 관리만 하는 줄 알았는데, 실제로는 네 단계를 전부 담당한다.

1. **의존성 관리** — `spring-boot-starter-web` 같은 라이브러리를 어디서 받아올지, 버전 충돌은 어떻게 해결할지
2. **컴파일** — `.java` → `.class`
3. **테스트 실행**
4. **패키징** — 실행 가능한 `.jar`(또는 `.war`)로 묶기

`./gradlew build` 한 줄이 이 네 단계를 순서대로 실행하는 것이다.

## Maven vs Gradle

같은 일을 하는 도구인데 설정 방식의 철학이 다르다.

**Maven**은 XML로 "뭘 원하는지"만 선언한다.

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**Gradle**은 실제 스크립트 언어(Groovy 또는 Kotlin)로 작성하기 때문에, 조건문·반복문 같은 로직도 넣을 수 있다.

```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

| | Maven | Gradle |
|---|---|---|
| 설정 문법 | XML | Groovy/Kotlin 스크립트 |
| 빌드 속도 | 느림 (매번 전체 재빌드 경향) | 빠름 (증분 빌드, 캐싱) |
| 유연성 | 낮음 (선언적, 정형화) | 높음 (코드처럼 커스터마이징 가능) |
| 러닝 커브 | 낮음 (구조가 정형화돼 있어 읽기 쉬움) | 상대적으로 높음 |
| 업계 비중 | 여전히 많음 (특히 오래된/대기업 프로젝트) | Android 공식 표준, 최신 프로젝트 다수 |

정답은 없다. 최근 신규 프로젝트나 스타트업 쪽은 Gradle 비중이 높은 편이고, 대기업 레거시는 Maven이 아직 많다.

## Gradle - Groovy vs Gradle - Kotlin

이건 빌드 도구 선택이 아니라, **같은 Gradle의 설정 파일을 어떤 언어로 쓸지**의 차이다.

```groovy
// build.gradle (Groovy DSL)
plugins {
    id 'java'
}
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

```kotlin
// build.gradle.kts (Kotlin DSL)
plugins {
    java
}
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
}
```

Kotlin DSL이 IDE 자동완성과 타입 체크가 더 잘 되지만, 아직은 예제와 문서 대부분이 Groovy 기준이라 입문 단계에서는 Groovy 쪽이 참고 자료를 찾기 쉽다.

## 정리

- **Project**에서 고르는 건 "빌드 도구(Maven/Gradle)"와 "그 설정 파일 문법(Groovy/Kotlin)"이다
- Maven은 XML 선언형, Gradle은 스크립트형이라 더 유연하고 빌드도 빠르다
- Gradle 안에서 Groovy/Kotlin은 문법 취향 차이일 뿐, 빌드 도구 자체는 동일하다
- 처음 배울 때는 예제가 제일 많은 **Gradle - Groovy + Java** 조합이 무난하다

다음은 Spring Boot 버전 선택과 Java 버전 호환성을 정리한다.
