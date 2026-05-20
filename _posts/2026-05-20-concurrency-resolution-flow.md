---
layout: post
title: "동시성 문제 해결 흐름 — 설계부터 락까지"
categories: [동시성 문제]
---

## 락부터 고르는 게 아니다

동시성 문제가 생겼을 때 바로 `synchronized`나 `SELECT FOR UPDATE`를 꺼내는 건 4번 단계부터 시작하는 것.
앞 단계에서 해결할 수 있으면 훨씬 단순해진다.

---

## 0단계. 공유 상태 최소화 설계

**락을 걸기 전에 공유 상태 자체를 안 만든다**

- `final` / `static final` — 변경 불가 상수
- **불변 객체 (Immutable Object)** — 한번 만들면 상태를 바꾸지 않는 객체
- **순수 함수 (Pure Function)** — 외부 상태를 읽지도, 변경하지도 않음
- **DDD Value Object** — 상태 없이 값만 표현하는 객체

→ 여러 스레드가 건드릴 공유 상태 자체를 없애서 동시성 문제 발생을 차단

---

## 1단계. 공유를 없앨 수 있나? → ThreadLocal

공유가 불가피하지만, 스레드마다 자기 복사본을 주면 해결되는 경우

- 스레드끼리 데이터를 아예 공유 안 함 → 락 불필요
- Spring `@Transactional`이 DB 커넥션을 스레드에 묶을 때 내부적으로 사용

**주의**: 스레드 풀 환경에서 `remove()` 안 하면 이전 요청 데이터가 다음 요청에 남아있음

→ 락 없이 해결

---

## 2단계. 읽기가 대부분인가? → ReadWriteLock

읽기는 동시에 여럿이 OK, 쓰기만 독점

- `synchronized`는 읽기도 전체 잠금 → 읽기가 압도적으로 많으면 낭비
- `ReadWriteLock`은 읽기끼리는 서로 안 막고, 쓰기가 일어날 때만 전체 대기

→ `synchronized` 대비 동시 처리량 대폭 향상

---

## 3단계. 1개가 아닌 N개까지 허용해도 되나? → Semaphore

"한 명만"이 아니라 **"N명까지만"** 허용

- 외부 API 동시 호출 5개 제한
- DB 커넥션 풀 직접 구현

→ 리소스 제한이 목적이면 Semaphore로 충분

---

## 4단계. 그래도 락이 필요하다면 — 비관적 vs 낙관적

**충돌이 잦을 때 → 비관적**

| 레이어 | 도구 |
|---|---|
| 앱/서비스 | `synchronized`, `ReentrantLock` |
| DB | `SELECT FOR UPDATE`, 2PL |
| 분산 | Redis Redlock |

**충돌이 드물 때 → 낙관적**

| 레이어 | 도구 |
|---|---|
| 앱/서비스 | `CAS`, `AtomicInteger`, `ConcurrentHashMap` |
| DB | `@Version`, MVCC |
| 분산 | Raft/Paxos |

---

> DB 락도 결국 4단계의 **DB 레이어 구현체**일 뿐.
> 락의 종류를 고르기 전에 **동시성 문제를 얼마나 앞 단계에서 줄일 수 있는지**가 먼저다.
