---
layout: post
title: "Mutable, val/var"
categories: [동시성 문제]
---

> 상태가 바뀔 수 있나? 를 판단하는 추상적인 개념

---

### 1. Kotlin의 `val` / `var`

자바로 대응시키면:

| Kotlin | Java |
|---|---|
| `val` | `final` |
| `var` | 일반 변수 |

---

### 핵심: `final`인데 Mutable 가능?

여기서 많이 헷갈립니다.

**Java 예시**

```java
final List<String> list = new ArrayList<>();
list.add("A"); // 가능!
```

---

### 왜 가능한가?

`final`이 막는 건 **변수 재할당**이지, **객체 내부 상태 변경**이 아니기 때문입니다.

| 구분 | 가능 여부 |
|---|---|
| `list = new ArrayList<>()` | ❌ 불가능 (재할당) |
| `list.add("A")` | ✅ 가능 (내부 상태 변경) |
