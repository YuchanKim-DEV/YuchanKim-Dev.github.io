---
title: "[BACKEND] PostgreSQL과 JPA로 결정 - 의존성은 순서대로 넣기"
date: 2026-10-21 14:14:44 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [postgresql, jpa, testcontainers, docker-compose, spring-boot]
---

미뤄 두었던 기술 스택 결정 중 두 가지를 정했다. **DB는 PostgreSQL, 데이터 접근은 Spring Data JPA**다. 그리고 첫 단계로 의존성을 추가했다 ([PR #53](https://github.com/GovTech-korea/GovTech-Korea_backend/pull/53), 이슈 #50).

## 결정 1: PostgreSQL

| | PostgreSQL | MySQL |
|---|---|---|
| AI 판정 결과 저장 | JSONB로 저장하고 내부 값으로 검색·인덱싱 | JSON 타입은 있지만 검색·인덱싱이 약한 편 |
| 통계·감사 조회 | 윈도우 함수, 복잡한 집계에 강함 | 충분하지만 덜 유연함 |

이 서비스는 AI가 통화 발화를 보고 내린 판정(위험유형, 등급, 확신도, 근거발화, 모델 버전)을 저장하고, 나중에 "왜 그렇게 판정했는지" 재현할 수 있어야 한다. 모델이 바뀌면 판정 결과의 구조도 바뀌기 쉬워서 JSONB가 잘 맞는다고 봤다.

## 결정 2: Spring Data JPA

서비스는 기관(Tenant)마다 데이터를 완전히 분리해야 한다. JPA(Hibernate)는 `@TenantId`로 모든 조회·저장에 tenant 조건을 자동으로 붙일 수 있다. 쿼리마다 `WHERE tenant_id = ?`를 직접 쓰다가 하나라도 빠뜨리면 다른 기관 데이터가 보이는 사고가 나는데, 그걸 프레임워크가 막아 준다. 복잡한 통계 쿼리는 필요한 곳에만 직접 SQL을 쓰기로 했다.

## 의존성은 왜 나눠서 넣나

처음엔 JPA, PostgreSQL, Testcontainers를 한 번에 넣으려 했다. 그런데 `spring-boot-starter-data-jpa`가 들어가는 순간 Spring Boot는 애플리케이션을 띄울 때 DB 연결을 요구한다. 지금 있는 테스트(`@SpringBootTest` 컨텍스트 로드)는 DB가 없으니 바로 실패한다.

커밋마다 빌드가 통과해야 한다는 규칙이 있어서 순서를 나눴다.

```
#50 deps    PostgreSQL 드라이버, Testcontainers, Docker Compose 지원   ← 이번 PR
#23 chore   compose.yaml (로컬 PostgreSQL)
#24 test    테스트용 PostgreSQL 컨테이너
#51 deps    Spring Data JPA        ← 여기서부터 DB 연결
#52 config  JPA 기본 설정
```

드라이버와 Testcontainers는 클래스패스에 있어도 아무 것도 자동으로 연결하지 않는다. 그래서 먼저 넣어도 빌드가 깨지지 않는다.

## 추가한 의존성

```groovy
developmentOnly 'org.springframework.boot:spring-boot-docker-compose'
runtimeOnly 'org.postgresql:postgresql'
testImplementation 'org.springframework.boot:spring-boot-testcontainers'
testImplementation 'org.testcontainers:testcontainers-junit-jupiter'
testImplementation 'org.testcontainers:testcontainers-postgresql'
```

좌표는 기억에 의존하지 않고 Spring Initializr에 같은 조합(Boot 4.1.1)으로 프로젝트를 만들어 보게 한 뒤 그대로 가져왔다. 실제로 Testcontainers 2.x는 모듈 이름이 `postgresql`에서 `testcontainers-postgresql`처럼 바뀌어 있었다.

| 의존성 | 하는 일 |
|---|---|
| `spring-boot-docker-compose` | `./gradlew bootRun` 때 `compose.yaml`의 컨테이너를 띄우고 접속 정보를 자동으로 연결. `developmentOnly`라 배포 jar에는 안 들어간다 |
| `spring-boot-testcontainers` | 테스트에서 띄운 컨테이너의 접속 정보를 `@ServiceConnection`으로 자동 연결 |

## 시행착오: 커밋 메시지 50자

첫 커밋 제목이 57자라 로컬 `commit-msg` hook에 막혔다.

```
commit-msg: 제목이 57자입니다. 50자 이내로 줄이세요.
```

마침 저장소 폴더를 `GOV_TECH/BACKEND`로 옮긴 직후라, git hook이 저장소와 함께 잘 따라왔는지도 덤으로 확인됐다.

## 다음 할 일

- 로컬 PostgreSQL용 `compose.yaml`
- 테스트용 PostgreSQL 컨테이너
