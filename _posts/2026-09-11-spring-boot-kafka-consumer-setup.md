---
title: "Spring Boot에 Kafka 컨슈머 붙이기 - 의존성과 자동 설정, 그리고 프로퍼티"
date: 2026-09-11 21:00:00 +0900
categories: [SpringBoot, Kafka]
tags: [kafka, spring-kafka, spring-boot, java, yaml, auto-configuration]
---

Kafka 컨슈머를 Spring Boot에 붙이면서 정리한 내용을 남긴다.
첫 글은 **의존성과 기본 설정**이다. 설정값 하나하나에 "왜 이 값인가"를 붙여둔다.

## 환경

| 항목 | 버전 |
|------|------|
| Java | 17 |
| Spring Boot | 3.3.4 |
| spring-kafka | 3.2.4 |
| kafka-clients | 3.7.1 |
| Gradle | 8.10.1 |

## 의존성

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.4'
    id 'io.spring.dependency-management' version '1.1.6'
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'

    // kafka
    implementation 'org.springframework.kafka:spring-kafka'
    testImplementation 'org.springframework.kafka:spring-kafka-test'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

### `spring-kafka`에 버전을 적지 않은 이유

```groovy
implementation 'org.springframework.kafka:spring-kafka'   // 버전 없음
```

`io.spring.dependency-management` 플러그인이 Spring Boot BOM을 적용해주기 때문에
`spring-kafka`와 그 안에 들어 있는 `kafka-clients`까지 **부트 버전에 맞는 조합으로 고정**된다.

여기에 버전을 직접 박으면 나중에 부트를 올릴 때 조합이 어긋난다.
Boot 3.3.4가 지정한 조합은 `spring-kafka 3.2.4` → `kafka-clients 3.7.1`이었다.

실제로 무엇이 잡혔는지는 이렇게 확인한다.

```bash
./gradlew dependencies --configuration runtimeClasspath | grep kafka
```

`kafka-clients`는 브로커와의 호환 범위가 넓어서 버전이 정확히 일치하지 않아도 대체로 동작하지만,
브로커 버전은 따로 확인해두는 게 좋다. 특정 기능은 브로커 최소 버전을 요구한다.

### `spring-kafka-starter`는 없다

`spring-boot-starter-web`처럼 `spring-boot-starter-kafka`가 있을 것 같지만 **없다.**
`org.springframework.kafka:spring-kafka`를 직접 넣는 게 맞다.
그래도 자동 설정은 `spring-boot-autoconfigure` 안에 들어 있어서 정상적으로 동작한다.

### `spring-kafka-test`

`EmbeddedKafka`를 제공한다. 브로커를 띄우지 않고 테스트에서 인메모리 브로커를 쓸 수 있다.
이 글의 범위를 넘어서므로 여기서는 의존성만 걸어둔다.

## 설정 클래스를 하나도 만들지 않았다

`ConsumerFactory`나 `ConcurrentKafkaListenerContainerFactory`를 직접 빈으로 등록한 곳이 **없다.**

```bash
grep -rl "ConsumerFactory\|KafkaListenerContainerFactory" src/main/java/
# (결과 없음)
```

`KafkaAutoConfiguration`이 `spring.kafka.*` 프로퍼티를 읽어
`ConsumerFactory` → `ConcurrentKafkaListenerContainerFactory`까지 다 만들어준다.
`@EnableKafka`도 부트가 자동으로 처리하므로 붙이지 않아도 된다.

직접 만들어야 하는 건 이런 경우다.

- 리스너마다 **다른 컨테이너 팩토리**를 쓰고 싶을 때
- `ErrorHandler`, `RecordInterceptor`, 커스텀 `RetryTemplate`을 꽂을 때
- 역직렬화를 타입별로 다르게 가져갈 때

그런 요구가 없으면 **프로퍼티만으로 끝내는 쪽이 낫다.**
설정 클래스를 만들어두면 프로퍼티가 무시되는 구간이 생겨서, 나중에 yaml만 보고는 동작을 알 수 없게 된다.

## 기본 설정

```yaml
spring:
  kafka:
    bootstrap-servers: <broker-host>:9092
    consumer:
      group-id: my-consumer-group
      enable-auto-commit: false
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    listener:
      concurrency: 1
      ack-mode: MANUAL_IMMEDIATE
```

### `bootstrap-servers`

브로커 **전체 목록이 아니라 진입점**이다.
여기 적힌 브로커에 붙어서 클러스터 메타데이터(전체 브로커, 파티션 리더 위치)를 받아온 뒤,
실제 통신은 각 파티션의 리더 브로커와 직접 한다.

그래서 한 대만 적어도 동작하지만, **그 한 대가 죽으면 기동이 안 된다.**
운영이라면 쉼표로 두세 대를 적는다.

```yaml
bootstrap-servers: host-1:9092,host-2:9092,host-3:9092
```

### `group-id`

오프셋이 저장되는 단위이자, 파티션이 분배되는 단위다.
**이 값을 바꾸면 완전히 새로운 컨슈머가 된다.** 저장된 오프셋이 없으니
`auto-offset-reset`에 따라 처음부터 다시 읽기 시작한다.

오타 하나로 전량 재처리가 시작될 수 있는데 에러는 나지 않는다.
그래서 그룹명은 yaml에 고정해두고 함부로 건드리지 않는다.

### 역직렬화를 `String`으로 둔 이유

```yaml
key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

JSON 메시지를 받지만 `JsonDeserializer`를 쓰지 않았다. **원문 문자열로 받아서 직접 파싱한다.**

`JsonDeserializer`를 쓰면 역직렬화 실패가 **리스너에 들어오기 전에** 터진다.
그러면 그 메시지의 원문을 로그로 남기기 어렵고, 에러 핸들러를 따로 붙여야 한다.
문자열로 받으면 파싱 실패를 **내 코드 안에서** 잡을 수 있어서, 원문을 남기고 분기하기 쉽다.

대신 대가가 있다. 타입 안전성이 없고 파싱 코드를 직접 써야 한다.
스키마가 안정적이고 관리 주체가 같다면 `JsonDeserializer` 쪽이 편하다.
**메시지를 만드는 쪽이 외부라면** 문자열로 받는 편이 방어적이다.

### `enable-auto-commit: false`

기본값은 `true`이고, 백그라운드에서 주기적으로 오프셋을 커밋한다.
문제는 커밋 기준이 "**가져온** 메시지"라는 점이다.
아직 처리하지 않은 메시지의 오프셋이 커밋될 수 있고, 그 상태로 죽으면 그 메시지는 다시 오지 않는다. **유실**이다.

그래서 자동 커밋을 끄고 직접 커밋한다. 커밋 전략은 따로 정리한다.

### `ack-mode: MANUAL_IMMEDIATE`

`enable-auto-commit: false`와 짝이다. 수동 ack을 쓰겠다는 선언.

- `MANUAL` : `acknowledge()` 호출을 모아뒀다가 **다음 poll 주기**에 커밋
- `MANUAL_IMMEDIATE` : 호출 **즉시** 커밋

즉시 커밋하면 커밋 요청이 잦아지는 대신, 장애 시 다시 받는 메시지 수가 줄어든다.
재처리 구간을 줄이는 쪽을 택했다.

### `auto-offset-reset: earliest`

**저장된 오프셋이 없을 때만** 쓰인다. 평소 운영 중에는 아무 영향이 없다.

| 값 | 동작 |
|----|------|
| `latest` (기본) | 지금 이후 들어오는 것부터. 쌓여 있던 건 건너뛴다 |
| `earliest` | 브로커에 남아 있는 가장 오래된 것부터 |

새 그룹을 만들거나 오프셋이 만료됐을 때 쌓인 메시지를 흘려보내지 않으려고 `earliest`로 뒀다.
바꿔 말하면 **그룹명을 잘못 건드리면 전량 재처리**라는 뜻이라, 위의 `group-id` 주의와 한 쌍이다.

### `concurrency: 1`

리스너 컨테이너가 띄우는 컨슈머 스레드 수다. 기본값은 1.

파티션이 여러 개면 이 값을 올려 병렬로 당길 수 있지만, 여기서는 1로 뒀다.
**리스너는 가져오기만 하고 실제 처리는 내부 워커 풀에서** 하기 때문이다.
이 구조를 택한 이유는 순서 보장과 얽혀 있어서 따로 다룬다.

### 브로커 보안 설정을 넣는 자리

브로커가 SSL이나 SASL을 요구하면 `properties` 아래에 원본 Kafka 설정 키를 그대로 넣는다.

```yaml
spring:
  kafka:
    consumer:
      properties:
        security.protocol: SSL
        # SASL이면
        # security.protocol: SASL_SSL
        # sasl.mechanism: SCRAM-SHA-512
```

`spring.kafka.*`에 없는 옵션은 전부 이 `properties` 아래로 넘긴다.
여기 적은 키는 Spring이 해석하지 않고 **kafka-clients에 그대로 전달**된다.

## 함정: 표준 키와 커스텀 키를 같은 네임스페이스에 섞지 말 것

이건 실제로 헷갈렸던 부분이다. 설정이 이렇게 되어 있었다.

```yaml
spring:
  kafka:
    bootstrap-servers: <broker-host>:9092
    partitions: 64                 # ← 표준 키가 아니다
    consumer:
      id: my-consumer-id           # ← 표준 키가 아니다
      topics: topic-a              # ← 표준 키가 아니다
      group-id: my-consumer-group  # ← 표준 키
```

`spring.kafka.*`는 `KafkaProperties`에 바인딩되는데, `partitions` / `consumer.id` / `consumer.topics`는
**거기에 없는 키다.** 그러면 어떻게 될까.

**아무 일도 일어나지 않는다.** 에러도, 경고도 없다. 바인딩에서 조용히 무시된다.

이 값들은 `@Value`로 따로 읽어서 쓰고 있었다.

```java
@Value("${spring.kafka.consumer.id}")
private String consumerId;

@Value("${spring.kafka.partitions}")
private int kafkaPartitions;
```

동작은 한다. 하지만 yaml만 봐서는 **어느 키가 Spring이 해석하는 것이고 어느 키가 내 코드가 읽는 것인지 구분이 안 된다.**
`consumer.id`를 보고 "Kafka의 `client.id`겠거니" 하고 넘어가기 딱 좋다. (표준 키는 `consumer.client-id`다.)

그래서 **애플리케이션 전용 키는 자기 네임스페이스로 빼는 게 맞다.**

```yaml
spring:
  kafka:
    bootstrap-servers: <broker-host>:9092
    consumer:
      group-id: my-consumer-group

app:
  kafka:
    partitions: 64
    consumer-id: my-consumer-id
    topics: topic-a
```

이렇게 두면 `@ConfigurationProperties(prefix = "app.kafka")`로 묶어서 타입 안전하게 받을 수도 있고,
무엇보다 **읽는 사람이 경계를 알 수 있다.**

## 리스너 선언

여기까지 설정하면 리스너는 애노테이션 하나로 끝난다.

```java
@KafkaListener(id = "${app.kafka.consumer-id}", topics = "topic-a")
public void listenA(ConsumerRecord<String, String> record, Acknowledgment ack) {
    handleRecord(record, ack);
}

@KafkaListener(id = "${app.kafka.consumer-id}-b",
               topics = "topic-b",
               groupId = "${spring.kafka.consumer.group-id}")
public void listenB(ConsumerRecord<String, String> record, Acknowledgment ack) {
    handleRecord(record, ack);
}
```

몇 가지 짚어둔다.

- **`id`는 리스너 컨테이너의 식별자다.** 지정해두면 나중에
  `KafkaListenerEndpointRegistry`로 이 컨테이너를 찾아 `start()` / `stop()`을 걸 수 있다.
  Graceful shutdown에서 쓰게 되는데, 이것도 따로 다룬다.
- `id`는 **유일해야 한다.** 그래서 두 번째 리스너에 `-b`를 붙였다.
- 파라미터로 `String` 대신 `ConsumerRecord<String, String>`을 받으면
  값뿐 아니라 `partition()`, `offset()`, `timestamp()`까지 쓸 수 있다.
  로그에 파티션/오프셋을 남겨두면 장애 추적이 훨씬 쉬워진다.
- `Acknowledgment`는 `ack-mode`가 수동일 때만 주입된다.
  `MANUAL` / `MANUAL_IMMEDIATE`가 아니면 이 파라미터를 받는 순간 예외가 난다.

## 정리

- `spring-boot-starter-kafka`는 없다. `org.springframework.kafka:spring-kafka`를 직접 넣는다
- 버전은 **적지 않는다.** Boot BOM이 `kafka-clients`까지 맞춰준다
- 요구가 없으면 **설정 클래스를 만들지 않는다.** 자동 설정 + 프로퍼티로 충분하다
- `bootstrap-servers`는 진입점이다. 운영이면 여러 대를 적는다
- `group-id`는 오프셋의 주인이다. **바꾸면 새 컨슈머가 된다**
- 외부가 만든 메시지라면 `String`으로 받아 직접 파싱하는 쪽이 방어적이다
- `auto-offset-reset`은 **오프셋이 없을 때만** 쓰인다
- `spring.kafka.*`에 **표준이 아닌 키를 섞지 않는다.** 조용히 무시되고, 읽는 사람이 헷갈린다

여기까지가 "붙이기" 다. 컨슈머가 실제로 어떻게 동작하는지 — 수동 커밋과 at-least-once,
순서 보장, 종료 처리 — 는 `Kafka > Consumer` 쪽에서 이어서 정리한다.
