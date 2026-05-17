---
layout: post
title: "동시성 문제가 발생하는 조건"
categories: [동시성 문제]
---

### 공유 자원 (Shared Resource)

다수 쓰레드가 함께 접근하는 자원 (메모리, DB, 파일 등)

---

### 가변 상태 (Mutable State)

읽기 전용이 아닌, 변경 가능한 상태일 때 문제 발생

---

### 동시에 접근 (Concurrent Access)

여러 쓰레드가 동시에 같은 자원에 접근

---

### 조회 후 수정 (Read-Modify-Write)

읽어서 계산하고 다시 쓰는 작업 사이에 다른 쓰레드가 개입할 수 있음

---

### 원자성 깨짐 (Atomicity Violation)

여러 단계의 연산이 원자적으로 실행되지 않을 때 중간 상태가 노출되어 교란 발생

#### ⚠️ DB 원자성 vs 동시성 원자성 — 맥락이 다르다

| | DB 원자성 (ACID) | 동시성 원자성 |
|---|---|---|
| 의미 | 트랜잭션 전체가 전부 성공 or 전부 실패 | 연산이 더 이상 쪼개지지 않고 한 번에 수행 |
| 단위 | 트랜잭션 단위 | 연산 단위 |
| 예시 | INSERT/UPDATE 롤백 | `count++` = read → modify → write 3단계 |

`count++`는 한 줄처럼 보여도 실제론 read → modify → write 3단계
→ 원자적이지 않아서 중간에 다른 스레드가 끼어들 수 있음
