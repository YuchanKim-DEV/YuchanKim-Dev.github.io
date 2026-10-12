---
title: "[LIMITED_DEALS] 프로젝트 세팅 - Spring Boot 4.1.1, 의존성, 그리고 JDK 21을 두고 이 프로젝트만 25로 빌드하기"
date: 2026-10-12 14:30:00 +0900
published: false
categories: [SIDE PROJECTS, LIMITED_DEALS]
tags: [spring-boot, java, jdk, gradle, toolchain, jvm, homebrew]
---

백엔드 면접 준비용으로 **LIMITED_DEALS**(한정 특가) 프로젝트를 시작했다.
빈 껍데기에서 시작해서 개념을 하나씩 얹는 학습용 프로젝트라서, 첫 글은 **기본 세팅**이다.
어떤 버전과 의존성으로 시작했는지, 그리고 JDK 버전이 안 맞았던 문제를 어떻게 풀었는지 남긴다.

## 환경

| 항목 | 버전 |
|------|------|
| Spring Boot | 4.1.1 |
| Gradle (wrapper) | 9.7.1 (Kotlin DSL) |
| Java (프로젝트 요구) | 25 |
| Java (맥에 기본으로 설치된 것) | 21.0.11 (Homebrew OpenJDK) |

프로젝트는 Spring Initializr로 만든 뼈대 상태다. 클래스는 `LimitedDealProductsApplication`과 테스트 하나뿐이고, 루트 패키지는 `com.limiteddealproducts`다.

## 의존성

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-h2console")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-webmvc")
    compileOnly("org.projectlombok:lombok")
    developmentOnly("org.springframework.boot:spring-boot-devtools")
    runtimeOnly("com.h2database:h2")
    runtimeOnly("org.postgresql:postgresql")
    annotationProcessor("org.projectlombok:lombok")
    // test 의존성은 위 starter에 대응하는 *-test starter들
}
```

| 의존성 | 용도 |
|--------|------|
| `starter-webmvc` | REST API용 Spring MVC (WebFlux 아님) |
| `starter-data-jpa` | JPA, Spring Data JPA |
| `starter-validation` | 요청 값 검증 (`@Valid`) |
| `starter-actuator` | 상태 확인, 모니터링 엔드포인트 |
| H2, `h2console` | 로컬 개발용 인메모리 DB와 웹 콘솔 |
| PostgreSQL 드라이버 | 운영 DB 후보 |
| Lombok | 보일러플레이트 코드 제거 |
| DevTools | 코드 변경 시 자동 재시작 |

버전은 Spring Boot가 한꺼번에 관리하기 때문에 `build.gradle.kts`에 적지 않는다.
참고로 MySQL 드라이버는 아직 없고, application 설정에도 DB 접속 정보가 없다. DB 설정은 다음 단계에서 정한다.

## JDK, JVM, JRE 정리

헷갈려서 먼저 정리했다.

- **JVM**: 바이트코드를 현재 OS와 CPU에 맞게 실행하는 가상 머신
- **JRE**: JVM + 표준 라이브러리 (실행만 가능)
- **JDK**: JRE + 컴파일러(`javac`) 등 개발 도구. 개발자는 JDK를 설치한다

`javac`는 맥용 기계어가 아니라 **바이트코드**(`.class`)를 만든다. 같은 `.class`를 맥, 윈도우, 리눅스의 JVM이 각자 번역해서 실행한다. 그래서 OS가 달라도 코드를 다시 컴파일하지 않는다.
다만 바이트코드에는 컴파일한 자바 버전이 들어 있어서, Java 25로 컴파일한 클래스를 Java 21 JVM에서 실행하면 `UnsupportedClassVersionError`가 난다.

## 문제: 요구는 25인데 설치된 건 21

`build.gradle.kts`에는 이렇게 되어 있었다.

```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}
```

그런데 터미널에서 `java -version`을 해보면 **21**이었다.

```bash
$ /usr/libexec/java_home -V
21.0.11 (arm64) "Homebrew" - "OpenJDK 21.0.11" ...
```

JDK 25는 Homebrew로 이미 설치되어 있었다(`/opt/homebrew/opt/openjdk@25`). 그런데 `java_home -V` 목록에는 안 보였다.
Homebrew로 설치한 JDK는 macOS가 JDK를 찾는 폴더(`/Library/Java/JavaVirtualMachines`)에 자동 등록되지 않기 때문이다.

### 시스템 기본 java를 바꿀 필요는 없다

`toolchain`은 "이 프로젝트는 25로 빌드해라"라는 요청이다. Gradle이 기본 `java`와 상관없이 25를 직접 찾아서 컴파일과 실행에 쓴다.
그래서 **다른 프로젝트는 그대로 21로 두고, 이 프로젝트만 25**로 쓸 수 있다. 문제는 Gradle이 25를 못 찾는 것뿐이다.

## 해결: gradle.properties에 JDK 경로 알려주기

해결 방법은 두 가지였다.

| 방법 | 특징 |
|------|------|
| `sudo ln -s`로 macOS JDK 폴더에 심볼릭 링크 | 시스템 전체에 등록됨, sudo 필요 |
| 프로젝트의 `gradle.properties`에 경로 지정 | 이 프로젝트에만 적용, sudo 불필요 |

**심볼릭 링크**는 파일이나 폴더의 바로가기다. 링크를 만들어도 `PATH`나 `JAVA_HOME`은 바뀌지 않아서 기본 `java`는 계속 21이다. 목록에 25가 "등록"될 뿐이다.

"이 프로젝트에서만"이 목적이라 두 번째 방법을 골랐다. 프로젝트 루트(`build.gradle.kts`와 같은 위치)에 `gradle.properties`를 만들었다.

```properties
org.gradle.java.installations.paths=/opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk/Contents/Home
```

"JDK를 찾을 때 이 경로도 봐라"라는 뜻이다.

### 확인

```bash
$ ./gradlew javaToolchains
 + Homebrew JDK 21 (21.0.11)
 + Homebrew JDK 25 (25.0.4.1)     # 이제 25가 보인다

$ ./gradlew compileJava
BUILD SUCCESSFUL
```

컴파일 중에 Lombok이 `sun.misc.Unsafe` 경고를 출력한다. Lombok이 JDK 25에서 내부 API를 쓰기 때문인데, 빌드는 성공하니 지금은 무시한다.

## 정리

- 기본 `java`는 21 그대로, 이 프로젝트만 `toolchain` + `gradle.properties`로 25를 사용한다.
- `gradle.properties`의 경로는 내 맥북 기준이라, 다른 환경에서는 경로를 고쳐야 한다.
- 다음은 GitHub 컨벤션을 정하고, 이후 DB 설정과 첫 도메인(상품, 특가)으로 넘어간다.
