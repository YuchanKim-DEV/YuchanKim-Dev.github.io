---
title: "[LIMITED_DEALS] 프로젝트 세팅 - Spring Boot 4.1.1, 의존성, 그리고 JDK 21을 두고 이 프로젝트만 25로 빌드하기"
date: 2026-10-12 14:50:00 +0900
categories: [SIDE PROJECTS, LIMITED_DEALS]
tags: [spring-boot, java, jdk, gradle, toolchain, jvm, bytecode, vscode, homebrew]
---

백엔드 면접 준비용으로 **LIMITED_DEALS**(한정 특가) 프로젝트를 시작했다.
빈 껍데기에서 시작해서 개념을 하나씩 얹는 학습용 프로젝트라서, 첫 글은 **기본 세팅**이다.
어떤 버전과 의존성으로 시작했는지, JDK 버전이 안 맞았던 문제를 어떻게 풀었는지, 그리고 세팅하면서 새로 알게 된 것들을 남긴다.

## 환경

| 항목 | 버전 / 도구 |
|------|------|
| 에디터 | VS Code |
| Spring Boot | 4.1.1 |
| Gradle (wrapper) | 9.7.1 (Kotlin DSL) |
| Java (프로젝트 요구) | 25 |
| Java (맥에 기본으로 설치된 것) | 21.0.11 (Homebrew OpenJDK) |

프로젝트는 **Spring Initializr**로 만들었다. 그래서 클래스는 `LimitedDealProductsApplication`과 테스트 하나뿐인 뼈대 상태이고, 루트 패키지는 `com.limiteddealproducts`다.
작업 환경은 IntelliJ가 아니라 **VS Code**다.

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
MySQL 드라이버는 아직 없고, `application.yaml`에도 DB 접속 정보가 없다. DB 설정은 다음 단계에서 정한다.

## 새로 배운 것 1: 컴파일러는 기계어가 아니라 바이트코드를 만든다

처음에는 "컴파일러가 코드를 맥 OS가 알아듣는 컴퓨터 언어로 바꾸는 것"이라고 이해했다. C 같은 언어는 그 말이 맞지만, 자바는 한 단계가 더 있다.

```
Hello.java --javac(컴파일러)--> Hello.class(바이트코드) --JVM--> 맥/윈도우/리눅스에서 실행
```

`javac`가 만드는 건 특정 OS용 기계어가 아니라 **바이트코드**(`.class`)다. 맥도 윈도우도 이걸 직접 못 알아듣고, **JVM**이 현재 OS와 CPU에 맞게 번역해서 실행한다.

| 이름 | 뜻 | 역할 |
|---|---|---|
| **JVM** | Java Virtual Machine | 바이트코드를 현재 OS/CPU에 맞게 실행하는 가상 머신 |
| **JRE** | Java Runtime Environment | JVM + 표준 라이브러리 (실행만 가능) |
| **JDK** | Java Development Kit | JRE + 컴파일러(`javac`) 등 개발 도구. 개발자는 이걸 설치한다 |

포함 관계는 **JDK ⊃ JRE ⊃ JVM**이다. "JDK 25를 쓴다"는 말은 곧 "JVM 25를 쓴다"는 말이기도 하다.

### OS가 달라도 JVM만 다르다
같은 `.class` 파일을 쓰고, OS와 CPU에 맞는 JVM만 따로 설치한다. 맥(Apple Silicon)용, 윈도우용, 리눅스용 JDK를 따로 받는 이유다. 그래서 맥북에서 만든 앱을 리눅스 서버에 올려도 다시 컴파일할 필요가 없다("Write Once, Run Anywhere").

다만 바이트코드에는 **컴파일한 자바 버전**이 들어 있다. Java 25로 컴파일한 `.class`를 Java 21 JVM에서 실행하면 `UnsupportedClassVersionError`가 난다. 낮은 버전으로 컴파일한 걸 높은 버전 JVM에서 돌리는 건 대체로 된다.

## 문제: 요구는 25인데 설치된 건 21

`build.gradle.kts`에는 이렇게 되어 있었다.

```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}
```

그런데 터미널에서 확인해 보니 기본 java는 **21**이었다.

```bash
$ /usr/libexec/java_home -V
21.0.11 (arm64) "Homebrew" - "OpenJDK 21.0.11" ...
```

JDK 25는 Homebrew로 이미 설치되어 있었다(`/opt/homebrew/opt/openjdk@25`). 그런데 `java_home -V` 목록에는 안 보였다.
Homebrew로 설치한 JDK는 macOS가 JDK를 찾는 폴더(`/Library/Java/JavaVirtualMachines`)에 자동 등록되지 않기 때문이다.

## 새로 배운 것 2: 기본 java와 프로젝트의 JDK는 따로 쓸 수 있다

터미널에서 `java`를 치면 나오는 건 **PATH에 먼저 잡힌 JDK**(여기서는 21)다. 반면 Gradle `toolchain`은 "이 프로젝트는 25로 빌드해라"라는 요청이라서, 기본 `java`와 상관없이 25를 직접 찾아서 컴파일과 실행에 쓴다.

그래서 **다른 프로젝트는 21 그대로 두고, 이 프로젝트만 25**로 쓸 수 있다. 시스템 기본을 바꿀 필요가 없다. 문제는 Gradle이 25를 못 찾는 것뿐이었다.

## 새로 배운 것 3: 심볼릭 링크

해결 방법은 두 가지였다.

| 방법 | 특징 |
|------|------|
| `sudo ln -s`로 macOS JDK 폴더에 심볼릭 링크 | 시스템 전체에 등록됨, sudo 필요 |
| 프로젝트의 `gradle.properties`에 경로 지정 | 이 프로젝트에만 적용, sudo 불필요 |

**심볼릭 링크**는 파일이나 폴더의 바로가기다. 실제 파일은 한 곳에만 있고, 다른 위치에 "저기 있어요"라고 가리키는 표지판만 만든다.

```
/Library/Java/JavaVirtualMachines/openjdk-25.jdk   ← 링크 (표지판)
        └──▶ /opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk   ← 실제 JDK
```

링크를 만들면 macOS의 JDK 목록에 25가 "등록"될 뿐이다. 그래서 `java_home -V`에는 보이게 되고 Gradle도 찾을 수 있지만, **`PATH`와 `JAVA_HOME`은 바뀌지 않아서** 터미널의 기본 `java`는 계속 21이다.

| | 링크를 만들면 |
|---|---|
| `java_home -V` 목록에 25 표시 | 바뀜 |
| Gradle이 25를 찾을 수 있음 | 바뀜 |
| 터미널 기본 `java` 버전 | 안 바뀜 |
| `JAVA_HOME` 환경변수 | 안 바뀜 |

## 해결: gradle.properties에 JDK 경로 알려주기

"이 프로젝트에서만"이 목적이고 sudo도 피하고 싶어서 두 번째 방법을 골랐다. 프로젝트 루트(`build.gradle.kts`와 같은 위치)에 `gradle.properties`를 만들었다.

```properties
org.gradle.java.installations.paths=/opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk/Contents/Home
```

"JDK를 찾을 때 이 경로도 봐라"라는 뜻이다. `toolchain`이 25를 요청하면 Gradle이 이 경로에서 찾아온다.

### 확인

```bash
$ ./gradlew javaToolchains
 + Homebrew JDK 21 (21.0.11)
 + Homebrew JDK 25 (25.0.4.1)     # 이제 25가 보인다

$ ./gradlew compileJava
BUILD SUCCESSFUL
```

컴파일 중에 Lombok이 `sun.misc.Unsafe` 경고를 출력한다. Lombok이 JDK 25에서 내부 API를 쓰기 때문인데, 빌드는 성공하니 지금은 무시한다.
이 변경은 GitHub 컨벤션대로 별도 브랜치(`build/jdk25-toolchain/2026-10-12`)와 PR로 올렸다. Gradle 설정은 문서 변경과 성격이 달라서 따로 나눴다.

## 새로 배운 것 4: VS Code의 "Null annotation types" 팝업

VS Code로 프로젝트를 열자 이런 팝업이 떴다.

> Null annotation types have been detected in the project. Do you wish to enable null analysis for this project?

VS Code의 Java 확장이 프로젝트에서 `@Nullable`, `@NonNull` 같은 **null 어노테이션**을 발견하고, "이걸 기준으로 null 가능성을 에디터에서 검사할까요?"라고 묻는 것이다. Spring Boot 4가 쓰는 Spring Framework 7이 JSpecify라는 null 어노테이션 표준을 쓰기 때문에 감지됐다.

| 선택 | 결과 |
|---|---|
| Yes | `null`일 수 있는 값을 null 체크 없이 쓰면 에디터에 경고가 뜬다 (컴파일/실행에는 영향 없음) |
| No | 지금과 똑같이 동작하고 경고만 안 뜬다 |

NPE를 미리 잡는 습관을 들이려면 Yes가 좋고, 경고가 거슬리면 No로 해도 된다. 설정에서 나중에 바꿀 수 있다.
나는 **Yes**를 선택하기로 했다.

## 정리

- 기본 `java`는 21 그대로, 이 프로젝트만 `toolchain` + `gradle.properties`로 25를 사용한다.
- `javac`는 바이트코드를 만들고, OS마다 JVM만 다르다. 바이트코드의 자바 버전보다 낮은 JVM에서는 실행되지 않는다.
- 심볼릭 링크는 바로가기이고, 만들어도 `PATH`와 `JAVA_HOME`은 바뀌지 않는다.
- `gradle.properties`의 경로는 내 맥북 기준이라, 다른 환경에서는 경로를 고쳐야 한다.
- 다음은 GitHub 컨벤션을 정하고, 이후 DB 설정과 첫 도메인(상품, 특가)으로 넘어간다.
