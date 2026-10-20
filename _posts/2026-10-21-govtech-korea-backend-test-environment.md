---
title: "[BACKEND] 테스트 환경 구성 - Mockito 에이전트, 실패 로그, JaCoCo"
date: 2026-10-21 02:00:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [spring-boot, junit, mockito, jacoco, gradle, test, jdk25]
---

기능을 만들기 전에 테스트 환경부터 정비했다 ([PR #3](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/3)).
JUnit과 Mockito를 "추가"하려고 열어 봤더니 이미 들어 있었다. 할 일은 라이브러리 추가가 아니라 **실행 환경 설정**이었다.

## 이미 들어 있는 것

Spring Boot 4.1.1의 test starter(`spring-boot-starter-webmvc-test` 등)가 테스트 라이브러리를 함께 가져온다.

```
$ ./gradlew dependencies --configuration testRuntimeClasspath
```

| 라이브러리 | 버전 | 용도 |
|---|---|---|
| JUnit Jupiter | 6.0.3 | 테스트 실행 (Spring Boot 4부터 JUnit 6) |
| Mockito | 5.23.0 | mock 객체 |
| AssertJ | 3.27.7 | `assertThat(...)` 단언 |
| JsonPath, Hamcrest | - | MockMvc 응답 검증 |

## 변경 내용

커밋은 성격별로 넷으로 나눴다.

```
deps:  Mockito 에이전트용 의존성 구성 추가
build: 테스트 JVM에 Mockito 에이전트 등록
build: 테스트 실패 시 원인을 콘솔에 출력
build: JaCoCo 테스트 커버리지 리포트 추가
```

```groovy
configurations {
    mockitoAgent
}

dependencies {
    mockitoAgent('org.mockito:mockito-core') {
        transitive = false
    }
}

tasks.named('test') {
    useJUnitPlatform()
    jvmArgumentProviders.add({ ["-javaagent:${configurations.mockitoAgent.singleFile}"] } as CommandLineArgumentProvider)
    jvmArgs '-Xshare:off'
    testLogging {
        events 'failed', 'skipped'
        exceptionFormat = 'full'
    }
    finalizedBy tasks.named('jacocoTestReport')
}
```

## 새로 배운 것 1: Mockito는 왜 에이전트로 등록하나

Mockito는 `final` 클래스까지 mock으로 만들기 위해 **inline mock maker**를 쓴다. 이 방식은 실행 중인 JVM에 Java 에이전트를 붙여 클래스의 바이트코드를 바꾼다.

문제는 JDK 21부터 **실행 중 에이전트를 동적으로 붙이는 것**에 경고가 나고, 이후 JDK에서는 기본으로 막힐 예정이라는 점이다. 그래서 Mockito 문서는 테스트 JVM을 띄울 때 `-javaagent`로 미리 등록하라고 권한다.

Spring Boot Gradle 플러그인이 이걸 자동으로 해 주는지 확인하려고, init script로 test 태스크의 JVM 인자를 찍어 봤다.

```groovy
// print-jvmargs.gradle
gradle.projectsEvaluated {
    rootProject.tasks.named('test').configure { t ->
        t.doFirst { println "ALLJVMARGS=" + t.allJvmArgs.findAll { it.contains('agent') } }
    }
}
```

```
설정 전: ALLJVMARGS=[]
설정 후: ALLJVMARGS=[-javaagent:.../mockito-core-5.23.0.jar]
```

자동 등록은 없었다. 직접 설정한 뒤에는 에이전트가 붙는 것을 확인했다.

## 새로 배운 것 2: "Sharing is only supported for boot loader classes"

빌드할 때마다 이 경고가 찍혔다.

```
OpenJDK 64-Bit Server VM warning: Sharing is only supported for boot loader classes
because bootstrap classpath has been appended
```

**CDS(Class Data Sharing)**는 JDK 클래스를 미리 덤프해 두고 JVM 시작을 빠르게 하는 기능이다. 에이전트가 bootstrap classpath에 jar를 추가하면 CDS가 일부만 동작하고, JVM이 이 사실을 경고로 알린다. 동작에는 문제가 없지만 매번 로그에 찍히는 잡음이라, 테스트 JVM에서만 `-Xshare:off`로 CDS를 껐다.

## 새로 배운 것 3: Gradle 10에서 없어질 문법

처음에는 `exceptionFormat 'full'`처럼 공백으로 썼다. `--warning-mode all`로 돌려 보니 경고가 나왔다.

```
Properties should be assigned using the 'propName = value' syntax.
... This is scheduled to be removed in Gradle 10.
```

Groovy DSL에서 속성을 공백 문법으로 설정하는 방식은 Gradle 10에서 제거된다. `exceptionFormat = 'full'`로 고친 뒤 deprecation 경고는 0건이 됐다. 반면 `events 'failed', 'skipped'`나 `jvmArgs '-Xshare:off'`는 속성이 아니라 메서드 호출이라 그대로 써도 된다.

## 확인 방법

설정이 실제로 동작하는지 임시 테스트로 확인하고, 커밋 전에 지웠다.

- `@Mock List<String>`을 쓰는 테스트로 Mockito 동작 확인
- 일부러 실패하는 단언을 넣어 콘솔 출력 확인

```
TmpMockitoTest > mock() FAILED
    org.opentest4j.AssertionFailedError:
    expected: "b"
     but was: "a"
        at app//com.govtech.tmp.TmpMockitoTest.mock(TmpMockitoTest.java:19)
```

이제 테스트가 실패하면 리포트 파일을 열지 않아도 원인이 바로 보인다. CI 로그에서도 마찬가지다. 커버리지 리포트는 `build/reports/jacoco/test/html/index.html`에 생긴다.

## 다음 할 일

- CI에서 커버리지 리포트를 결과물(artifact)로 올리기
- 개발 착수 전 결정할 것들(기술 스택, 아키텍처, WBS)을 백로그로 정리
