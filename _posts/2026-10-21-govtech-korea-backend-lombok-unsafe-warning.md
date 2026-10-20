---
title: "[BACKEND] JDK 25에서 Lombok의 sun.misc.Unsafe 경고 - 버전을 올려도 안 사라질 때"
date: 2026-10-21 02:19:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [jdk25, lombok, sun-misc-unsafe, gradle, javac, jep-498]
---

빌드할 때마다 컴파일 단계에서 경고 네 줄이 찍혔다. 동작에는 문제가 없지만, 매번 보이는 경고는 진짜 문제가 생겼을 때 눈에 안 띄게 만든다. 원인을 찾아 정리했다 ([PR #40](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/40), 이슈 #18).

```
WARNING: A terminally deprecated method in sun.misc.Unsafe has been called
WARNING: sun.misc.Unsafe::objectFieldOffset has been called by lombok.permit.Permit
WARNING: Please consider reporting this to the maintainers of class lombok.permit.Permit
WARNING: sun.misc.Unsafe::objectFieldOffset will be removed in a future release
```

## 원인

`sun.misc.Unsafe`는 JDK 내부의 저수준 메모리 접근 API다. 공식 API가 아닌데도 많은 라이브러리가 성능이나 내부 해킹을 위해 써 왔다. JDK는 이를 단계적으로 없애는 중이다.

| JEP | JDK | 내용 |
|---|---|---|
| JEP 471 | 23 | 메모리 접근 메서드를 제거 예정(deprecated for removal)으로 지정 |
| JEP 498 | 24 | 이 메서드를 처음 호출할 때 런타임 경고 출력 (기본값 `warn`) |

Lombok은 컴파일러(javac) 안에서 동작하는 어노테이션 프로세서다. 내부 클래스 `lombok.permit.Permit`이 `Unsafe::objectFieldOffset`을 쓰기 때문에, JDK 25로 컴파일하면 javac 프로세스에서 이 경고가 나온다.

## 시도 1: Lombok 버전 올리기 (실패)

Spring Boot 4.1.1이 관리하는 Lombok은 1.18.46이고, Maven Central의 최신은 1.18.48이었다.

```groovy
ext['lombok.version'] = '1.18.48'   // 임시로 버전 지정
```

```
+--- org.projectlombok:lombok -> 1.18.48
WARNING: sun.misc.Unsafe::objectFieldOffset has been called by lombok.permit.Permit
```

최신 버전도 같은 경고를 냈다. Lombok 쪽에서 아직 해결하지 않은 문제라 버전 상향으로는 안 된다. 이 변경은 되돌렸다.

## 시도 2: 컴파일러 프로세스에만 허용 옵션 주기 (성공)

JDK 24부터 이 동작을 정하는 옵션이 있다.

```
--sun-misc-unsafe-memory-access=allow | warn | debug | deny
```

애플리케이션 전체가 아니라 **컴파일러 프로세스에만** `allow`를 준다.

```groovy
tasks.withType(JavaCompile).configureEach {
    options.fork = true
    options.forkOptions.jvmArgs.add('--sun-misc-unsafe-memory-access=allow')
}
```

`forkOptions.jvmArgs`는 컴파일러를 별도 프로세스로 띄울 때만 적용된다. 그래서 `options.fork = true`를 명시했다. 이 프로젝트는 Gradle은 JDK 21로 돌고 컴파일은 toolchain의 JDK 25로 하기 때문에 원래도 별도 프로세스로 돌지만, 설정 의도를 코드에 드러내려고 적었다.

```
$ ./gradlew clean build --warning-mode all
BUILD SUCCESSFUL in 13s
```

경고가 하나도 없는 빌드 출력이 됐다.

## 새로 배운 것: 경고를 끄는 범위

경고를 없애는 방법은 여러 가지인데, **어디까지 끄느냐**가 중요하다.

- 애플리케이션 실행(`bootRun`, 운영 서버)에는 손대지 않았다. 나중에 우리 코드나 다른 라이브러리가 `Unsafe`를 쓰면 그때는 경고가 보여야 한다.
- 컴파일러 프로세스만 대상으로 했다. 원인이 Lombok(컴파일 시점 도구)이니까 범위도 거기까지다.
- 나중에 JDK가 이 메서드를 실제로 제거하면 Lombok이 대응 버전을 내야 한다. 그때 이 설정도 지울 수 있는지 다시 본다.

## 다음 할 일

- Dependabot, CI 커버리지 리포트, `.editorconfig`
