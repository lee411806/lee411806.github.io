---
layout: post
title: "Layered 아키텍처 의존성 설정 (응집도, 결합도)"
categories: [아키텍처]
---

> "변경 가능성이 높은 presentation과 infrastructure는 외부에 두고, application에서는 비즈니스 흐름만 조립하며, domain은 외부 기술을 전혀 모르는 상태로 유지합니다."

---

### 결합도가 낮다

👉 **"결합도가 낮다 = 변경이 '전파되지 않는다'"**

"자기 모듈에서만 바꾸면 된다" — 더 정확히는 **"다른 모듈을 건드릴 필요가 없다"**

---

### 응집도가 높다

"응집도가 높다 = 한 모듈이 하나의 역할만 제대로 한다"

- presentation : 외부 요청 받기
- application(service): 흐름 이어주기

---

### 왜 domain을 application, infrastructure가 참조하는가?

domain이 시스템의 "규칙(비즈니스 룰)"을 가지고 있기 때문입니다.

> "비즈니스 규칙을 domain에 모으고 외부 기술과 분리하면, 기술 변경이 발생해도 핵심 로직이 영향을 받지 않아 변경에 강한 구조가 됩니다."

```
presentation → application
application  → domain
infrastructure → domain
```

---

### 계층별 역할

**1️⃣ Presentation**
- HTTP 요청 받기
- application 호출

**2️⃣ Application**
- 흐름 제어 (orchestration)
- domain 호출, interface 사용
- "조립"이라기보다 **흐름만 관리**

**3️⃣ Domain**
- 핵심 비즈니스 규칙
- 외부 모름

**4️⃣ Infrastructure**
- DB, 외부 API, 기술 구현

---

### 프로젝트 실제 구현 예시

**핵심 요약**
- `presentation → application → domain ← infrastructure`
- Domain은 외부를 전혀 모르는 상태 유지
- 결합도 낮음 = 변경이 전파되지 않음

---

#### 1. 결합도 낮음 → Port 인터페이스로 구현

**UserPort.kt** — domain 패키지에 인터페이스로 정의:

```kotlin
// domain/user/port/UserPort.kt
interface UserPort {
    fun register(user: User): User
    fun isUsernameTaken(username: UsernameVO): Boolean
    fun getByUsername(username: UsernameVO): User?
}
```

**UserPersistenceAdapter.kt** — infrastructure 패키지에서 구현:

```kotlin
// infrastructure/user/adapter/UserPersistenceAdapter.kt
@Repository
class UserPersistenceAdapter(
    private val userEntityRepository: UserEntityRepository
) : UserPort {  // UserPort 인터페이스 구현

    override fun register(user: User): User =
        userEntityRepository.save(UserEntity.fromDomain(user)).toDomain()

    override fun isUsernameTaken(username: UsernameVO): Boolean =
        userEntityRepository.existsByUsername(username.value)

    override fun getByUsername(username: UsernameVO): User? =
        userEntityRepository.findByUsername(username.value)?.toDomain()
}
```

**JoinService.kt** — application 패키지, UserPort 인터페이스만 바라봄:

```kotlin
// application/user/service/JoinService.kt
@Service
class JoinService(
    private val userPort: UserPort,                           // 인터페이스만 앎
    private val passwordEncryptorPort: PasswordEncryptorPort  // 인터페이스만 앎
) : JoinUseCase {

    @Transactional
    override fun join(command: JoinCommand) {
        val user = User.signUp(
            username = UsernameVO.from(command.username),
            password = RawPasswordVO.from(command.password),
            passwordEncryptor = passwordEncryptorPort,
            userPort = userPort
        )
        userPort.register(user)
    }
}
```

👉 `JoinService`는 JPA, DB 기술을 전혀 모름. DB를 JPA → JOOQ로 바꿔도 `JoinService` 코드 변경 없음 = **결합도 낮음**

---

#### 2. 응집도 높음 → 계층별 역할 하나씩

| 파일 | 역할 하나만 |
|---|---|
| `JoinService` | 회원가입 흐름만 |
| `UserPort` | DB 접근 인터페이스 정의만 |
| `UserPersistenceAdapter` | JPA로 DB 저장/조회만 |
| `User` (domain) | 회원가입 비즈니스 규칙만 |

---

#### 3. 의존성 방향

```
JoinService (application)
    ↓ 바라봄
UserPort (domain - 인터페이스)
    ↑ 구현
UserPersistenceAdapter (infrastructure)
```

`UserPort`가 domain에 있기 때문에 infrastructure가 domain을 바라보는 구조. Domain은 infrastructure를 전혀 모름.
