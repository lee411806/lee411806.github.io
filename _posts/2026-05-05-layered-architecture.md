---
layout: post
title: "Layered Architecture — 3계층 vs 4계층, 그리고 DDD가 가능해지는 이유"
categories: [아키텍처, DDD]
---

## 🏗️ 3계층 아키텍처 (3-Tier Architecture)

가장 고전적이고 직관적인 구조. 소규모 프로젝트나 빠른 개발이 필요할 때 주로 사용한다.

- **Presentation Layer** — 사용자의 요청을 받고 응답을 돌려주는 입출력 창구 (Controller, View)
- **Business Layer** — 애플리케이션의 핵심 로직이 실행되는 곳 (Service)
- **Data Access Layer** — DB에 직접 접근하여 데이터를 처리 (Repository, DAO)

---

## 🏗️ 4계층 아키텍처 (Standard 4-Layer Architecture)

비즈니스 로직인 '도메인'을 '인프라'와 분리하여 유지보수와 확장성을 극대화한 구조.

- **Presentation Layer** — UI 렌더링 및 외부 API 요청/응답 처리 (Controller, DTO)
- **Application Layer** — 비즈니스 흐름 제어 및 트랜잭션 관리 (App Service, Facade)
- **Domain Layer** — 핵심 비즈니스 규칙과 데이터 모델 정의, 시스템의 심장부 (Entity, Value Object)
- **Infrastructure Layer** — DB, 외부 API 등 실제 기술 구현체 (Repository Impl, DB Driver)

> 구체적인 구현체에 휘둘리지 않고, 내가 정의한 규칙(인터페이스)에 따라 어떤 부품을 꽂을지 '설정'만 하면 되기 때문에 변화에 유연하고 관리가 편해진다.

---

## 왜 Application과 Domain 모듈을 나누는가?

- **Domain** — Spring, DB에 의존하지 않는 순수 Kotlin으로만 비즈니스 규칙과 객체 로직을 담는다.
- **Infrastructure / Presentation** — DB 연결이나 UI 출력 같은 외부 기술 구현만 담당한다.
- **Application** — 순수한 Domain 객체를 가져와 외부 요소와 연결하고, 실제 비즈니스 흐름을 조립한다.

---

## 실제 프로젝트에서 어떻게 구현했나?

### 모듈 구조 (Gradle 멀티모듈)

계층을 패키지가 아닌 **별도 모듈**로 물리적으로 분리했다. 잘못된 방향으로 import하면 빌드 자체가 실패한다.

```
oop-presentation   → Controller, Request/Response DTO, Filter
oop-application    → UseCase 인터페이스, Service (흐름 제어)
oop-domain         → Entity, Value Object, Port 인터페이스 (Spring 없음)
oop-infrastructure → JPA Entity, Repository Adapter, JWT, BCrypt
oop-common         → 공통 예외(BaseException), ErrorCode
oop-boot           → 전체 모듈 조립, SpringBootApplication
```

**의존 방향**: `Presentation → Application → Domain ← Infrastructure`

Domain은 누구에게도 의존하지 않는다.

---

### Application Layer가 하는 일 (흐름 제어란?)

Application Service가 하는 일은 딱 세 가지다.

1. Domain 객체 생성/호출
2. Port(인터페이스)를 통해 Infrastructure에 위임
3. `@Transactional`로 트랜잭션 경계 관리

**비즈니스 규칙 자체(소유자 검증, 길이 제한 등)는 Domain 객체가 직접 처리**한다.

```kotlin
// oop-application: PostService.kt
@Service
class PostService(private val postPort: PostPort) : PostUseCase {

    @Transactional
    override fun update(command: UpdatePostCommand): UpdatePostResult {
        val post = postPort.getById(command.id)
            ?: throw BaseException(ErrorCode.NOT_FOUND)

        post.validateOwner(command.authorUsername)  // 규칙은 Domain이 처리
        val updated = post.update(...)              // 업데이트도 Domain이 처리
        return UpdatePostResult.from(postPort.store(updated))
    }
}
```

---

### Domain 계층: Spring 코드 0줄

```kotlin
// oop-domain: Post.kt
class Post private constructor(
    val title: PostTitle,       // Value Object — 최대 100자 검증
    val content: PostContent,   // Value Object — 최대 5000자 검증
    val authorUsername: String,
) {
    fun validateOwner(username: String) {
        if (authorUsername != username)
            throw BaseException(ErrorCode.POST_ACCESS_DENIED)
    }

    companion object {
        fun create(title: String, content: String, author: String) =
            Post(null, PostTitle.from(title), PostContent.from(content), author, ...)
    }
}
```

`@Entity`, `@Service` 같은 Spring 어노테이션이 없다. Spring Context 없이 순수 단위 테스트가 가능하다.

---

### Port & Adapter: Domain ↔ Infrastructure 연결

Domain이 인터페이스(Port)로 요청하고, Infrastructure가 JPA로 실제 구현(Adapter)한다.

```kotlin
// oop-domain: PostPort.kt (인터페이스)
interface PostPort {
    fun store(post: Post): Post
    fun getById(id: Long): Post?
    fun delete(id: Long)
}
```

```kotlin
// oop-infrastructure: PostPersistenceAdapter.kt (구현체)
@Component
class PostPersistenceAdapter(
    private val postEntityRepository: PostEntityRepository
) : PostPort {

    override fun store(post: Post): Post {
        val entity = PostEntity.fromDomain(post)            // Domain → JPA Entity
        return postEntityRepository.save(entity).toDomain() // JPA Entity → Domain
    }
}
```

JPA Entity(`PostEntity`)와 Domain(`Post`)은 완전히 분리된 별개 클래스다.

---

### 3계층 vs 4계층 핵심 차이

| 관점 | 3계층 | 4계층 (이 프로젝트) |
|------|-------|---------------------|
| 비즈니스 규칙 위치 | Service 안에 혼재 | Domain 객체가 직접 보유 |
| DB/기술 교체 시 | Service까지 수정 필요 | Infrastructure Adapter만 수정 |
| 단위 테스트 | Spring Context 필요 | Domain은 순수 Kotlin 테스트 가능 |
| 모듈 경계 | 패키지 규칙 (강제 X) | Gradle 모듈로 물리적 강제 |
| Domain 의존성 | DB, ORM에 의존 | 아무것도 의존 안 함 |

---

## 💡 왜 4계층으로 나눠야 DDD가 가능한가?

### 3계층 Service의 문제

3계층 Service는 역할이 세 개다.

1. **DB 연동** — repository 호출
2. **흐름 제어** — 어떤 순서로 실행할지
3. **도메인 규칙** — 소유자 검증, 비밀번호 확인 등

이 세 개가 한 클래스 안에 섞이니까 의존성이 꼬이고, 도메인 규칙을 분리하고 싶어도 구조적으로 빠져나올 틈이 없다.

그래서 자연스럽게 Entity는 데이터 그릇이 되고 (Anemic Domain Model), 규칙은 전부 Service에 쏠린다.

```kotlin
// 3계층 — Post는 데이터만, 규칙은 Service에
class PostService {
    fun update(...) {
        if (post.authorUsername != username) throw Exception()  // 규칙이 여기
        post.title = newTitle
        repository.save(post)
    }
}
```

DDD의 핵심은 "도메인 객체가 자기 규칙을 스스로 안다"는 건데, Service가 규칙까지 들고 있으면 도메인한테 줄 게 없어진다.

### 4계층이 해결한 것

Service가 들고 있던 세 역할을 각자 하나씩 분리:

- **DB 연동** → Infrastructure
- **흐름 제어** → Application Service
- **도메인 규칙** → Domain

```kotlin
// 4계층 — Post가 자기 규칙을 직접 들고 있음
class Post {
    fun validateOwner(username: String) { ... }  // 규칙이 여기
}

// Service는 흐름만
class PostService {
    fun update(command) {
        val post = postPort.getById(command.id)
        post.validateOwner(command.username)  // Post한테 "네가 판단해"
        postPort.store(post.update(...))
    }
}
```

결국 4계층으로 Service를 쪼갠 것은 도메인한테 **자기 책임을 돌려준 것**이다.

---

## 회고

모듈을 나누고 확실하게 체감된 것:
- 테스트할 때 간편하다 (특히 도메인)
- 의존성이 얽혀있는 프로젝트를 테스트하려고 하니 복잡하다는 인상을 받았다. 구조의 명확함이 주는 이점을 직접 경험했다.
