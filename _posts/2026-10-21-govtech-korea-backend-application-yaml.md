---
title: "[BACKEND] 설정 파일을 application.yaml로 - 바꾼 설정이 읽히는지 로그로 확인하기"
date: 2026-10-21 02:10:00 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [spring-boot, application-yaml, configuration, logging]
---

Spring Initializr가 만들어 준 `application.properties`를 `application.yaml`로 바꿨다 ([PR #38](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/38)).
앞으로 DB, 프로파일(local/dev/prod) 설정이 늘어날 텐데, 계층 구조가 한눈에 보이는 yaml이 읽기 편하다.

## 변경 내용

```properties
# 전: application.properties
spring.application.name=govtech
```

```yaml
# 후: application.yaml
spring:
  application:
    name: govtech
```

내용은 같다. Spring Boot는 `application.properties`, `application.yml`, `application.yaml`을 모두 읽는다. 확장자는 `.yaml`로 정했다.

## 새로 배운 것: 설정이 실제로 읽혔는지 확인하는 법

파일 형식만 바꾸면 빌드는 당연히 통과한다. 하지만 들여쓰기를 잘못 쓰거나 파일 이름에 오타가 있어도 빌드는 통과한다. 설정이 **실제로 읽혔는지** 확인할 방법이 필요했다.

지금 쓰는 Spring Boot 4.1.1은 `spring.application.name`을 설정하면 기본 로그 형식에서 **로그 줄마다 애플리케이션 이름이 붙는다.** 테스트 로그를 보면 바로 확인할 수 있다.

```
[govtech] [    Test worker] com.govtech.GovtechApplicationTests ...
```

`[govtech]`가 찍혔으니 yaml의 값이 로드된 것이다. 파일을 못 읽었다면 이 접두어가 없다.

## 함께 한 일: 착수 전 백로그 정리

공모전 개발 보고서(모민, MOMIN)를 읽고 개발 착수 전에 정해야 할 것과 세팅할 것을 GitHub 이슈로 정리했다.

| 마일스톤 | 이슈 수 | 내용 |
|---|---|---|
| M0. 개발 착수 준비 | 23 | WBS, 기술 스택, 아키텍처, 멀티테넌시, 인증, AI·통화 연동 규격, 개인정보 등 결정 11개 + 개발 환경 세팅 12개 |
| M1. MVP (~2026.12) | 11 | Tenant·정책 관리, 통화 세션 수집, 정책엔진·보호조치, 담당자 예외판단, MAX CAC 집계 등 |

결정이 필요한 이슈에는 `decision` 라벨을 붙였고, 이슈마다 선행 이슈를 적었다. 이번 yaml 전환은 그중 "환경별 설정 프로파일 분리" 이슈의 첫 단계다. 프로파일 분리 자체는 DB와 배포 환경이 정해진 뒤에 진행한다.

## 다음 할 일

- README를 "내려받아서 실행하는 방법" 중심으로 정리
- Dependabot, CI 커버리지 리포트, `.editorconfig`
