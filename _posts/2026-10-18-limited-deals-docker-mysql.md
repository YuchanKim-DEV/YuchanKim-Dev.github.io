---
title: "[LIMITED_DEALS] Docker로 MySQL 8.4 띄우기 - 이미지, 컨테이너, compose, 그리고 막혔던 것들"
date: 2026-10-18 22:10:00 +0900
categories: [SIDE PROJECTS, LIMITED_DEALS]
tags: [docker, docker-compose, mysql, docker-hub, path, volume]
---

로드맵 0단계의 첫 걸음으로 로컬 MySQL을 Docker로 띄웠다. 명령 몇 줄이면 끝날 줄 알았는데, 개념부터 환경 문제까지 꽤 많이 부딪혔다. 결과물은 [PR #9](https://github.com/YuchanKim-DEV/SIDEPROJECT_LIMITED_DEALS/pull/9)의 `docker-compose.yml`이다.

## 환경

| 항목 | 값 |
|---|---|
| 맥 | M1 (arm64) |
| Docker | 29.8.2, Compose v5.6.0 (Docker Desktop) |
| MySQL | 8.4.11 (`mysql:8.4.11` 이미지) |

## 새로 배운 것 1: Docker Hub, 이미지, 컨테이너

```
Docker Hub ──pull(다운로드)──▶ 내 컴퓨터의 이미지 ──run(실행)──▶ 컨테이너
 (저장소)                       (설계도)                         (실제로 돌아가는 것)
```

- **Docker Hub**: 이미지를 모아둔 공개 저장소. GitHub가 코드 저장소라면 Docker Hub는 이미지 저장소다. `mysql`은 공식 이미지(`Docker Official Image`)고, 정식 이름은 `docker.io/library/mysql`이다.
- **이미지**는 설계도, **컨테이너**는 그걸 실행한 것이다. 이미지 하나로 컨테이너를 여러 개 만들 수 있다.
- `run`은 이미지가 없으면 **알아서 pull까지** 한다. 처음 `hello-world`를 실행했을 때 `Unable to find image ... locally` 다음에 `Pulling from library/hello-world`가 나온 게 그것이다.

`hello-world`는 설치가 제대로 됐는지 한 번 확인하는 테스트 이미지다. 출력이 나왔다는 것 자체가 "명령(client) → 엔진(daemon) → Docker Hub에서 pull → 컨테이너 실행 → 출력 전달"이 전부 정상이라는 뜻이다. 컨테이너가 떠 있는 동안 앱이 살아 있는지 주기적으로 검사하는 **healthcheck**와는 다른 개념이다.

## 새로 배운 것 2: 띄우는 방법은 세 가지

| 방법 | 이미지 받기 | 실행 | 설정이 남는가 |
|---|---|---|---|
| Docker Desktop 화면 | Pull 버튼 | Run 버튼, 입력칸 | 컨테이너를 지우면 사라짐 |
| `docker run` | 필요하면 자동 | 옵션을 한 줄에 길게 | 터미널 기록에만 |
| **`docker compose`** | `up`할 때 자동 | 파일에 적고 `up -d` | **파일로 남음** |

Desktop에서 Stop만 하면 설정은 남는다. 지우면 다시 입력해야 하고, 무엇보다 다른 컴퓨터에서 재현할 수 없다. 이 프로젝트는 나중에 MySQL, Redis, Kafka를 같이 띄울 거라 **compose**로 정했다. Desktop 화면은 compose로 띄운 컨테이너의 상태와 로그를 보는 용도로 같이 쓴다.

### 배포까지 생각하면
- **RDS**를 쓰면 DB는 AWS가 운영하니 compose도 Docker도 필요 없다. 앱 설정의 접속 주소만 바꾸면 된다.
- **EC2**는 빌린 서버 한 대라서 compose를 써도 되고 안 써도 된다. 규모가 커지면 ECS나 쿠버네티스로 넘어간다.
- 서버에는 화면이 없다. 그래서 조작은 **터미널 명령과 compose 파일**로 익혀 두기로 했다. 로컬에서 치는 명령이 서버에서도 그대로 통한다.

## 새로 배운 것 3: `latest`를 쓰면 안 되는 이유

Desktop에서 mysql을 검색하니 태그 기본값이 `latest`였는데, 이게 **MySQL 26.7**(혁신 릴리스)이었다.

| 태그 | 의미 |
|---|---|
| `latest`, `26`, `innovation` | 26.7 (자주 바뀜) |
| `lts`, `9` | 9.7 |
| `8`, `8.4`, **`8.4.11`** | 8.4 LTS |

`8.4.11-oracle`, `8.4.11-oraclelinux9`처럼 접미사가 붙은 태그도 있었는데, 문서에서 한 줄에 쉼표로 묶여 있는 태그들은 **같은 이미지의 별칭**이다. 접미사는 이미지 안의 운영체제(Oracle Linux 9) 이름일 뿐이다. `8`이나 `8.4`는 새 패치가 나오면 다른 버전을 받게 되니, 결과를 비교할 실험에서는 **`8.4.11`로 고정**했다. 처음에 "MySQL 8"이라고만 정했는데, 실제로는 8.0이 이미 지원 종료라 8.4를 뜻한다는 것도 이때 알았다.

## compose 파일

```yaml
services:
  mysql:
    image: mysql:8.4.11
    container_name: limited-deals-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: limited_deals
      MYSQL_USER: admin
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

- `${...}`는 같은 폴더의 `.env`에서 읽는다. 비밀번호는 `.env`에만 두고 `.gitignore`에 넣었다.
- `volumes`가 없으면 컨테이너를 지울 때 **데이터도 같이 사라진다.** 로드맵의 "데이터 유실 방지"의 가장 기초 사례다.
- `restart: always`면 Docker가 켜질 때마다 이 컨테이너도 같이 켜진다. 직접 멈춘 뒤에는 안 켜지게 하려면 `unless-stopped`가 낫다.
- 문자셋 옵션은 넣지 않았다. MySQL 8.4는 기본값이 이미 `utf8mb4`다.

## 새로 배운 것 4: 초기화 로그 읽기

처음 `up` 하면 로그에 `ready for connections`가 두 번 나오고 그 사이에 `SHUTDOWN`이 있다. 오류가 아니라 초기화 과정이다.

1. `port: 0`으로 임시 서버를 띄운다(외부 접속 차단).
2. `.env`의 비밀번호로 root를 설정하고, `Creating database limited_deals`, `Creating user admin`.
3. 임시 서버를 종료한다.
4. `port: 3306`으로 진짜 서버를 띄운다.

여기서 중요한 함정: **`MYSQL_*` 환경변수는 볼륨이 비어 있을 때 처음 한 번만 적용된다.** 나중에 비밀번호나 사용자를 바꿔도 반영되지 않는다. 바꾸려면 `docker compose down -v`로 볼륨까지 지우고 다시 만들어야 한다. 그래서 `.env`를 먼저 만들고 `up` 하는 순서가 중요하다.

`down` 후 다시 `up` 하면 이번엔 초기화 없이 바로 뜬다. 볼륨에 데이터가 남아 있기 때문이다.

## 막혔던 것들

1. **`docker: command not found`.** Docker Desktop을 다시 설치했는데 명령을 못 찾았다. 새 설치는 `docker` 파일을 `~/.docker/bin`에 두는데, 이 폴더가 **PATH**에 없었다. 터미널은 PATH에 적힌 폴더들만 뒤져서 명령을 찾는다. `~/.docker/bin/docker --version`으로 파일이 있는 걸 확인하고, `~/.zshrc`에 `export PATH="$HOME/.docker/bin:$PATH"`를 추가해서 해결했다. 같은 원리로 지금 `java`가 21로 나오는 것도 PATH 맨 앞에 `openjdk@21`이 있어서다.
2. **`failed to connect to the docker API at unix:///.../docker.sock`.** Docker Desktop이 꺼져 있었다. 명령은 `docker.sock`이라는 통로로 엔진에 요청하는데, 엔진이 꺼지면 그 파일이 없다. `open -a Docker`로 켜고 해결했다. 서버에서는 Docker가 시스템 서비스로 부팅 때 켜지고, 맥에서는 그 역할을 Desktop 앱이 한다.
3. **compose 파일을 엉뚱한 폴더에 만들었다.** 프로젝트 루트가 아니라 `.claude/` 안에 만들어서 compose가 찾지 못했다. `docker compose`는 명령을 실행한 폴더에서 파일을 찾고, `.env`도 compose 파일과 같은 폴더에서 읽는다.
4. **`.env` 없이 `docker compose config`.** `The "MYSQL_ROOT_PASSWORD" variable is not set` 경고가 나왔다. 이 상태로 띄우면 root 비밀번호가 없어서 MySQL이 시작하자마자 종료된다.
5. **`ERROR 1044 Access denied ... to database`.** 접속할 때 없는 DB 이름을 적었다. 만든 DB는 `limited_deals` 하나다.

## 확인

```sql
SELECT VERSION(), CURRENT_USER(), DATABASE();
-- 8.4.11 | admin@% | limited_deals
```

DB 클라이언트(DBeaver)로도 연결해서 볼 수 있다. 다만 **테이블은 클라이언트에서 손으로 만들지 않는다.** 화면에서 만든 테이블은 기록이 남지 않아 재현할 수 없다. 테이블은 코드로 만든다. 다음은 JPA의 `ddl-auto`로 먼저 만들어 보고, Flyway로 옮겨서 차이를 겪어 볼 계획이다.
