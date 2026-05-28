---
layout: post
title: "Port-Adapter 패턴과 Spring DI 연결 흐름"
categories: [Spring, 아키텍처]
---

> Spring Boot가 멀티모듈에서 어떻게 Port-Adapter를 자동으로 연결해주는지 이해한 내용 정리

---

## Port-Adapter 흐름 정리

1. **domain**에 `PostPort` 인터페이스가 있다 — "이런 기능이 필요해"라는 약속만 정의, 구현 없음
2. **`PostService`**가 `PostPort`를 생성자 주입으로 인식한다
3. **infra**의 `PostPersistenceAdapter`가 `PostPort`의 구현체다
4. service랑 infra 둘 다 domain을 참조하니까 `PostPort`를 공유할 수 있다
5. Spring Boot가 스캔할 때 `PostService`에서 `PostPort` 찾고, `@Repository` 달린 구현체를 연결해준다

---

## Spring이 연결해주는 흐름

```
boot가 com.jaeyong.oop 전체 스캔
    → PostService Bean 만들려고 보니 PostPort 타입 필요
    → PostPort 구현한 Bean 찾아보니 PostPersistenceAdapter 발견
    → 생성자에 자동으로 꽂아줌
```

`implements PostPort` 선언이 핵심 — Spring이 **타입으로 찾아서** 넣어줌

---

## 왜 Adapter를 끼우냐

`PostService`가 `PostJpaRepository` 직접 알면, JPA → MongoDB로 바꿀 때 비즈니스 로직도 같이 수정해야 함.

Adapter를 끼우면 DB가 바뀌어도 `PostPersistenceAdapter`만 새로 만들면 끝. `PostService`는 손 안 댐.

```
PostController
    → PostUseCase (인터페이스)
           ↑ 구현
    PostService
           → PostPort (인터페이스)
                  ↑ 구현
        PostPersistenceAdapter
                  ↓
                 DB
```

각 계층이 인터페이스만 알고, **연결은 Spring이 다 해줌**

---

## Adapter vs EntityRepository 차이

- **`PostPersistenceAdapter`** — `Post`(도메인 객체) ↔ `PostEntity`(DB 객체) 변환 담당
- **`PostEntityRepository`** — 실제 JPA/JOOQ로 DB에 쿼리 날리는 역할

---

## 추가로 확인한 것

- `@Service`, `@Repository` 없애고 둘 다 `@Component`여도 작동함 → Spring이 타입으로 찾아서 넣어주는 거라 애노테이션 종류 상관없음
- 구현체가 두 개 생기면 `@Qualifier("빈이름")`으로 지정해줘야 함 (빈 이름 기본값은 클래스명 첫글자 소문자)
- 서로 직접 참조할 필요 없고 Bean 이름만 맞으면 됨
