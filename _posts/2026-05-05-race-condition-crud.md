---
layout: post
title: "Race Condition - CRUD 동작별 발생 가능 경로 도출"
categories: [동시성 문제]
---

### Race Condition이란?

2개 이상의 스레드가 공유 자원에 동시 접근할 때, 실행 순서에 따라 결과가 달라지는 버그

---

### CRUD별 Race Condition 발생 가능 경로

**CREATE 회원가입 — 평가: 구체적 위험**

```
isUsernameTaken("alice") → false   ← Thread A
isUsernameTaken("alice") → false   ← Thread B (동시)
save("alice")                       ← Thread A 성공
save("alice")                       ← Thread B → DB UNIQUE 제약 위반 (500 에러)
```

- 코드 체크(isUsernameTaken)와 저장(save) 사이 gap 존재
- DB UNIQUE 제약이 마지막 방어선, 애플리케이션 레벨에서 잘못 잡지 못함

**CREATE 게시글 작성 — 평가: 안전**

- `@GeneratedValue(IDENTITY)`로 DB가 ID 직접 채번 → 동시 INSERT 충돌 없음

**READ 목록 조회 — 평가: 주의 필요**

```
getAll() (JOOQ 페이지 조회)  ← 10개 가져옴
[다른 스레드가 3개 삭제]
countAll() (JPA count 쿼리)    ← 7 반환
→ totalPage 계산 오류
```

- `getAll()`과 `countAll()`이 원자적으로 실행 안 됨

**UPDATE 게시글 수정 — 평가: 구체적 위험**

```
Thread A: getById(1) → title="원본"
Thread B: getById(1) → title="원본"
Thread A: update("수정A") → save()
Thread B: update("수정B") → save()  ← A의 수정 덮어씀 (Lost Update)
```

- `@Version` 없음 → Optimistic Lock 없음
- `@Lock` 없음 → Pessimistic Lock 없음

**DELETE 게시글 삭제 — 평가: 구체적 위험**

```
Thread A: getById(1) → 존재 확인
Thread B: getById(1) → 존재 확인
Thread A: validateOwner() → 성공
Thread B: validateOwner() → 성공
Thread A: delete(1) → 성공
Thread B: delete(1) → 영향 행 0 (이미 없는데 성공으로 처리)
```

- `deleteById()` 반환값 안 씀 → 삭제 실패 감지 불가

---

### 종합 평가

| CRUD | 위험도 | 핵심 문제 |
|---|---|---|
| CREATE 회원가입 | HIGH | Check-Then-Act, DB 제약만 방어 |
| CREATE 게시글 | LOW | 안전 |
| READ 목록 | MEDIUM | count/getAll 비원자적 실행 |
| UPDATE | HIGH | Lost Update, 락 전무 |
| DELETE | HIGH | 이중 삭제, 반환값 미검증 |

**공통 부재 항목:** `@Version`(Optimistic Lock), `@Lock`(Pessimistic Lock), 트랜잭션 격리 수준 명시

---

### 해결책

- **Synchronized / Lock**: 한 번에 하나만 접근
- **Atomic 변수**: `AtomicInteger` 같이 read-modify-write가 원자적으로 처리
- **Immutable 객체**: 애초에 변경 불가능하게
