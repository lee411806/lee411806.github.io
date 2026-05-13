---
layout: post
title: "DB - @Transactional, Connection Pool, JPA Lock 설정 및 현황 정리"
categories: [동시성 문제]
---

### @Transactional 현황

| Service | @Transactional | readOnly |
|---|---|---|
| JoinService | O | X |
| HealthService | O | X |
| PostService.create/update/delete | O | X |
| PostService.get/getList | O | O (readOnly=true) |
| LoginService | X | - |
| RefreshService | X | - |
| TokenValidationService | X | - |

**설계 포인트**

- Login/Refresh/TokenValidation은 DB 안 건드리고 JWT만 검증 → 트랜잭션 불필요, 정상
- Repository/Adapter 계층에는 @Transactional 없음 → Service 계층에서만 관리, 아키텍처상 올바름
- propagation, isolation 명시 없음 → 전부 기본값 사용 (REQUIRED, DB 기본 격리 수준)

**동시성 관련**

현재 장점
- Service 계층에서만 트랜잭션 관리
- 조회/쓰기 분리 (readOnly=true)
- JWT 계층에 트랜잭션 없음

문제 여지
- 아직 락 전략이 꼭 필요한 수준의 충돌 시나리오가 안 보임

> **번외 — RefreshService DB 저장·비교 로직 없음 → 강제 무효화 취약점**
>
> 1. 완전 무상태(Stateless) 방식: DB 저장·대조 없이 토큰 자체의 암호화 서명만 확인하여 재발급
> 2. SecretKey 의존: yml에 숨겨진 SecretKey를 인감 도장 삼아 토큰 진위 여부만 판단
> 3. 트랜잭션 부재: DB 상태를 변경하는 로직이 없어 @Transactional 미적용
> 4. 동시성 방어 취약: 동일 토큰으로 여러 번 동시에 요청해도 제한 없이 새 토큰 발급
> 5. 제어권 부족: 토큰 유출 시 DB 기록이 없어 강제 무효화·추적 불가

---

### Connection Pool

**전 환경 모두 HikariCP 명시 설정 없음 → Spring Boot 기본값 그대로**

| 항목 | 기본값 |
|---|---|
| maximum-pool-size | 10 |
| minimum-idle | 10 |
| connection-timeout | 30초 |
| idle-timeout | 10분 |
| max-lifetime | 30분 |

| 환경 | 파일 | HikariCP 명시 설정 |
|---|---|---|
| 로컬 | application-local.yml | X (Spring Boot 기본값) |
| 개발 | application-dev.yml | X (Spring Boot 기본값) |
| 운영 | application-prod.yml | X (Spring Boot 기본값) |
| 테스트 | application-test.yml | X (H2 메모리 DB) |

**주의:** 운영(prod)도 기본값이라 트래픽 많아지면 pool 부족 가능성 있음

**문제 여부**

Connection Pool은 동시 요청이 DB에 접근할 수 있는 개수를 제한하는 자원이다.

평소: 요청 → 커넥션 사용 → 반납 → 재사용 흐름이 안정적으로 반복

동시 요청 증가 + 느린 조회/긴 트랜잭션이 겹치면:
```
커넥션 반환 속도 < 점유 속도
 → 커넥션 대기
 → 요청 적체
 → 경합 증가
```

---

### JPA Lock 설정

| 항목 | 현황 |
|---|---|
| `@Version` (Optimistic Lock) | X - PostEntity, UserEntity, HealthEntity 전부 없음 |
| `@Lock` (Pessimistic Lock) | X - 모든 Repository 메서드 없음 |

**JPA 환경별 설정**

| | 로컬 | 운영 | 테스트 |
|---|---|---|---|
| ddl-auto | update | validate | create-drop |
| show-sql | true | false | true |
| dialect | PostgreSQL 명시 | 자동 감지 | H2 명시 |

- 운영에서 `ddl-auto: validate` → 스키마 변경 막고 검증만 하는 올바른 설정

---

### Lock 종류 정리

**Optimistic Lock (`@Version`)**

- Entity에 version 값을 두고 수정 시 버전 비교 수행
- 동시에 수정되면 버전 충돌 → `OptimisticLockException` 발생
- DB Lock을 직접 잡지 않아 성능에 유리
- 일반 CRUD, 게시글 수정, 좋아요 등에 주로 사용

**Pessimistic Lock (`@Lock`)**

- DB에서 실제 Row Lock(`FOR UPDATE`) 획득
- 다른 트랜잭션의 접근을 대기시킴
- 정합성은 강하지만 성능 저하 가능
- 결제, 예약, 재고 차감 등에 주로 사용

**Named Lock**

- MySQL에서 문자열 이름 기준으로 Lock 관리
- 특정 Row가 아니라 `"coupon"` 같은 이름 자체를 잠금
- 여러 서버에서도 동일 DB 기준으로 동시성 제어 가능
- 쿠폰 발급, 중복 실행 방지 등에 사용

**Distributed Lock**

- Redis/ZooKeeper 같은 외부 시스템으로 Lock 관리
- 여러 서버가 동일한 Lock 정보를 공유
- MSA·멀티 서버 환경에서 동시성 제어 가능
- 선착순 이벤트, 재고 차감, 중복 결제 방지 등에 사용

---

### Service 단 동시성 제어 vs JPA Lock

- `synchronized`는 JVM 내부 줄 세우기 방식 → 같은 서버 안에서만 동시 실행 차단
- 서버가 2대 이상이면 서로 Lock을 몰라 동시에 실행될 수 있음
- JPA Lock은 DB 자체가 Lock을 관리 → 여러 서버에서도 동시성 제어 가능
- 실무에서는 단순 `synchronized` 보다 DB Lock/JPA Lock을 더 많이 사용

---

### 종합 평가

트랜잭션 구조는 깔끔하지만, Lock이 전혀 없고 Connection Pool도 기본값이라 실제 운영 트래픽에는 취약한 상태
