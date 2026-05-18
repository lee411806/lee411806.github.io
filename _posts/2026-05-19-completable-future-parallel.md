---
layout: post
title: "CompletableFuture - 병렬처리"
categories: [동시성 문제]
---

- CompletableFuture는 비동기 작업과 병렬 처리를 쉽게 하기 위한 Java API야.
- 여러 작업을 동시에 실행하고, 결과를 조합할 수 있어.
- 기존 Future보다 훨씬 유연하게 callback·체이닝·예외처리가 가능해.
- 주로 외부 API 호출, 병렬 조회, 비동기 로직 처리에 많이 사용돼.
- 핵심은 "스레드를 기다리지 않고 작업을 이어서 처리"하는 거야.

## 기존 Future vs CompletableFuture

| | Future | CompletableFuture |
|---|---|---|
| 결과 조합 | 불가 | 가능 |
| 콜백 | 불가 | 가능 (thenApply, thenAccept) |
| 예외처리 | 불편 | 가능 (exceptionally) |
| 병렬 실행 | 수동 | allOf / anyOf |

## 핵심 메서드

```java
// 비동기 실행
CompletableFuture.supplyAsync(() -> fetchData());

// 결과 변환 (체이닝)
.thenApply(data -> process(data));

// 병렬 실행 후 모두 완료 대기
CompletableFuture.allOf(future1, future2, future3).join();

// 예외처리
.exceptionally(ex -> fallback());
```

> 스레드를 기다리지 않고 작업을 이어서 처리하는 것이 핵심
