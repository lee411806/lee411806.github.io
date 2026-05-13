---
layout: post
title: "API 구조, DB 접근 패턴, 통신 방식 정리"
categories: [동시성 문제]
---

### API 구조

**Controller 3개**

- `HealthController` → GET /api/health (서버 상태 확인)
- `PostController` → POST/GET/PUT/DELETE /posts (게시글 CRUD + 페이지네이션)
- `UserController` → /users/join, /users/login, /users/refresh, /users/me (회원가입/로그인/토큰갱신)

**공통 응답 포맷**

- 단건: `ApiResponse<T>` → `{ code, data }`
- 목록: `PageResponse<T>` → `{ content, totalCount, totalPage, page, size }`

**API 책임 범위 — 동시성 관점**

| | API 하나로 묶기 | API 쪼개기 |
|---|---|---|
| 동시성 | 트랜잭션 하나로 쉽게 해결 | 레이어마다 따로 해결 (분산 락, Saga 등) |
| 멱등성 | 중간 실패 추적 어려움, 재시도 위험 | 각 API가 독립적으로 멱등하게 설계 가능 |
| 책임 | SRP 위반, 하나가 너무 무거움 | 각자 명확한 책임 |

> API 길이가 길어지면 Lock을 오래 점유 → 경합 심화

**통신 방식과 동시성**

- **동기(REST)**: 요청이 많아지면 스레드 풀 고갈 → 스레드 경쟁
- **비동기/이벤트**: 동시성을 큐 시스템으로 위임 — 문제가 없어지는 게 아니라 중복 처리 방지를 대신 제공

---

### DB 접근 패턴 - JPA + JOOQ 하이브리드

```
PostEntityRepository (인터페이스)
  └── PostEntityRepositoryImpl (구현체)
        ├── PostJpaRepository   → 기본 CRUD (save, findById, delete)
        └── PostJooqRepository  → 페이지네이션 (정렬/오프셋 포함 복잡 쿼리)
```

- 단순 CRUD는 JPA, 정렬/페이징 포함 복잡 쿼리는 JOOQ로 분리
- User, Health는 단순 조회라 JPA만 사용
- DB: PostgreSQL (운영), H2 (테스트)

**왜 조회를 분리했는가**

JPA는 조회 시 Entity 생성 + 영속성 컨텍스트 관리 + Dirty Checking + Lazy Loading + 연관관계 관리까지 수행  
→ 복잡 조회에서 N+1, 불필요한 추가 쿼리, Flush 영향 등으로 조회 비용이 커짐

**동시성 관점**

```
복잡 조회 무거워짐
 → 커넥션 오래 점유
 → 요청 적체
 → 스레드/커넥션 경합 증가
```

JOOQ는 필요한 컬럼만 직접 SQL 조회 → Lazy Loading 없음, N+1 없음, Entity 상태관리 없음, 조회 비용 예측 가능

```
쓰기/정합성  → JPA
읽기/복잡조회 → JOOQ
```

조회 성능 안정화 + 커넥션 점유 시간 감소 → 동시 요청 상황에서 경합 완화

---

### 통신 방식

**헥사고날 아키텍처**

- Domain 계층이 Port(인터페이스)만 정의 → Infrastructure가 Adapter(구현체)로 구현
- 데이터 흐름: Controller → UseCase → Service → Port → Adapter → JPA/JOOQ

**JWT 인증 흐름**

- Access Token 30분 / Refresh Token 7일 (JJWT + HMAC-SHA)
- 요청마다 `JwtAuthFilter`가 Authorization 헤더 검증
- 검증 통과 시 request attribute에 username 저장 → `@CurrentUser`로 컨트롤러 주입
- 비밀번호: BCrypt 암호화

**외부 통신**

- WebClient, FeignClient, Kafka 없음 → REST + JWT만 사용

---

### 현재 구조의 동시성 트레이드오프

**장점 — 구조가 단순함**

```
Request → Service → DB → Response
```

- 트랜잭션 흐름 추적 쉬움, 디버깅 단순
- 분산 락, 이벤트 순서, 중복 소비, 최종 일관성 같은 분산 시스템 문제 고려 불필요

**단점 — 요청 증가 시 경합을 직접 받음**

```
요청 증가
 → 스레드/DB 커넥션 점유 증가
 → 응답 지연
 → 경합 증가
```

복잡 조회·느린 쿼리가 발생하면 스레드 오래 점유 → 처리량 감소

**비동기 구조였다면**: Kafka/Event 기반으로 Queue가 부하를 흡수할 수 있으나, 중복 처리·멱등성·Eventual Consistency 같은 분산 시스템 복잡도 증가

```
현재 구조 → 단순하고 예측 가능, 대신 부하가 오면 경합을 직접 받음
비동기 구조 → 확장성·버퍼링 유리, 대신 분산 시스템 복잡도 증가
```
