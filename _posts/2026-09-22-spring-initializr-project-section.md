---
title: "Spring Initializr의 Project 섹션 - 빌드 도구가 하는 일과 Maven vs Gradle"
date: 2026-09-22 20:00:00 +0900
categories: [SpringBoot, 기초]
tags: [spring-boot, gradle, maven, build-tool, java]
---

start.spring.io에서 프로젝트를 만들 때 가장 먼저 나오는 **Project** 섹션이 뭘 정하는 건지 헷갈려서 정리한다.
"Gradle - Groovy / Gradle - Kotlin / Maven" 중에 뭘 골라야 하는지가 궁금했다.

## 빌드 도구가 뭔지부터

요리를 한다고 치면, 재료 사 오고, 손질하고, 조리하고, 그릇에 담는 과정이 있다.
**빌드 도구는 이 전체 과정을 자동으로 해주는 로봇**이다.

```
재료 사 오기   → 필요한 라이브러리 인터넷에서 받아오기
손질하기      → 코드 컴파일 (Java → 실행 가능한 형태로 변환)
조리하기      → 테스트 실행
그릇에 담기   → 실행 파일(.jar)로 완성
```

이 로봇이 없으면 이 과정을 전부 손으로 해야 한다. 라이브러리도 하나하나 받아서 넣고,
버전도 직접 맞춰야 한다. 그걸 자동으로 해주는 게 **Maven**과 **Gradle**이고,
둘 다 같은 일을 하는 서로 다른 브랜드의 로봇이라고 생각하면 된다.

## Maven vs Gradle — 로봇 조작 방식 차이

**Maven**은 정해진 양식지에 체크만 하는 로봇이다. "이 재료 필요함"이라고 종이(XML)에 적으면 끝이다.
대신 정해진 양식대로만 써야 해서 응용이 어렵다.

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**Gradle**은 직접 명령어를 짜서 시키는 로봇이다. "이거 필요하고, 만약 이런 상황이면 이렇게 해줘" 같은 걸
프로그래밍 언어로 짤 수 있다. 더 똑똑하고 빠른 대신, 다루는 법을 좀 더 배워야 한다.

```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

| | Maven | Gradle |
|---|---|---|
| 설정 문법 | XML (정해진 양식지) | 스크립트 (직접 명령어를 짜는 방식) |
| 빌드 속도 | 느림 (매번 전체 재빌드 경향) | 빠름 (증분 빌드, 캐싱) |
| 유연성 | 낮음 (선언적, 정형화) | 높음 (코드처럼 커스터마이징 가능) |
| 러닝 커브 | 낮음 (구조가 정형화돼 있어 읽기 쉬움) | 상대적으로 높음 |
| 업계 비중 | 여전히 많음 (특히 오래된/대기업 프로젝트) | 최신 프로젝트 다수, 요즘 대세 |

정답은 없다. 다만 요즘 신규 프로젝트는 Gradle 비중이 높은 편이고, 속도와 유연성 때문에 대세로 자리 잡는 중이다.

## Gradle - Groovy vs Gradle - Kotlin — 로봇에게 말 거는 언어

Gradle이라는 로봇은 똑같은데, **그 로봇한테 명령어를 어떤 말투로 적을지**의 차이다.
빌드 도구 선택이 아니라 같은 Gradle의 설정 파일을 어떤 언어로 쓸지의 문제다.

```groovy
// build.gradle (Groovy DSL)
implementation 'org.springframework.boot:spring-boot-starter-web'
```

```kotlin
// build.gradle.kts (Kotlin DSL)
implementation("org.springframework.boot:spring-boot-starter-web")
```

괄호 하나 정도의 차이다. 로봇이 알아듣는 내용은 똑같지만, Kotlin DSL이 IDE 자동완성과 타입 체크가 더 잘 된다.
다만 아직은 예제와 문서 대부분이 Groovy 기준이라, **처음 배울 땐 인터넷에 예제가 많은 Groovy가 편하다.**

## 정리

- **Project**에서 고르는 건 "빌드 도구(Maven/Gradle)"와 "그 설정 파일 문법(Groovy/Kotlin)"이다
- 빌드 도구는 라이브러리 받기 → 컴파일 → 테스트 → 패키징까지 전 과정을 자동으로 해주는 로봇 같은 것이다
- Maven은 정해진 양식(XML)에 채우는 방식, Gradle은 스크립트로 직접 짜는 방식이라 더 유연하고 빠르다
- Gradle 안에서 Groovy/Kotlin은 로봇에게 말 거는 언어(문법) 차이일 뿐, 로봇 자체(빌드 도구)는 동일하다
- 처음 배울 때는 예제가 제일 많은 **Gradle - Groovy + Java** 조합이 무난하다

다음은 Spring Boot 버전 선택과 Java 버전 호환성을 정리한다.
