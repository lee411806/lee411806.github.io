---
layout: post
title: "동시성 제어 — 비관적 vs 낙관적 (원리 기반 정리)"
categories: [동시성 문제]
---

## 동시성 제어의 두 원리

알고리즘 이름보다 중요한 게 원리. 계층이 달라져도 동일한 두 갈래가 반복된다.

| 전략 | 도구 | 언제 쓰나 |
|---|---|---|
| **비관적 (먼저 잠그고)** | `synchronized`, `ReentrantLock`, `SELECT FOR UPDATE` | 충돌이 잦을 때 |
| **비관적 변형** | `ReadWriteLock`, `Semaphore` | 읽기가 많거나, N개까지만 허용할 때 |
| **낙관적 (나중에 검증)** | `CAS`, `AtomicInteger`, `ConcurrentHashMap`, `@Version`, MVCC | 충돌이 드물 때 |
| **공유 제거** | `ThreadLocal` | 공유 자체를 없앨 수 있을 때 |

분산 환경으로 가면 → Distributed Lock (Redlock, Raft/Paxos)

---

## 비관적 락 (Pessimistic Lock)

**철학**: 충돌이 난다고 가정 → 먼저 자물쇠를 채운다

- `synchronized` / `ReentrantLock` — JVM 레벨 1:1 독점
- `SELECT FOR UPDATE` / 2PL — DB 트랜잭션 레벨 독점
- `ReadWriteLock` — 읽기는 동시에 여럿이 OK, 쓰기만 독점. 읽기가 압도적으로 많은 서비스에서 `synchronized` 대신 씀
- `Semaphore` — "1명만"이 아닌 **"N명까지만"** 허용. 외부 API 동시 호출 제한, DB 커넥션 풀이 이 원리

**트레이드오프**: 안전하지만 데드락 위험, 처리량 낮음

---

## 낙관적 락 (Optimistic Lock)

**철학**: 충돌이 드물다고 가정 → 일단 작업하고, 저장 직전에 검증

- `CAS` / `AtomicInteger` — "내가 읽은 값 == 현재 값"이면 교체, 아니면 재시도
- `ConcurrentHashMap` — 맵 전체가 아닌 **버킷 단위로 잠금** (Lock Striping + CAS). `HashMap`은 공유하면 위험, `synchronizedMap`은 전체 잠금으로 느림
- `@Version` (JPA) — 커밋 시점에 버전 충돌 감지 → `OptimisticLockException`
- MVCC — DB가 낙관적 락을 구현하는 방식. 데이터 버전을 여러 개 유지해서 **읽기가 쓰기를 안 막음** (PostgreSQL, MySQL InnoDB)

**트레이드오프**: 빠르고 데드락 없음, 충돌 잦으면 무한 재시도 위험

---

## 공유 제거 (ThreadLocal)

**철학**: 잠글 필요 자체가 없게 → 스레드마다 자기 복사본을 가짐

- 스레드끼리 데이터를 아예 공유 안 함
- Spring `@Transactional`이 DB 커넥션을 스레드에 묶을 때 내부적으로 사용

**주의**: 스레드 풀 환경에서 `remove()` 안 하면 이전 요청 데이터가 다음 요청에 남아있음

---

> 어떤 동시성 문제든 **"충돌이 잦냐? 드무냐? 아니면 공유를 없앨 수 있냐?"**
> 이 세 질문이 설계 방향을 결정한다.
