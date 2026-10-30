---
title: "[VOICE_GATEWAY] 100vh와 100dvh - 모바일 앱 화면 아래 여백 없애기"
date: 2026-10-30 12:46:03 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [css, mobile, viewport, ux-writing]
---

팀원들이 휴대폰으로 시험해 보고 두 가지를 알려 줬다.

## 1. 카드 제목은 역할 이름으로

시작 화면 카드 제목이 "전화를 받는 사람", "전화를 거는 사람"이었다. 서비스 안에서는 모두 **상담원**, **민원인**이라고 부르는데 첫 화면만 다른 말을 쓰고 있었다. 제목을 역할 이름으로 바꾸고, 바로 위에 있던 "상담원 앱"·"민원인 앱" 작은 글씨는 제목과 겹쳐서 뺐다 ([PR #52](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/52)).

## 2. 화면 끝인데 더 내려간다

휴대폰에서 앱 화면 아래에 빈 여백이 생기고, 끝까지 내려도 페이지가 조금 더 스크롤됐다 ([PR #54](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/54)).

원인은 높이 단위가 섞인 것이었다.

| 요소 | 높이 | 뜻 |
|---|---|---|
| 앱 틀 `.device` | `100dvh` | 지금 보이는 높이 (주소창 제외) |
| 바깥 `body`, `.stage` | `min-height: 100vh` | 주소창이 **숨었을 때**의 큰 높이 |

모바일 브라우저의 `100vh`는 주소창이 접힌 상태를 기준으로 한다. 그래서 주소창이 보이는 동안에는 바깥 페이지가 앱보다 주소창 높이만큼 길어지고, 그만큼이 아래 여백과 남는 스크롤이 됐다.

좁은 화면에서는 바깥 페이지를 보이는 높이에 고정하고, 스크롤은 앱 안에서만 일어나게 했다.

```css
@media (max-width: 820px) {
  html:has(.stage), body:has(.stage) { height: 100%; overflow: hidden; overscroll-behavior: none; }
  .stage { height: 100vh; height: 100dvh; }   /* dvh를 모르는 브라우저는 vh */
  .screen { overscroll-behavior: contain; }   /* 안쪽 스크롤이 끝나도 바깥으로 넘기지 않음 */
}
```

`:has(.stage)`로 앱 화면이 있는 페이지에만 적용했다. 시작 화면과 통합 콘솔은 원래 긴 페이지라 평소처럼 스크롤된다.
