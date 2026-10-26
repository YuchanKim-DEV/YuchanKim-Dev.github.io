---
title: "[VOICE_GATEWAY] 상담 연결 구현 - 큐, 자동 배정, 벨을 WebSocket으로"
date: 2026-10-26 22:51:09 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [spring-boot, websocket, java, state-machine, jackson3, testing]
---

[연동 규격 v2]({% post_url 2026-10-26-govtech-korea-voice-gateway-queue-ringing-spec %})대로 상담 연결을 구현했다 ([PR #15](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/15)). 민원인 큐, 상담원 자동 배정, 벨과 미응답 처리, 통화 중 음성 중계까지다.

## 구조

```
CallWebSocketHandler   /ws/v1/call — 텍스트는 해석, 바이너리는 음성
   │
   ├─ ClientMessageParser  JSON → 메시지 객체 (규격 위반은 오류 코드)
   │
   └─ CallCoordinator      큐·배정·벨·통화 상태 전부
         └─ ClientConnection (인터페이스)   ← WebSocket과 분리
```

핵심은 `CallCoordinator`가 **WebSocket을 모르게** 만든 것이다. 앱 연결을 `ClientConnection`이라는 인터페이스로만 다루니까, 테스트에서는 보낸 메시지를 기록만 하는 가짜 연결을 넣어 상담 흐름 전체를 검증할 수 있다. 벨 타이머도 `RingTimer` 인터페이스로 빼서, 테스트에서 20초를 기다리지 않고 "타이머 지금 실행"으로 미응답 상황을 만든다.

```java
@Test
@DisplayName("20초 안에 안 받으면 벨이 멈추고 상담원은 AWAY, 민원인은 다음 상담원에게 배정된다")
void timeout() {
    FakeConnection first = agent("agent-01");
    FakeConnection second = agent("agent-02");
    FakeConnection citizen = citizen("call-1");

    ringTimer.fireAll();

    assertThat(first.messages).containsSubsequence(
            new ServerMessage.RingCancelled("call-1", RingCancelReason.TIMEOUT),
            new ServerMessage.StatusChanged(AgentState.AWAY, StatusReason.NO_ANSWER));
    assertThat(second.last()).isEqualTo(new ServerMessage.Ringing("call-1", RING_TIMEOUT_MS));
}
```

## "가장 오래 대기한 상담원"을 시각 대신 번호로

처음엔 READY가 된 시각을 저장하려 했다. 그런데 테스트에서 두 상담원이 같은 밀리초에 등록되면 순서가 흔들린다. 그래서 상담원이 READY가 될 때마다 1씩 늘어나는 번호(`readyOrder`)를 붙이고, 가장 작은 번호부터 배정한다. 통화를 마친 상담원은 새 번호를 받아 맨 뒤로 간다. 통화가 한 사람에게 몰리지 않는다.

## 잠금 하나, 그리고 음성만 예외

상태 변경(등록, 배정, 벨, 종료)은 모두 `CallCoordinator`의 잠금 하나 안에서 일어난다. 상담원 수가 적은 MVP에서는 이게 가장 버그가 적다. 다만 음성 프레임은 통화마다 초당 50개씩 양방향으로 오가서, 매번 잠금을 잡으면 상태 처리와 경쟁하게 된다. 그래서 통화가 연결되면 "연결 ID → 상대 연결"을 동시성 맵에 넣어 두고, 음성은 잠금 없이 이 맵만 보고 전달한다.

## 새로 배운 것 1: Spring Boot 4는 Jackson 3

JSON을 다루다가 패키지 이름이 `com.fasterxml.jackson.databind`가 아니라 `tools.jackson.databind`인 것을 보고 알았다. Spring Boot 4는 Jackson 3을 쓴다.

| | Jackson 2 | Jackson 3 |
|---|---|---|
| 패키지 | `com.fasterxml.jackson.databind` | `tools.jackson.databind` |
| 문자열 노드 확인 | `isTextual()` / `asText()` | `isString()` / `asString()` |
| 예외 | `JsonProcessingException` (checked) | `JacksonException` (unchecked) |
| 어노테이션 | `com.fasterxml.jackson.annotation` | **그대로** |

어노테이션 패키지는 그대로라, 메시지 record의 `type` 필드는 기존처럼 `@JsonProperty`로 붙였다.

## 새로 배운 것 2: 테스트에서 터진 WebSocket 설정

WebSocket 메시지 버퍼를 키우려고 `ServletServerContainerFactoryBean`을 등록했더니, 앱을 띄우는 기본 테스트가 이렇게 실패했다.

```
Attribute 'jakarta.websocket.server.ServerContainer' not found in ServletContext
```

`@SpringBootTest`는 기본으로 실제 서버 없이 가짜 웹 환경에서 뜨는데, 이 빈은 실제 서블릿 컨테이너를 요구한다. 다시 계산해 보니 버퍼 기본값이 8KB이고 우리 음성 프레임은 640바이트라 키울 이유가 없었다. 설정을 지웠다. 대신 실제 서버를 띄우는 통합 테스트(`RANDOM_PORT`)를 따로 두고, 진짜 WebSocket 클라이언트로 등록 → 전화 → 벨 → 받기 → 음성 왕복 → 끊기까지 확인한다.

## 리뷰에서 나온 것

PR 전에 리뷰를 돌렸더니 전송 계층 문제가 나왔다. 상담원 네트워크가 2초 넘게 멈추면 전송 한도 예외가 처리되지 않아, 엉뚱하게 민원인 연결이 끊긴다. 또 잠금 안에서 네트워크로 보내고 있어 느린 접속자 하나가 게이트웨이 전체를 멈출 수 있다. 이건 바로 다음 PR에서 고친다.

## 테스트

| 테스트 | 수 | 내용 |
|---|---|---|
| `CallCoordinatorTest` | 24 | 등록·중복, 큐 순번, 최장 대기 우선, 받기, 미응답·거절, 끊김 |
| `ClientMessageParserTest` | 13 | 메시지 해석, 오류 코드 |
| `CallWebSocketIntegrationTest` | 2 | 실제 서버 접속 전체 흐름, 규격 위반 1008 종료 |

## 다음 할 일

- 느린 연결 전송 문제 수정
- 민원인 → 상담원 음성 지연 버퍼와 차단
