---
title: "[VOICE_GATEWAY] 로그와 음원을 한 폴더에 - runtime/과 터미널을 닫아도 켜 두는 실행"
date: 2026-10-29 11:47:39 +0900
categories: [SIDE PROJECTS, GOVTECH-KOREA]
tags: [devops, git, macos, cloudflare-tunnel]
---

팀원들이 원격으로 들어와 시험하기 시작하면서 실행 중에 생기는 파일이 흩어져 있다는 게 불편해졌다. 서버·터널 로그는 `~/Library/Logs`에, STT 음원 기록은 `build/stt-dump`에 있었다. 특히 `build/` 안의 음원은 `./gradlew clean` 한 번이면 다 지워진다. 그래서 프로젝트 안 **`runtime/` 한 곳**으로 모았다 ([PR #32](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/32), [#33](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/33), [#34](https://github.com/GovTech-korea/GovTech-Korea_VoiceGateway/pull/34)).

```
runtime/            ← .gitignore
├── logs/           server.log, tunnel.log, *.pid
└── stt-dump/       <callId>-<시작 ms>.pcm
```

## 성격별로 PR을 나눴다

바뀐 건 세 줄짜리지만, 이 저장소는 브랜치 하나에 목적 하나, 커밋 하나에 성격 하나가 규칙이다.

| PR | 타입 | 내용 |
|---|---|---|
| #32 | chore | `.gitignore`에 `runtime/` |
| #33 | config | `application.yaml`의 기록 경로 예시 |
| #34 | docs | README, STT 연동 규격의 경로와 실행 방법 |

세 PR을 거의 동시에 올렸더니, 첫 PR이 머지된 뒤 나머지가 **BEHIND** 상태로 멈췄다. develop 보호 규칙이 "최신 develop 기준으로 통과한 PR만 머지"이기 때문이다. `gh pr update-branch`로 develop을 반영하고 CI를 다시 돌려 하나씩 머지했다.

## 새로 배운 것: 열려 있는 로그 파일을 옮겨도 계속 써진다

터널(cloudflared)을 다시 켜면 공개 주소가 바뀐다. 팀원들이 이미 쓰고 있는 주소라 터널은 건드리고 싶지 않았다. 그런데 그 터널이 지금 쓰고 있는 로그 파일을 새 폴더로 옮겨야 했다.

macOS(유닉스)에서 프로세스는 파일을 **이름이 아니라 열어 둔 파일 자체**(inode)로 붙잡고 있다. 같은 디스크 안에서 `mv`는 이름표만 바꾸고 파일 자체는 그대로 둔다. 그래서 터널은 아무것도 모른 채 새 위치의 같은 파일에 계속 로그를 쓴다. 터널을 끄지 않고 로그만 옮길 수 있었다. 다른 디스크로 옮기면 복사 후 삭제가 되어 이 방법은 통하지 않는다.

## 터미널을 닫아도 켜 두기

Claude Code 세션의 백그라운드 작업으로 띄운 서버와 터널은 세션이 끝나면 같이 꺼졌다. 그래서 세션과 분리해서 띄우는 방법을 README에 적었다.

```bash
nohup ./gradlew bootRun --args='--voicegateway.stt.dump-dir=runtime/stt-dump' > runtime/logs/server.log 2>&1 &
echo $! > runtime/logs/server.pid
nohup cloudflared tunnel --no-autoupdate --url http://localhost:8080 > runtime/logs/tunnel.log 2>&1 &
echo $! > runtime/logs/tunnel.pid
nohup caffeinate -dimsu -w $(cat runtime/logs/tunnel.pid) > /dev/null 2>&1 &
```

- `nohup`: 터미널이 닫혀도 프로세스를 끝내지 않는다
- `*.pid`: 나중에 `kill $(cat ...)`으로 정확히 그 프로세스만 끈다
- `caffeinate -w <pid>`: 그 프로세스(터널)가 살아 있는 동안만 Mac이 잠들지 않게 한다. 터널을 끄면 자동으로 풀린다
