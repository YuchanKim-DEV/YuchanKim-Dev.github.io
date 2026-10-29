---
title: "[VOICE_GATEWAY] 주소만으로는 모른다 - STT 서버 상태 확인과 HttpClient 타임아웃의 함정"
date: 2026-10-29 16:39:38 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [java-httpclient, timeout, health-check, spring-boot]
---

통합 콘솔의 STT 칸에는 `연결 주소: ws://localhost:8100/...` 한 줄만 있었다. 이 줄은 게이트웨이에 **설정된 값**일 뿐이다. STT 서버가 꺼져 있어도 똑같이 보인다. 그래서 게이트웨이가 STT 서버에 직접 물어보게 했다 ([PR #48](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/48)).

## /health 부르기

STT 담당자의 서버(streaming-asr1)는 `GET /health`에 모델과 장치를 알려 준다.

```json
{"status":"ok","model":"nemotron-3.5-asr-streaming-0.6b.q8_0.gguf","device":"gpu:0"}
```

게이트웨이는 지금 쓰는 WebSocket 주소를 HTTP로 바꿔(`ws→http`, `wss→https`) 같은 호스트의 `/health`를 부른다. 결과는 `GET /api/v1/stt/endpoint/health`로 내보낸다. 콘솔은 이 API를 5초마다 불러 "연결됨 · 57ms · gpu:0"이나 "응답 없음 · 연결 안 됨"을 보여 준다.

## 리뷰에서 걸린 함정: timeout은 헤더까지만

처음에는 이렇게 썼다.

```java
HttpRequest.newBuilder(uri).timeout(Duration.ofSeconds(2))...
client.send(request, BodyHandlers.ofString());
```

`HttpRequest.timeout`은 **응답 헤더를 받을 때까지만** 적용된다. 헤더를 곧바로 보내고 본문을 천천히 흘리는 서버를 만나면, `ofString()`은 본문이 끝날 때까지 크기 제한 없이 기다린다. 그동안 Tomcat 요청 스레드가 묶이고, 콘솔 탭마다 5초에 한 번씩 이런 요청이 쌓인다.

그래서 두 가지를 바꿨다.

- **전체 시간 제한:** `sendAsync(...).get(2초)`로 연결부터 본문까지 전부 2초 안에 끝나야 한다. 시간이 넘으면 `future.cancel(true)`로 요청 자체를 끊는다.
- **본문 크기 상한:** 직접 만든 `BodySubscriber`가 64KB까지만 모으고 그 뒤는 구독을 취소한다.

테스트에는 "헤더를 4초 늦게 보내는 서버"와 "헤더만 보내고 본문을 4초 끄는 서버"를 넣었다. 두 경우 모두 약 2.0초에 끊긴다.

## 덤: HTTP/2 업그레이드

JDK HttpClient의 기본 버전은 HTTP/2다. 그래서 `http://` 요청에도 `Upgrade: h2c` 헤더가 붙는다. uvicorn은 h2c를 받지 않아서 5초마다 STT 서버 로그에 경고가 쌓일 수 있다. `.version(HTTP_1_1)`으로 고정했다.

## 콘솔 쪽 경쟁 조건

주소를 바꾼 직후의 확인과 5초 주기 확인이 겹치면, 늦게 도착한 이전 주소의 결과가 새 결과를 덮을 수 있다. 요청마다 번호를 붙이고 마지막 번호의 응답만 화면에 반영한다.
