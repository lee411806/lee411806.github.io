---
layout: post
title: "synchronized / 트랜잭션 / DB 락 비교"
categories: [동시성 문제]
---

### synchronized

- JVM 레벨 락
- 임계영역(Critical Section) 보호
- 단일 인스턴스에서만 유효

---

### 서비스 레벨 트랜잭션

- 작업 단위 보장 (ACID)
- rollback / commit 관리 목적
- 트랜잭션만으로는 동시성 제어 완전 해결 못 함

---

### DB 락 (비관적 / 낙관적)

- 실제 데이터 충돌 제어
- 멀티 인스턴스 환경에서도 정합성 보장 가능
- DB가 중앙에서 락 관리
