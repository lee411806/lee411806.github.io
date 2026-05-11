---
layout: post
title: "Thread Safety"
categories: [동시성 문제]
---

### 핵심

> 같은 JAR 안에서 스레드 풀이 여러 스레드를 동시에 돌릴 때, 그 스레드들이 **같은 자원(변수/객체)을 건드려도 결과가 항상 올바른 상태**

### 레이어 위치

| | 공통 자원 | 범위 |
|---|---|---|
| 멀티 인스턴스 | DB, Redis | 프로세스 간 |
| **Thread Safety** | 메모리(변수, 객체) | **프로세스 안** |

---

### 왜 위험하냐

`count++`은 사실 3단계예요:

```
1. count 읽기
2. +1 계산
3. count 저장
```

이 사이에 다른 스레드가 끼어들면:

```
count = 0

스레드 A: count 읽음 (0)
스레드 B: count 읽음 (0)  ← A가 저장하기 전에 읽어버림
스레드 A: +1 해서 저장 (1)
스레드 B: +1 해서 저장 (1)  ← 자기가 읽은 0 기준으로

기대값: 2  /  실제값: 1
```

---

### 변수 종류별 Thread Safety

| 변수 종류 | 설명 | Thread Safe? |
|---|---|---|
| 지역 변수 | 메서드 안 선언 | ✅ 안전 (스레드마다 따로 가짐) |
| 인스턴스 변수 | 객체 필드 | ⚠️ 위험 (싱글톤이면 공유됨) |

**코드 예시:**

```java
class Counter {
    int count = 0;  // 인스턴스 변수 (필드) - 객체에 속함

    void increment() {
        int temp = 0;  // 지역 변수 - 이 메서드 안에서만 존재
        temp++;
        // 메서드 끝나면 temp 사라짐
    }
}
```

```
스레드 A가 increment() 호출 → temp 생성 (A 전용)
스레드 B가 increment() 호출 → temp 생성 (B 전용)  ← 완전히 별개 → 안전

스레드 A가 count++
스레드 B가 count++  ← 둘 다 같은 count 건드림 → 위험
```

---

### 스프링과의 연결

**스프링 빈 = 싱글톤**
- `@Service`, `@Component` 등 등록하면 스프링이 서버 시작 시 **단 한 번** new
- Application Context에 보관
- 요청 올 때마다 컨테이너에서 **같은 객체** 꺼냄

```
스레드 A → Application Context → Counter 객체 반환 ─┐
스레드 B → Application Context → Counter 객체 반환 ─┤ ← 셋 다 같은 객체!
스레드 C → Application Context → Counter 객체 반환 ─┘
                                              ↓
                                     인스턴스 변수 공유 → 위험!
```

**스프링 실제 예시:**

```java
@Service
public class PostService {

    private int requestCount = 0;  // 인스턴스 변수 - 위험! 쓰면 안됨

    public void createPost(String title) {
        int length = title.length();  // 지역 변수 - 안전
        requestCount++;               // 인스턴스 변수 건드림 - 위험
    }
}
```

`requestCount` → 모든 요청 스레드가 공유 → 카운트 틀려질 수 있음

`length` → 요청마다 새로 만들어짐 → 안전

---

### 싱글톤 = 객체 1개만 존재

- 싱글톤 ≠ 한 번에 한 스레드만 쓸 수 있다는 것
- 싱글톤 = **객체가 1개만 존재**, 여러 스레드가 **동시에** 같은 객체 사용 가능
- 그래서 스프링 서비스 클래스에 **인스턴스 변수 쓰지 말 것**

---

### 면접 흐름

| 레벨 | 공통 자원 | 해결책 |
|---|---|---|
| 멀티 인스턴스 | DB, Redis | 분산 락 |
| 멀티 스레드 | 메모리(인스턴스 변수) | synchronized, AtomicInteger 등 |
