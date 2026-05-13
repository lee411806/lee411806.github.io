---
layout: post
title: "인스턴스 변수, static 자원 사용 현황 식별"
categories: [동시성 문제]
---

### 인스턴스 변수

**모든 Spring Bean(@Service, @Repository, @Controller)은 상태 없음**

- 의존성 주입 필드만 존재, mutable 상태 없음 → 멀티스레드 안전

**해결방안**

- 클래스 안에 인스턴스변수 대신 지역변수로 막기. 행위를 할 때 companion 메서드로 static 선언 → 안전

1. **싱글톤 공유**: @Service, @RestController는 딱 하나의 객체만 생성되므로, 모든 스레드가 동시에 같은 객체를 참조
2. **동시성 위험**: 공유 객체 안에 `var`로 된 인스턴스 변수가 있으면, 여러 스레드가 동시에 그 값을 바꾸려다 데이터 충돌 발생
3. **우리 프로젝트**: 모든 인스턴스 변수를 `val`로 만들거나 의존성(도구)으로만 구성 → 동시성 문제 설계적 차단

> `val`(final)도 참조형 객체에 붙이면 객체 내부 수정이 가능하므로 완전한 immutable이 아님

- **객체 내부 필드를 전부 `val`로 박기**: 밖에서 참조를 가져도 내부 값을 바꿀 통로가 차단
- **Setter를 아예 안 만들기**: `listOf()`처럼 수정 메서드 자체가 없으면 참조형임에도 안전

**주목할 곳 1개**

- `PasswordEncryptorAdapter` → `private val encoder = BCryptPasswordEncoder()`
- `val`이라 재할당 불가, BCryptPasswordEncoder 자체는 스레드세이프 → 실질적 문제 없음

**도메인/DTO 클래스 전부 `val`(불변)으로 설계**

---

### static 자원

| 위치 | 종류 | 예시 |
|---|---|---|
| 모든 도메인/DTO/결과 클래스 | `companion object` factory 메서드 | `Post.create()`, `Post.reconstruct()`, `LoginResult.of()` |
| `PostTitle`, `PostContent` | `const val` 상수 | `MAX_LENGTH = 100`, `MAX_LENGTH = 5000` |
| `ApiResponse` | `const val` · factory 메서드 | `SUCCESS_CODE = "SUCCESS"`, `success()`, `fail()` |
| `ErrorCode` | enum 상수 | D/A/C/S 시리즈 에러코드 |
| `JwtHandlerAdapter` | `private val secretKey` (lazy 초기화) | HMAC-SHA 서명 키 |

---

### 전체 패턴 요약

- 모든 객체 생성을 `companion object` factory 메서드로 제어
- private constructor + `from()` / `of()` / `create()` / `reconstruct()` 네이밍 일관되게 사용
- Side-effect 최소화, 함수형 설계 원칙 준수

1. **무상태 팩토리 메서드 (companion object)**: 메서드 내부에서 클래스 변수를 수정하지 않고 새 객체를 생성/반환 → 여러 스레드가 동시에 호출해도 충돌 없음
2. **불변 상수 (const val, enum)**: 컴파일 시점에 고정, 런타임에 변경 불가 → 수천 스레드가 동시에 읽어도 완전 안전
3. **안전한 초기화 (lazy, private val)**: Kotlin의 `lazy`는 기본적으로 동기화(Synchronized) 처리 → `secretKey`를 여러 스레드가 동시에 처음 접근해도 딱 한 번만 안전하게 생성
