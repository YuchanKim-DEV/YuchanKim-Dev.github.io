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

## 3. 민원인 앱도 같은 색으로, 스크롤 없이 한 화면에

[PR #56](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/56)

상담원 앱은 남색 바탕에 빨강 포인트인데 민원인 앱만 흰 바탕이었다. 같은 서비스의 두 앱인데 따로 만든 것처럼 보여서, 민원인 앱의 색 변수를 상담원 앱과 같은 값으로 맞췄다.

스크롤 문제는 직접 재 보니 원인이 분명했다. 민원인 화면 내용은 647px인데 휴대폰에서 쓸 수 있는 높이는 552~616px이었다. 게다가 대기 상태에서도 **보이지 않는 칸이 자리를 차지**하고 있었다.

- 통화 중에만 움직이는 소리 막대는 `visibility: hidden`이라 안 보여도 44px를 그대로 차지했다.
- 비어 있는 통화 시간·안내 문구도 `min-height: 22px`만큼 자리를 잡고 있었다.

그래서 아래 빈 공간까지 스크롤이 내려갔다. 소리 막대는 `visibility: hidden` 대신 `display: none`으로 숨기고, 빈 문구는 `:empty`일 때 숨겼다. 해태와 소리 그래프는 화면 높이에 따라 줄어들게 했다.

```css
.app-citizen .call-sub:empty, .app-citizen .call-timer:empty { display: none; }
.app-citizen .avatar { width: clamp(96px, 20dvh, 150px); height: clamp(96px, 20dvh, 150px); }
```

이제 360×600과 390×664 크기에서 대기·통화 중·파일 선택 화면 모두 스크롤이 없다.
